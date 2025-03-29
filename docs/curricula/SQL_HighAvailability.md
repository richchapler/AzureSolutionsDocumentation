# SQL: High Availability

## Basics

### Use Case

The database administration team at a mid-sized organization must ensure data availability. Key Requirements:

* Continuous Operations: Support 24/7 database availability
* Data Integrity: Prevent data loss caused by disruptions
* Security: Data must be secure at all times
* Proactive Monitoring: Provide for quick issue detection

------------------------- ------------------------- -------------------------

### On‑Prem

#### Discussion

##### Fundamentals

###### Concept
* Ensure a database is always available to users
* Set up multiple copies on different servers, possibly in separate geographic locations
* Allow a secondary server to take over quickly if the primary fails
* Provide a seamless user experience with minimal disruption

###### Components
* Primary Replica: The active database handling all read/write operations
* Secondary Replica(s): Up-to-date copies that can serve as failover targets

###### Benefits
* Minimal downtime during failures
* Reduced risk of data loss through continuous data synchronization

------------------------- -------------------------

##### Setup and Configuration

* Preparation
  * Set up at least two servers that can communicate with each other
  * Install and configure the necessary SQL Server features for high availability
* Step-by-Step Process
  * Designate a Primary: Choose one server to handle all primary transactions
  * Configure Secondaries: Set up additional servers to receive data updates from the primary
  * Establish Connectivity: Ensure network and security settings allow the servers to exchange data
* Outcome
  * A working environment where the primary server’s data is continuously replicated to secondary servers
* Why It Matters
  * Establishes the foundation for continuous operations and lays the groundwork for advanced features later

------------------------- -------------------------

##### Synchronous vs. Asynchronous

* Synchronous Replication
  * Definition: Data is copied to the secondary server at the same time it is written to the primary
  * Advantages:
    * Near-perfect data consistency
    * Minimal risk of data loss in the event of a failure
  * Tradeoffs:
    * May introduce slight latency, as the primary waits for confirmation from the secondary
* Asynchronous Replication
  * Definition: Data is sent to the secondary server with a delay
  * Advantages
    * Lower latency on the primary server since it doesn't wait for secondary confirmation
  * Tradeoffs
    * A small risk of data loss if a failure occurs before the secondary catches up
* Choosing the Right Method
  * Depends on the organization’s need for data consistency versus system performance

------------------------- -------------------------

##### Security Considerations

* Data Protection
  * Encryption: Use encryption to secure data as it travels between the primary and secondary servers
  * Access Controls: Configure strict permissions so that only authorized users can access and manage the databases
* Compliance and Auditing
  * Ensure that the configuration meets any regulatory or organizational policies
  * Set up auditing to monitor any access or changes to the configuration
* Best Practices
  * Regularly update security patches and review access permissions
  * Use secure communication protocols to safeguard data integrity

------------------------- -------------------------

##### Monitoring Considerations

* Importance of Monitoring: Early detection of issues such as replication delays or server failures is crucial.

* Tools and Techniques:
  * Dynamic Management Views (DMVs): Use DMVs to query system health and performance statistics
  * Performance Dashboards: Visual tools that display real-time metrics on replication status, CPU usage, and memory consumption

* What to Monitor
  * Replication latency between primary and secondary replicas
  * Server resource usage to detect potential bottlenecks
  * Error logs for any signs of communication failures or synchronization issues

* Outcome: Continuous assurance that the high availability setup is functioning optimally and that issues are addressed promptly

------------------------- -------------------------
------------------------- -------------------------
------------------------- -------------------------

#### Exercise

##### Pre-Requisites

Two machines (`SERVER_PRIMARY` and `SERVER_SECONDARY`), each with:

* Windows Server
    * PowerShell 7.x
    * Firewall + Inbound Rules:
      * SQL Server TCP 1433
      * SQL Server Availability Group Endpoint 5022
    * (Optional) Static entries in `C:\Windows\System32\drivers\etc\hosts` if DNS name resolution is unavailable

* SQL Server
    * SQL Server Agent started (`Start-Service -Name SQLSERVERAGENT`)

* SQL Server Management Studio

------------------------- -------------------------
------------------------- -------------------------

##### Windows Server

###### Failover Clustering

Install Failover Clustering feature and management tools on `SERVER_PRIMARY` and `SERVER_SECONDARY`

```powershell
Install-WindowsFeature Failover-Clustering -IncludeManagementTools
```

Verify installation on `SERVER_PRIMARY` and `SERVER_SECONDARY`

