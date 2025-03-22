# PostgreSQL: Parent-Child PID

## Introduction

In response to your request, we have researched the parent-child PID relationships in PostgreSQL, specifically in the context of parallel job execution and stored procedures. You asked whether it is possible to identify a parent process that fires off child processes (PIDs) in PostgreSQL, with the aim of better managing and cleaning up blocking or hanging threads.

Below, we detail our research and demonstration efforts. We investigated whether the PostgreSQL system exposes a parent PID for each connection and explored ways to surface this information for troubleshooting purposes. Our findings demonstrate that while PostgreSQL does spawn backend processes from a single postmaster (parent process), the individual child PIDs do not have an exposed "parent PID" field in the standard SQL views (such as `pg_stat_activity`). This limitation is due to PostgreSQL's architecture, where the operating system manages the process hierarchy and only the postmaster’s PID is common to all backends.

In this document, we explain why the parent-child PID relationship is not directly surfaceable through SQL, describe our simulation of concurrent process activity, and outline the steps taken to validate active processes in a controlled environment.

## 1. Prepare Resources

### Step 1.1: Create PostgreSQL Flexible Server

- Log into [portal.azure.com](https://portal.azure.com/) and sign in

- Create a PostgreSQL Flexible Server

  - Click “Create a resource”

  - Search for “Azure Database for PostgreSQL Flexible Server” and select it

    

### Step 1.2: Confirm Connectivity

#### Configure Firewall

- Navigate to the new PostgreSQL Flexible Server >> Settings >> Networking
- Check "Allow public access from any Azure services within Azure to this server"

#### Test Connectivity

- In the Azure Portal, open Cloud Shell, ensure you are in PowerShell mode, and then run the following commands:

  ```powershell
  $client = New-Object System.Net.Sockets.TcpClient
  $client.Connect("<servername>.postgres.database.azure.com", 5432)
  $client.Connected
  $client.Close()
  ```

- Verify that the output returns True

  

### Step 1.3: Create Objects

In the Azure Portal, open Cloud Shell, ensure you are in PowerShell mode, and then run the following command:

```powershell
psql "host=<serverName>.postgres.database.azure.com port=5432 dbname=postgres user=<<admin_username>> sslmode=require"
```

#### Create Database

At the `postgres=>` prompt, run the following command:

```sql
CREATE DATABASE demo_db;
```

List databases to validate creation:

```
\l
```

Expected output:

```text
                                              List of databases
       Name        |     Owner      | Encoding |  Collate   |   Ctype    |         Access privileges         
-------------------+----------------+----------+------------+------------+-----------------------------------
 azure_maintenance | azuresu        | UTF8     | en_US.utf8 | en_US.utf8 | 
 azure_sys         | azuresu        | UTF8     | en_US.utf8 | en_US.utf8 | 
 demo_db           | rchapler       | UTF8     | en_US.utf8 | en_US.utf8 | 
 postgres          | azure_pg_admin | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0         | azure_pg_admin | UTF8     | en_US.utf8 | en_US.utf8 | =c/azure_pg_admin                +
                   |                |          |            |            | azure_pg_admin=CTc/azure_pg_admin
 template1         | azure_pg_admin | UTF8     | en_US.utf8 | en_US.utf8 | =c/azure_pg_admin                +
                   |                |          |            |            | azure_pg_admin=CTc/azure_pg_admin
(6 rows)
```

#### Create Table

At the `postgres=>` prompt, run the following command:

```sql
CREATE TABLE sample_data ( id SERIAL PRIMARY KEY, description TEXT, created_at TIMESTAMPTZ DEFAULT NOW() );
```

List tables to validate creation:

```
\dt
```

Expected output:

```text
            List of relations
 Schema |    Name     | Type  |  Owner   
--------+-------------+-------+----------
 public | sample_data | table | rchapler
(1 row)
```

#### Insert Data

At the `postgres=>` prompt, run the following command:

```sql
INSERT INTO sample_data (description)
VALUES ('Demo entry 1'), ('Demo entry 2'), ('Demo entry 3');
```

Validate data insertion by querying the table:

```sql
SELECT * FROM sample_data;
```

Expected output:

```text
 id |    description    |          created_at           
----+-------------------+-------------------------------
  1 | Demo entry 1      | 2025-03-22 10:00:00+00
  2 | Demo entry 2      | 2025-03-22 10:00:01+00
  3 | Demo entry 3      | 2025-03-22 10:00:02+00
(3 rows)
```

*Note: The timestamps in `created_at` will vary based on the actual insertion time.*



-------------------------

## 2. Simulate Activity

### Step 2.1: Create Function

- At the `postgres=>` prompt, run the following command:

  ```sql
  CREATE OR REPLACE FUNCTION simulate_long_query()
  RETURNS VOID AS $$
  BEGIN
      PERFORM pg_sleep(30);
      INSERT INTO sample_data (description) VALUES ('Long query complete at ' || NOW());
  END;
  $$ LANGUAGE plpgsql;
  ```

- Validate that the function was created successfully by running:

  ```sql
  \df simulate_long_query
  ```

- Expected output:

  ```text
                                List of functions
   Schema |        Name         | Result data type | Argument data types | Type 
  --------+---------------------+------------------+---------------------+------
   public | simulate_long_query | void             |                     | func
  (1 row)
  ```



------

### Step 2.2: Invoke Function and Validate Active Process Activity

- **Restart the Cloud Shell:** To ensure a clean environment, restart your Cloud Shell session.

- Run the following PowerShell script to: 1) launch multiple background jobs to run the function concurrently and then 2) validate the active sessions in PostgreSQL:

  ```powershell
  # Set the PGPASSWORD environment variable to avoid password prompts in each background job.
  $env:PGPASSWORD = "P@55word!"
  
  # Launch 3 background jobs that execute simulate_long_query() concurrently.
  for ($i = 1; $i -le 3; $i++) {
      Start-Job -ScriptBlock {
          psql "host=ubspfs.postgres.database.azure.com port=5432 dbname=postgres user=rchapler sslmode=require" -c "SELECT simulate_long_query();"
      }
  }
  
  # Wait for 5 seconds to allow the background jobs to start.
  Start-Sleep -Seconds 5
  
  # Validate active sessions by querying pg_stat_activity.
  # This command connects via psql and retrieves sessions running simulate_long_query(),
  # excluding the current session.
  psql "host=ubspfs.postgres.database.azure.com port=5432 dbname=postgres user=rchapler sslmode=require" -c "SELECT pid, usename, application_name, client_addr, backend_start, state, query FROM pg_stat_activity WHERE query LIKE '%simulate_long_query()%' AND pid <> pg_backend_pid();"
  ```

  **Expected output:**
   You should see several active entries with details similar to:

  ```text
    pid  | usename  | application_name |  client_addr  |         backend_start         | state  |             query             
  -------+----------+------------------+---------------+-------------------------------+--------+-------------------------------
   12331 | rchapler | psql             | 13.64.187.107 | 2025-03-22 15:49:08.472339+00 | active | SELECT simulate_long_query();
   12330 | rchapler | psql             | 13.64.187.107 | 2025-03-22 15:49:08.365667+00 | active | SELECT simulate_long_query();
   12329 | rchapler | psql             | 13.64.187.107 | 2025-03-22 15:49:08.309973+00 | active | SELECT simulate_long_query();
  (3 rows)
  ```

  These entries confirm that multiple processes (with their PIDs) are active, generated by running the function in parallel.
