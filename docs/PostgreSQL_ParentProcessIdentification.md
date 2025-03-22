# PostgreSQL: Parent Process Identification

## Prepare Resources

### Create PostgreSQL Flexible Server

- Log into [portal.azure.com](https://portal.azure.com/) and sign in

- Create a PostgreSQL Flexible Server
  - Click “Create a resource”

  - Search for “Azure Database for PostgreSQL Flexible Server” and select it

### Confirm Connectivity

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


### Create Objects

In the Azure Portal, open Cloud Shell, ensure you are in PowerShell mode, and then run the following command:

```powershell
psql "host=ubspfs.postgres.database.azure.com port=5432 dbname=postgres user=<admin_username> sslmode=require"
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



## "PostgreSQL does less than we want..."

> "PostgreSQL implements a 'process per user' client/server model. In this model, every client process connects to exactly one backend process. As we do not know ahead of time how many connections will be made, we have to use a 'supervisor process' that spawns a new backend process every time a connection is requested. This supervisor process is called postmaster and listens at a specified TCP/IP port for incoming connections. Whenever it detects a request for a connection, it spawns a new backend process."

[PostgreSQL: Documentation: 17: 50.2. How Connections Are Established](https://www.postgresql.org/docs/current/connect-estab.html)

> "Thus, the supervisor server process is always running, waiting for client connections, whereas client and associated server processes come and go. (All of this is of course invisible to the user. We only mention it here for completeness.)"

[PostgreSQL: Documentation: 17: 1.2. Architectural Fundamentals](https://www.postgresql.org/docs/17/tutorial-arch.html)

> "Process ID of the parallel group leader if this process is a parallel query worker, or process ID of the leader apply worker if this process is a parallel apply worker. NULL indicates that this process is a parallel group leader or leader apply worker, or does not participate in any parallel operation."

[PostgreSQL: Documentation: 17: 27.2. The Cumulative Statistics System](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-PG-STAT-ACTIVITY-VIEW)

### Synopsis

- PostgreSQL’s native process model spawns a separate backend process for each connection
- The SQL interface (`pg_stat_activity`) exposes only the individual backend PIDs; it does not include a generic parent PID
- Although the operating system maintains a parent-child hierarchy (with the postmaster as the common parent), this relationship is not surfaced through SQL
- The `leader_pid` column is available only for parallel query workers and does not serve as a generic parent identifier
- Users who need to group or manage processes based on a parent-child relationship must implement custom annotations or rely on OS-level tools



### Demonstration

In the Azure Portal, open Cloud Shell, ensure you are in PowerShell mode, and then run the following command:

```powershell
psql "host=ubspfs.postgres.database.azure.com port=5432 dbname=postgres user=<admin_username> sslmode=require"
```

At the `postgres=>` prompt, run the following command:

```sql
CREATE OR REPLACE FUNCTION simulate_long_query()
RETURNS VOID AS $$
BEGIN
    PERFORM pg_sleep(30);
    INSERT INTO sample_data (description) VALUES ('Long query complete at ' || NOW());
END;
$$ LANGUAGE plpgsql;
```

Validate that the function was created successfully by running:

```sql
\df simulate_long_query
```

Expected output:

```text
                              List of functions
 Schema |        Name         | Result data type | Argument data types | Type 
--------+---------------------+------------------+---------------------+------
 public | simulate_long_query | void             |                     | func
(1 row)
```

Restart the Cloud Shell session, then run the following PowerShell script to: 1) launch multiple background jobs to run the function concurrently and then 2) validate the active sessions in PostgreSQL:

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

Expected output: You should see several active entries with details similar to:

```text
  pid  | usename  | application_name |  client_addr  |         backend_start         | state  |             query             
-------+----------+------------------+---------------+-------------------------------+--------+-------------------------------
 12331 | rchapler | psql             | 13.64.187.107 | 2025-03-22 15:49:08.472339+00 | active | SELECT simulate_long_query();
 12330 | rchapler | psql             | 13.64.187.107 | 2025-03-22 15:49:08.365667+00 | active | SELECT simulate_long_query();
 12329 | rchapler | psql             | 13.64.187.107 | 2025-03-22 15:49:08.309973+00 | active | SELECT simulate_long_query();
(3 rows)
```

These entries confirm that multiple processes (with their PIDs) are active, generated by running the function in parallel.



## "How Postgres could do more..."

Since PostgreSQL does not natively expose a parent PID, we must fabricate a parent-child relationship using custom annotations.  In this section, we demonstrate how to tag sessions with a fabricated Parent ID via the application name, allowing you to group related processes and selectively terminate them for process management.

### Demonstration

Restart the Cloud Shell session, then run the following PowerShell script to: 1) launch multiple sessions with two different fabricated Parent IDs, 2) validate the active sessions, and then 3) enable selective termination of a single group.

```powershell
# Set the PGPASSWORD environment variable to avoid password prompts in each background job.
$env:PGPASSWORD = "P@55word!"  # Replace with your actual password

# Launch 2 background jobs for Group 1 with fake Parent ID 'FakeParent:1001'
for ($i = 1; $i -le 2; $i++) {
    Start-Job -ScriptBlock {
        psql "host=ubspfs.postgres.database.azure.com port=5432 dbname=postgres user=rchapler sslmode=require" -c "SET application_name = 'FakeParent:1001'; SELECT simulate_long_query();"
    }
}

# Launch 2 background jobs for Group 2 with fake Parent ID 'FakeParent:2002'
for ($i = 1; $i -le 2; $i++) {
    Start-Job -ScriptBlock {
        psql "host=ubspfs.postgres.database.azure.com port=5432 dbname=postgres user=rchapler sslmode=require" -c "SET application_name = 'FakeParent:2002'; SELECT simulate_long_query();"
    }
}

# Wait for 5 seconds to allow all background jobs to start.
Start-Sleep -Seconds 5

# Validate active sessions by querying pg_stat_activity.
psql "host=ubspfs.postgres.database.azure.com port=5432 dbname=postgres user=rchapler sslmode=require" -c "SELECT pid, usename, application_name, client_addr, backend_start, state, query FROM pg_stat_activity WHERE application_name LIKE 'FakeParent:%';"
```

Expected output:

```text
  pid  | usename  | application_name | client_addr  |         backend_start         | state  |                                  query                                  
-------+----------+------------------+--------------+-------------------------------+--------+-------------------------------------------------------------------------
 43829 | rchapler | FakeParent:2002  | 13.86.153.81 | 2025-03-22 20:22:09.03268+00  | active | SET application_name = 'FakeParent:2002'; SELECT simulate_long_query();
 43828 | rchapler | FakeParent:2002  | 13.86.153.81 | 2025-03-22 20:22:08.842759+00 | active | SET application_name = 'FakeParent:2002'; SELECT simulate_long_query();
 43827 | rchapler | FakeParent:1001  | 13.86.153.81 | 2025-03-22 20:22:08.759848+00 | active | SET application_name = 'FakeParent:1001'; SELECT simulate_long_query();
 43826 | rchapler | FakeParent:1001  | 13.86.153.81 | 2025-03-22 20:22:08.521403+00 | active | SET application_name = 'FakeParent:1001'; SELECT simulate_long_query();
(4 rows)
```



To selectively terminate a group {i.e., terminate all active sessions tagged with `FakeParent:1001`}:

```powershell
psql "host=ubspfs.postgres.database.azure.com port=5432 dbname=postgres user=rchapler sslmode=require" -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE application_name = 'FakeParent:1001';"
```

Expected output:

```text
 pg_terminate_backend 
----------------------
 t
 t
(2 rows)
```