```powershell
Get-WindowsFeature Failover-Clustering
```

------------------------- -------------------------

###### Cluster

Validate prerequisites before creating the cluster on `SERVER_PRIMARY` and `SERVER_SECONDARY`

```powershell
Test-Cluster -Node localhost -Verbose
```

Create a new cluster on `SERVER_PRIMARY`

```powershell
# Get the first interface with a default gateway and an IPv4 address
$ipConfig = Get-NetIPConfiguration | Where-Object { $_.IPv4DefaultGateway -and $_.IPv4Address } | Select-Object -First 1

if (-not $ipConfig) {
    Write-Error "No network interface with a default gateway was found."
    return
}

# Extract the IPv4 address (as a string) from the first available entry
$primaryIP = $ipConfig.IPv4Address[0].IPAddress
# Split into octets and build the subnet (assuming a /24 subnet)
$subnetParts = $primaryIP.Split('.')
$subnet = "$($subnetParts[0]).$($subnetParts[1]).$($subnetParts[2])"

# Define a starting host number (adjust if needed) and find the first available IP on that subnet
$firstHost = (2..254) | Where-Object { -not (Test-Connection -Count 1 -Quiet -Destination "$subnet.$_") } | Select-Object -First 1
if (-not $firstHost) {
    Write-Error "No available IP address found in the subnet $subnet.0/24."
    return
}

$IP = "$subnet.$firstHost"
Write-Output "Using available IP: $IP"

# Create the cluster using the identified available IP
New-Cluster -Name "myCluster" -Node localhost -StaticAddress $IP
```

Open Failover Cluster Manager on `SERVER_PRIMARY`, connect to your cluster, and confirm cluster status = "Online"

------------------------- -------------------------

###### Node

Add node to cluster on `SERVER_SECONDARY`
```powershell
Add-ClusterNode -Name "myCluster"
```

Verify that SERVER_SECONDARY has joined the cluster by running (on either server):

```powershell
Get-ClusterNode
```

------------------------- -------------------------
------------------------- -------------------------

##### SQL Server

###### Always On

On `SERVER_PRIMARY`, open SQL Server Configuration Manager and then select "SQL Server Services"
  * Right-click on `SQL Server (MSSQLSERVER)` and select "Properties" from the resulting menu
  * Click the "Always On Availability Groups" tab and then check "Enable Always On Availability Groups"
  * Click "OK" and then restart `SQL Server (MSSQLSERVER)`

Repeat the process for `SERVER_SECONDARY`

------------------------- -------------------------

###### Certificate-Based Endpoint Authentication

Create a master key on `SERVER_PRIMARY` and `SERVER_SECONDARY`

```sql
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'P@55word!'
```

Create a certificate for the endpoint on `SERVER_PRIMARY` and `SERVER_SECONDARY`

```sql
CREATE CERTIFICATE Hadr_cert WITH SUBJECT = 'HADR Endpoint Certificate'
```

Backup the certificate on `SERVER_PRIMARY`

```sql
BACKUP CERTIFICATE Hadr_cert TO FILE = 'C:\Temp\Hadr_cert.cer'
WITH PRIVATE KEY (
    FILE = 'C:\Temp\Hadr_cert_privatekey.pvk',
    ENCRYPTION BY PASSWORD = 'P@55word!'
)
```

Copy `Hadr_cert.cer`, `Hadr_cert.pvk`, and `Hadr_cert_privatekey.pvk` to c:\Temp on `SERVER_SECONDARY`

Restore the certificate on `SERVER_SECONDARY`

```sql
CREATE CERTIFICATE Hadr_cert FROM FILE = 'C:\Temp\Hadr_cert.cer'
WITH PRIVATE KEY (
    FILE = 'C:\Temp\Hadr_cert_privatekey.pvk',
    DECRYPTION BY PASSWORD = 'P@55word!'
)
```

Recreate the HADR endpoint on `SERVER_PRIMARY` and `SERVER_SECONDARY`

```sql
DROP ENDPOINT Hadr_endpoint;
GO
CREATE ENDPOINT Hadr_endpoint  
    STATE = STARTED  
    AS TCP (LISTENER_PORT = 5022)  
    FOR DATA_MIRRORING (
        ROLE = ALL, 
        AUTHENTICATION = CERTIFICATE Hadr_cert, 
        ENCRYPTION = REQUIRED ALGORITHM AES
    )
```

------------------------- -------------------------

###### Prepare Databases

Create "c:\Temp" directories on both Windows Servers

On `SERVER_PRIMARY`, open SQL Server Management Studio and connect to the database engine

Create a database
```sql
CREATE DATABASE myDatabase ON (NAME = myDatabase_Data, FILENAME = 'C:\Temp\myDatabase.mdf')
```

Verify the database was created and is online:
```sql
SELECT name FROM sys.databases WHERE name = 'myDatabase'
```

Ensure the database is in full recovery mode
```sql
ALTER DATABASE myDatabase SET RECOVERY FULL
```

Backup the database
```sql
BACKUP DATABASE myDatabase TO DISK = 'C:\Temp\myDatabase.bak' WITH INIT
BACKUP LOG myDatabase TO DISK = 'C:\Temp\myDatabase_log.bak' WITH INIT;
```

On `SERVER_SECONDARY`, copy `myDatabase.bak` and `myDatabase_log.bak` to c:\Temp and then restore the backup
```sql
RESTORE DATABASE myDatabase FROM DISK = 'C:\Temp\myDatabase.bak' WITH NORECOVERY,
    MOVE 'myDatabase_Data' TO 'C:\Temp\myDatabase.mdf',
    MOVE 'myDatabase_Log' TO 'C:\Temp\myDatabase.ldf';

RESTORE LOG myDatabase FROM DISK = 'C:\Temp\myDatabase_log.bak' WITH NORECOVERY;
```

------------------------- -------------------------

###### Always On Availability Group

On `SERVER_PRIMARY`, open SQL Server Management Studio and connect to the database engine:

* "Object Explorer": Right-click "Always On High Availability" and then "New Availability Group Wizard" in the resulting menu

* Complete the "New Availability Group" wizard:

  * "Introduction": Skip

  * "Specify Options"

    * Enter "Availability group name" `myAvailabilityGroup` 

    * Choose "Cluster type": `Windows Server Failover Cluster`

      * Default value... for health checks and failovers  
    
      * Other options:
        * EXTERNAL: For third-party cluster managers (e.g., Pacemaker on Linux)... useful if WSFC isn’t feasible  
    
        * NONE: Creates a read-scale, cluster-less Availability Group... no automatic failover, suitable for offloading reads to secondaries
    
    * "Database Level Health Detection": `unchecked`  
      * Checks the health of each database individually  
      * Allows more granular failover if one database becomes unhealthy  
      * Useful if multiple databases are in the same group
    * "Per Database DTC Support": `unchecked`
      * Manages distributed transactions on a per-database basis  
      * Enable only if your applications rely on DTC across multiple databases
    * "Contained": `unchecked`
      * Stores users/logins inside the database  
      * Easier to move the database to another instance without re-creating logins

* "Select Databases": Check the `myDatabase` database (and confirms "Meets prerequisites")

* "Select Replicas": Click "Add Replica" and complete "Connect to Server" form for `SERVER_SECONDARY`
     * "Replicas" tab >> "Availability Replicas" matrix:
       * Automatic Failover: `Disabled` (default)
         * Enabled: The cluster automatically fails over to a secondary replica if the primary becomes unavailable. Requires synchronous commit mode and adds slight latency.
         * Disabled: Failover must be performed manually. Provides more control and avoids unintentional failover, but may increase downtime during failures.
       * Availability Mode: `Asynchronous-commit mode` (default)
           * Synchronous-commit: Transactions wait for the secondary to confirm before committing, minimizing data loss but adding latency.
           * Asynchronous-commit: Transactions commit immediately without waiting for the secondary, improving performance but risking data loss during failover.
       * Readable Secondary: `No` (default)
           * No: All read/write activity stays on the primary, ensuring strict consistency and simplifying configuration.
           * Yes / Read-intent only: Secondary can handle read-only queries to reduce load on the primary — with optional enforcement using `ApplicationIntent=ReadOnly`.

"Select Data Synchronization": `Join only`

* Automatic Seeding: Streams the database directly from primary to secondary. Easiest option but may silently fail in lab setups without DNS, proper permissions, or shared services.
* Full Database and Log Backup: Wizard handles backup and restore using a shared location. Reliable, but requires a common file path accessible by all replicas.
* Join Only: Use when you've already manually restored the database with `NORECOVERY`. The wizard just joins it to the availability group — ideal for custom or large database setups.
* Skip Initial Data Synchronization: Creates the AG without any data movement. You’re fully responsible for preparing and joining each secondary manually.

     * "Validation": Confirm "Success" results
     * "Summary": Review and then click "Finish"
     * "Results": Monitor progress and confirm success

* After creating the availability group and adding replicas, recreate the endpoints on both `SERVER_PRIMARY` and `SERVER_SECONDARY`. This step ensures consistent endpoint configuration across both replicas, especially if endpoints were created implicitly or are misaligned.

  ```sql
  DROP ENDPOINT Hadr_endpoint;
  GO
  CREATE ENDPOINT Hadr_endpoint  
      STATE = STARTED  
      AS TCP (LISTENER_PORT = 5022)  
      FOR DATA_MIRRORING (ROLE = ALL, AUTHENTICATION = WINDOWS NEGOTIATE, ENCRYPTION = REQUIRED ALGORITHM AES);
  ```

After adding the secondary replica, ensure its seeding mode is set to `AUTOMATIC` (required for automatic seeding):

```sql
ALTER AVAILABILITY GROUP myAvailabilityGroup 
MODIFY REPLICA ON 'SERVER_SECONDARY' 
WITH (SEEDING_MODE = AUTOMATIC);
```

------------------------- -------------------------------------------------- -------------------------

###### Troubleshooting  

**Verify Configuration**: On `SERVER_PRIMARY`, execute the following T‑SQL:

```sql
SELECT 
  ag.name AS AGName,
  ar.replica_server_name AS ReplicaName,
  rs.role_desc AS ReplicaRole,
  rs.operational_state_desc AS OperationalState,
  rs.connected_state_desc AS ConnectedState,
  rs.synchronization_health AS SyncState
FROM sys.availability_groups AS ag
JOIN sys.availability_replicas AS ar 
  ON ag.group_id = ar.group_id
JOIN sys.dm_hadr_availability_replica_states AS rs 
  ON ar.replica_id = rs.replica_id;
```

The numeric values in the `SyncState` column correspond to specific synchronization states:

* `0` = NOT SYNCHRONIZING: Replica isn't synchronizing data
* `1` = SYNCHRONIZING: Replica is in the process of synchronizing
* `2` = SYNCHRONIZED: Replica is synchronized (healthy, fully synchronized with the primary)
* `3` = REVERTING: Replica reverting state (rarely encountered)
* `4` = INITIALIZING: Replica is initializing data synchronization

------------------------- -------------------------

**Verify Synchronization**: On `SERVER_PRIMARY`, execute the following T‑SQL:

 ```sql
 SELECT 
     ag.name AS AGName,
     ar.replica_server_name AS ReplicaName,
     drs.database_id AS DatabaseID,
     drs.synchronization_state_desc AS SyncState,
     drs.synchronization_health_desc AS SyncHealth
 FROM sys.availability_groups AS ag
 JOIN sys.availability_replicas AS ar 
     ON ag.group_id = ar.group_id
 JOIN sys.dm_hadr_database_replica_states AS drs
     ON ar.replica_id = drs.replica_id;
 ```

   This will give you the synchronization status of each database in the availability group. If the status is `NOT SYNCHRONIZING`, that’s the root of the issue.

------------------------- -------------------------

**Test Failover**: If the secondary replica is stuck in the "SYNCHRONIZING" state for a prolonged period, you can perform a forced manual failover:

```sql
ALTER AVAILABILITY GROUP [myAvailabilityGroup] FORCE_FAILOVER_ALLOW_DATA_LOSS
```

This command forces the secondary replica to take over as the primary. **Data loss** may occur if the secondary replica is not fully synchronized with the primary, particularly if it's stuck in the **SYNCHRONIZING** state and unable to complete synchronization.

If the databases are still synchronizing and you want to avoid data loss, perform a **planned manual failover** after the databases are fully synchronized. Once synchronization completes, you can execute the failover safely.

   ```sql
   ALTER AVAILABILITY GROUP [myAvailabilityGroup] FAILOVER;
   ```

Verify the Role Change: Run the following query on the new primary (which was the former secondary) to verify that it is now the primary replica:

```sql
SELECT ag.name AS AGName,
    ar.replica_server_name AS ReplicaName,
    rs.role_desc AS ReplicaRole,
    rs.operational_state_desc AS OperationalState,
    rs.connected_state_desc AS ConnectedState,
    rs.synchronization_health AS SyncState
FROM sys.availability_groups AS ag
    JOIN sys.availability_replicas AS ar ON ag.group_id = ar.group_id
    JOIN sys.dm_hadr_availability_replica_states AS rs ON ar.replica_id = rs.replica_id;
```

------------------------- -------------------------

Resynchronize the Secondary Replica: Now that the failover is complete, we need to make sure that the new secondary replica is synchronized properly. You can try running the following command to start synchronization:

```sql
ALTER AVAILABILITY GROUP [myAvailabilityGroup]
    MODIFY REPLICA ON 'PRIMARY'
    WITH (SEEDING_MODE = AUTOMATIC);
```

This will ensure the new secondary replica is automatically seeded if needed.

------------------------- -------------------------
------------------------- -------------------------
------------------------- -------------------------

#### Quiz

1. **What is the primary goal of a high availability configuration in SQL Server?**  
   A) Reducing licensing costs  
   B) Ensuring continuous operations and minimal downtime  
   C) Minimizing storage requirements  
   D) Enhancing data compression

2. **Which statement best describes the primary replica in a high availability setup?**  
   A) A replica dedicated to read-only queries  
   B) The active server that handles all read/write operations  
   C) A backup server that remains inactive until needed  
   D) A server used exclusively for log backups

3. **What distinguishes synchronous replication from asynchronous replication?**  
   A) Synchronous replication is only used for backups  
   B) Synchronous replication waits for confirmation from the secondary before committing  
   C) Asynchronous replication guarantees zero latency  
   D) Asynchronous replication uses encryption by default

4. **What is a key benefit of synchronous replication?**  
   A) Eliminates all latency  
   B) Ensures near-perfect data consistency with minimal risk of data loss  
   C) Increases write performance dramatically  
   D) Requires less network bandwidth

5. **What is one tradeoff of using synchronous replication?**  
   A) Higher risk of data loss  
   B) Slight latency because the primary waits for the secondary’s confirmation  
   C) Inconsistent data across replicas  
   D) Incompatibility with Windows Server environments

6. **During the setup process, why is establishing connectivity between servers crucial?**  
   A) It speeds up disk defragmentation  
   B) It ensures reliable data exchange and replication  
   C) It allows for automatic antivirus updates  
   D) It enables remote desktop functionality

7. **Which of the following is an essential security measure in a high availability configuration?**  
   A) Leaving firewall ports open for convenience  
   B) Implementing strict access controls and using encryption for data in transit  
   C) Disabling auditing to improve performance  
   D) Allowing unrestricted access between all network nodes

8. **How do Dynamic Management Views (DMVs) assist in monitoring high availability?**  
   A) They provide real-time insights into system health and performance metrics  
   B) They automate the backup process  
   C) They handle load balancing between servers  
   D) They configure network settings automatically

9. **Which Windows Server feature must be installed to support failover clustering?**  
   A) Active Directory Domain Services  
   B) Failover Clustering feature and management tools  
   C) Internet Information Services  
   D) Remote Desktop Services

10. **Which PowerShell command is used to validate the prerequisites for a cluster before creation?**  
    A) Test-NetConnection  
    B) Test-Cluster  
    C) New-Cluster  
    D) Get-Cluster

11. **Why might you configure static entries in the hosts file during a high availability lab exercise?**  
    A) To accelerate internet browsing  
    B) To ensure reliable name resolution when DNS is unavailable  
    C) To bypass firewall restrictions  
    D) To enhance data encryption

12. **What is the purpose of having SQL Server Agent running in the high availability exercise?**  
    A) To manage SQL Server services  
    B) To schedule and automate maintenance and monitoring tasks  
    C) To facilitate real-time data replication  
    D) To serve as the primary data backup mechanism

13. **What primary function does the "Always On Availability Groups" feature serve?**  
    A) Encrypting database backups automatically  
    B) Providing automated high availability and disaster recovery  
    C) Improving query performance through caching  
    D) Compressing data to save storage space

14. **Why is certificate‑based endpoint authentication important in a high availability setup?**  
    A) It authenticates users for the primary database  
    B) It secures the communication between primary and secondary replicas  
    C) It encrypts data at rest on the server  
    D) It monitors SQL query performance

15. **Which T‑SQL command is used to create a new database as part of the exercise?**  
    A) CREATE DATABASE  
    B) NEW DATABASE  
    C) INIT DATABASE  
    D) ADD DATABASE

16. **Before performing backups in the lab, the database should be set to which recovery mode?**  
    A) Simple recovery mode  
    B) Bulk-logged recovery mode  
    C) Full recovery mode  
    D) Minimal recovery mode

17. **When manually restoring a database for an Always On Availability Group, which seeding mode is most appropriate?**  
    A) Automatic seeding  
    B) Full backup seeding  
    C) Join Only  
    D) Skip Initial Data Synchronization

18. **What is the purpose of testing failover in a high availability configuration?**  
    A) To measure the speed of the primary server  
    B) To verify that the failover mechanism is correctly configured and functional  
    C) To assess network bandwidth utilization  
    D) To validate the backup schedule

19. **Which method is recommended for checking the synchronization status of availability replicas?**  
    A) Running a disk defragmentation check  
    B) Executing a T‑SQL query on sys.dm_hadr_availability_replica_states  
    C) Rebooting the secondary server  
    D) Updating SQL Server to the latest version

20. **Why is continuous monitoring critical in a high availability environment?**  
    A) It forces constant configuration changes  
    B) It detects replication delays, resource bottlenecks, and failures in real time  
    C) It adds unnecessary complexity to the system  
    D) It disables automatic updates for security reasons

---

##### Answers

1. **B – Ensuring continuous operations and minimal downtime**  
   *High availability aims to keep the system operational 24/7 and prevent data loss.*

2. **B – The active server that handles all read/write operations**  
   *The primary replica is responsible for all active transactions and serves as the central node in the configuration.*

3. **B – Synchronous replication waits for confirmation from the secondary before committing**  
   *This ensures data consistency but may introduce slight latency.*

4. **B – Ensures near-perfect data consistency with minimal risk of data loss**  
   *Synchronous replication minimizes data loss by confirming that secondary replicas have received data before committing.*

5. **B – Slight latency because the primary waits for the secondary’s confirmation**  
   *This waiting period can introduce delays compared to asynchronous replication.*

6. **B – It ensures reliable data exchange and replication**  
   *Proper connectivity is essential for continuous communication between the primary and secondary servers.*

7. **B – Implementing strict access controls and using encryption for data in transit**  
   *These measures protect the integrity and security of the data being replicated.*

8. **A – They provide real-time insights into system health and performance metrics**  
   *DMVs are used to monitor performance, replication status, and overall health of the high availability setup.*

9. **B – Failover Clustering feature and management tools**  
   *This feature is required to create and manage clusters for high availability scenarios.*

10. **B – Test-Cluster**  
    *Test-Cluster is used to validate the network, node configuration, and overall prerequisites before creating a cluster.*

11. **B – To ensure reliable name resolution when DNS is unavailable**  
    *Static hosts file entries provide a fallback for name resolution between servers.*

12. **B – To schedule and automate maintenance and monitoring tasks**  
    *SQL Server Agent is essential for automating backups, alerts, and various maintenance tasks.*

13. **B – Providing automated high availability and disaster recovery**  
    *Always On Availability Groups are designed to ensure minimal downtime and provide failover capabilities.*

14. **B – It secures the communication between primary and secondary replicas**  
    *Certificate-based authentication is critical for establishing a secure channel for data mirroring.*

15. **A – CREATE DATABASE**  
    *The standard T‑SQL command to create a new database is CREATE DATABASE.*

16. **C – Full recovery mode**  
    *Full recovery mode is necessary to ensure that all transactions are fully logged, which is essential for recovery and backup integrity.*

17. **C – Join Only**  
    *When the database is manually restored, the Join Only option allows it to be added to the availability group without re-seeding.*

18. **B – To verify that the failover mechanism is correctly configured and functional**  
    *Testing failover confirms that the high availability setup works as expected in a live failover scenario.*

19. **B – Executing a T‑SQL query on sys.dm_hadr_availability_replica_states**  
    *This DMV provides detailed information on the synchronization and health status of each replica.*

20. **B – It detects replication delays, resource bottlenecks, and failures in real time**  
    *Continuous monitoring is crucial to promptly identify and resolve issues, ensuring ongoing high availability.*

------------------------- -------------------------
------------------------- -------------------------
------------------------- -------------------------

### Azure

* Built‑in High Availability Features in Azure SQL Database:
  * Explore how failover groups and built‑in replication ensure seamless connectivity.
* Impact of Service Tiers (DTU/vCore) on HA:
  * Understand how service tier selection affects performance and availability.
* Initial Configuration Using the Azure Portal:
  * Guided steps for configuring HA settings through the portal.
* Security Considerations:
  * Implement best practices such as firewall configurations, network isolation, and auditing within Azure HA setups.
* Monitoring Considerations:
  * Leverage Azure Monitor and Query Performance Insight to track HA performance and health.
* Quiz:
  * Short assessments to verify your knowledge of service tiers and initial HA configuration.
  * Answers: Detailed answer keys for each quiz question.
