# SQL: High Availability

## Use Case

The database administration team at a mid-sized organization must ensure data availability. Key Requirements:

* **Continuous Operations**: Support 24/7 database availability
* **Data Integrity**: Prevent data loss caused by disruptions
* **Security**: Data must be secure at all times
* **Proactive Monitoring**: Provide for quick issue detection

------------------------- ------------------------- -------------------------

## On‑Prem

### Fundamentals

#### Concept
* Ensure a database is **always available** to users
* Set up **multiple copies** on different servers, possibly in **separate geographic locations**
* Allow a secondary server to **take over quickly if the primary fails**
* Provide a **seamless user experience** with **minimal disruption**

#### Components
* **Primary Replica**: The active database handling all read/write operations
* **Secondary Replica(s)**: Up-to-date copies that can serve as failover targets

#### Setup and Configuration

* Preparation
  * Set up **at least two servers** that can communicate with each other
  * Install and **configure the necessary SQL Server features** for high availability
  
* Step-by-Step Process
  * **Designate a Primary**: Choose one server to handle all primary transactions
  * **Configure Secondaries**: Set up additional servers to receive data updates from the primary
  * **Establish Connectivity**: Ensure network and security settings allow the servers to exchange data

#### Replication Types

* **Synchronous** Replication: Data is copied to the secondary server **at the same time it is written to the primary**
  * Advantages: **Near-perfect data consistency** and **minimal risk of data loss** in the event of a failure
  * Tradeoffs: May introduce slight **latency**, as the primary waits for confirmation from the secondary
  
* **Asynchronous** Replication: Data is sent to the secondary server with a delay
  * Advantages: **Lower latency** on the primary server since it doesn't wait for secondary confirmation
  * Tradeoffs: A small **risk of data loss** if a failure occurs before the secondary catches up

  Choosing the right method will hinge on the organization’s need for **data consistency versus system performance**

#### Security Considerations

* Data Protection
  * **Encryption**: Use encryption to secure data **as it travels** between the primary and secondary servers
  * **Access** Controls: Configure strict permissions so that only authorized users can access and manage the databases
  
* Compliance and Auditing
  * Ensure that the configuration meets any **regulatory or organizational policies**
  * Set up auditing to **monitor any access or changes to the configuration**
  
* Best Practices
  * Regularly **update security patches** and **review access permissions**
  * Use **secure communication protocols** to safeguard data integrity

#### Monitoring Considerations

* Importance of Monitoring: **Early detection** of issues such as replication delays or server failures is crucial

* What to Monitor
  * Replication **latency** between primary and secondary replicas
  * Server resource usage to detect potential **bottlenecks**
  * Error logs for any signs of **communication failures** or **synchronization issues**

------------------------- -------------------------
------------------------- -------------------------

### Exercise

#### Servers

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

#### Windows Server

##### Failover Clustering

Windows Failover Clustering is **a built‐in Windows feature that lets multiple servers act as one**, automatically shifting the workload to a standby server when the primary fails.

Install the Windows Failover Clustering feature and management tools on `SERVER_PRIMARY` and `SERVER_SECONDARY`

```powershell
Install-WindowsFeature Failover-Clustering -IncludeManagementTools
```

Verify installation on `SERVER_PRIMARY` and `SERVER_SECONDARY`

```powershell
Get-WindowsFeature Failover-Clustering
```

------------------------- -------------------------

##### Cluster

A Windows cluster is **a group of interconnected servers that operate as one system**, automatically shifting workloads to a backup node when a failure occurs to keep your services running without interruption.

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

##### Node

A node is **an individual server within a Windows cluster** that actively hosts applications and automatically takes over workloads from a failing peer to ensure continuous service availability.

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

#### SQL Server

##### Always On

Always On is SQL Server’s built‑in high availability feature that synchronizes databases across multiple replicas and enables rapid failover to keep your system running when failures occur.

On `SERVER_PRIMARY`, open SQL Server Configuration Manager and then select "SQL Server Services"
  * Right-click on `SQL Server (MSSQLSERVER)` and select "Properties" from the resulting menu
  * Click the "Always On Availability Groups" tab and then check "Enable Always On Availability Groups"
  * Click "OK" and then restart `SQL Server (MSSQLSERVER)`

Repeat the process for `SERVER_SECONDARY`

------------------------- -------------------------

##### Authentication

Windows Authentication is the primary security mechanism for SQL Server endpoints—leveraging **domain credentials** for robust verification.

Certificate-Based Endpoint Authentication serves as a fallback, using digital certificates to encrypt and validate communications when AD isn't up to par.

###### Master Key

A master key is a critical encryption key in SQL Server that **secures other keys and certificates**, forming the backbone of your database’s security in high availability setups.

Create a master key on `SERVER_PRIMARY` and `SERVER_SECONDARY`
```sql
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'P@55word!'
```

###### Certificate-Based Endpoint Authentication

Create a certificate for the endpoint on `SERVER_PRIMARY` and `SERVER_SECONDARY`
```sql
CREATE CERTIFICATE Hadr_cert WITH SUBJECT = 'High Availability Disaster Recovery Endpoint Certificate'
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

Recreate the High Availability Disaster Recovery endpoint on `SERVER_PRIMARY` and `SERVER_SECONDARY`
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

##### Databases

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

##### Always On Availability Group

Always On Availability Group is a SQL Server feature that groups databases for automatic replication and seamless failover, ensuring continuous availability during server outages.

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

* "Select Data Synchronization": `Join only`

  * **Automatic Seeding**: Streams the database directly from primary to secondary. Easiest option but may silently fail in lab setups without DNS, proper permissions, or shared services.
  * **Full Database and Log Backup**: Wizard handles backup and restore using a shared location. Reliable, but requires a common file path accessible by all replicas.
  * **Join Only**: Use when you've already manually restored the database with `NORECOVERY`. The wizard just joins it to the availability group — ideal for custom or large database setups.
  * **Skip Initial Data Synchronization**: Creates the AG without any data movement. You’re fully responsible for preparing and joining each secondary manually.

       * "Validation": Confirm "Success" results
       * "Summary": Review and then click "Finish"
       * "Results": Monitor progress and confirm success

After creating the availability group and adding replicas, recreate the endpoints on both `SERVER_PRIMARY` and `SERVER_SECONDARY`. This step ensures consistent endpoint configuration across both replicas, especially if endpoints were created implicitly or are misaligned.

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

##### Troubleshooting  

Verify Configuration: On `SERVER_PRIMARY`, execute the following T‑SQL:

```sql
SELECT 
    ag.name AS AGName,
    ar.replica_server_name AS ReplicaName,
    rs.role_desc AS ReplicaRole,
    rs.operational_state_desc AS OperationalState,
    rs.connected_state_desc AS ConnectedState,
    rs.synchronization_health AS SyncState
FROM sys.availability_groups AS ag
    JOIN sys.availability_replicas AS ar ON ag.group_id = ar.group_id
    JOIN sys.dm_hadr_availability_replica_states AS rs ON ar.replica_id = rs.replica_id;
```

The numeric values in the `SyncState` column correspond to specific synchronization states:

* `0` = NOT SYNCHRONIZING: Replica isn't synchronizing data
* `1` = SYNCHRONIZING: Replica is in the process of synchronizing
* `2` = SYNCHRONIZED: Replica is synchronized (healthy, fully synchronized with the primary)
* `3` = REVERTING: Replica reverting state (rarely encountered)
* `4` = INITIALIZING: Replica is initializing data synchronization

------------------------- -------------------------

Test Failover: If the secondary replica is stuck in the "SYNCHRONIZING" state for a prolonged period, you can perform a forced manual failover:

```sql
ALTER AVAILABILITY GROUP [myAvailabilityGroup] FORCE_FAILOVER_ALLOW_DATA_LOSS
```

This command forces the secondary replica to take over as the primary. Data loss may occur if the secondary replica is not fully synchronized with the primary, particularly if it's stuck in the SYNCHRONIZING state and unable to complete synchronization.

If the databases are still synchronizing and you want to avoid data loss, perform a planned manual failover after the databases are fully synchronized. Once synchronization completes, you can execute the failover safely.

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

### Quiz

1. What is the primary goal of a high availability configuration in SQL Server?  
   A) Reducing licensing costs  
   B) Ensuring continuous operations and minimal downtime  
   C) Minimizing storage requirements  
   D) Enhancing data compression

1. Which statement best describes the primary replica in a high availability setup?  
   A) A replica dedicated to read-only queries  
   B) The active server that handles all read/write operations  
   C) A backup server that remains inactive until needed  
   D) A server used exclusively for log backups

1. What distinguishes synchronous replication from asynchronous replication?  
   A) Synchronous replication is only used for backups  
   B) Synchronous replication waits for confirmation from the secondary before committing  
   C) Asynchronous replication guarantees zero latency  
   D) Asynchronous replication uses encryption by default

1. What is a key benefit of synchronous replication?  
   A) Eliminates all latency  
   B) Ensures near-perfect data consistency with minimal risk of data loss  
   C) Increases write performance dramatically  
   D) Requires less network bandwidth

1. What is one tradeoff of using synchronous replication?  
   A) Higher risk of data loss  
   B) Slight latency because the primary waits for the secondary’s confirmation  
   C) Inconsistent data across replicas  
   D) Incompatibility with Windows Server environments

1. During the setup process, why is establishing connectivity between servers crucial?  
   A) It speeds up disk defragmentation  
   B) It ensures reliable data exchange and replication  
   C) It allows for automatic antivirus updates  
   D) It enables remote desktop functionality

1. Which of the following is an essential security measure in a high availability configuration?  
   A) Leaving firewall ports open for convenience  
   B) Implementing strict access controls and using encryption for data in transit  
   C) Disabling auditing to improve performance  
   D) Allowing unrestricted access between all network nodes

1. How do Dynamic Management Views (DMVs) assist in monitoring high availability?  
   A) They provide real-time insights into system health and performance metrics  
   B) They automate the backup process  
   C) They handle load balancing between servers  
   D) They configure network settings automatically

1. Which Windows Server feature must be installed to support failover clustering?  
   A) Active Directory Domain Services  
   B) Failover Clustering feature and management tools  
   C) Internet Information Services  
   D) Remote Desktop Services

1. Which PowerShell command is used to validate the prerequisites for a cluster before creation?  
    A) Test-NetConnection  
    B) Test-Cluster  
    C) New-Cluster  
    D) Get-Cluster

1. Why might you configure static entries in the hosts file during a high availability lab exercise?  
    A) To accelerate internet browsing  
    B) To ensure reliable name resolution when DNS is unavailable  
    C) To bypass firewall restrictions  
    D) To enhance data encryption

1. What is the purpose of having SQL Server Agent running in the high availability exercise?  
    A) To manage SQL Server services  
    B) To schedule and automate maintenance and monitoring tasks  
    C) To facilitate real-time data replication  
    D) To serve as the primary data backup mechanism

1. What primary function does the "Always On Availability Groups" feature serve?  
    A) Encrypting database backups automatically  
    B) Providing automated high availability and disaster recovery  
    C) Improving query performance through caching  
    D) Compressing data to save storage space

1. Why is certificate‑based endpoint authentication important in a high availability setup?  
    A) It authenticates users for the primary database  
    B) It secures the communication between primary and secondary replicas  
    C) It encrypts data at rest on the server  
    D) It monitors SQL query performance

1. Which T‑SQL command is used to create a new database as part of the exercise?  
    A) CREATE DATABASE  
    B) NEW DATABASE  
    C) INIT DATABASE  
    D) ADD DATABASE

1. Before performing backups in the lab, the database should be set to which recovery mode?  
    A) Simple recovery mode  
    B) Bulk-logged recovery mode  
    C) Full recovery mode  
    D) Minimal recovery mode

1. When manually restoring a database for an Always On Availability Group, which seeding mode is most appropriate?  
    A) Automatic seeding  
    B) Full backup seeding  
    C) Join Only  
    D) Skip Initial Data Synchronization

1. What is the purpose of testing failover in a high availability configuration?  
    A) To measure the speed of the primary server  
    B) To verify that the failover mechanism is correctly configured and functional  
    C) To assess network bandwidth utilization  
    D) To validate the backup schedule

1. Which method is recommended for checking the synchronization status of availability replicas?  
    A) Running a disk defragmentation check  
    B) Executing a T‑SQL query on sys.dm_hadr_availability_replica_states  
    C) Rebooting the secondary server  
    D) Updating SQL Server to the latest version

1. Why is continuous monitoring critical in a high availability environment?  
    A) It forces constant configuration changes  
    B) It detects replication delays, resource bottlenecks, and failures in real time  
    C) It adds unnecessary complexity to the system  
    D) It disables automatic updates for security reasons

---

#### Answers

1. B – Ensuring continuous operations and minimal downtime  
   *High availability aims to keep the system operational 24/7 and prevent data loss.*

1. B – The active server that handles all read/write operations  
   *The primary replica is responsible for all active transactions and serves as the central node in the configuration.*

1. B – Synchronous replication waits for confirmation from the secondary before committing  
   *This ensures data consistency but may introduce slight latency.*

1. B – Ensures near-perfect data consistency with minimal risk of data loss  
   *Synchronous replication minimizes data loss by confirming that secondary replicas have received data before committing.*

1. B – Slight latency because the primary waits for the secondary’s confirmation  
   *This waiting period can introduce delays compared to asynchronous replication.*

1. B – It ensures reliable data exchange and replication  
   *Proper connectivity is essential for continuous communication between the primary and secondary servers.*

1. B – Implementing strict access controls and using encryption for data in transit  
   *These measures protect the integrity and security of the data being replicated.*

1. A – They provide real-time insights into system health and performance metrics  
   *DMVs are used to monitor performance, replication status, and overall health of the high availability setup.*

1. B – Failover Clustering feature and management tools  
   *This feature is required to create and manage clusters for high availability scenarios.*

1. B – Test-Cluster  
    *Test-Cluster is used to validate the network, node configuration, and overall prerequisites before creating a cluster.*

1. B – To ensure reliable name resolution when DNS is unavailable  
    *Static hosts file entries provide a fallback for name resolution between servers.*

1. B – To schedule and automate maintenance and monitoring tasks  
    *SQL Server Agent is essential for automating backups, alerts, and various maintenance tasks.*

1. B – Providing automated high availability and disaster recovery  
    *Always On Availability Groups are designed to ensure minimal downtime and provide failover capabilities.*

1. B – It secures the communication between primary and secondary replicas  
    *Certificate-based authentication is critical for establishing a secure channel for data mirroring.*

1. A – CREATE DATABASE  
    *The standard T‑SQL command to create a new database is CREATE DATABASE.*

1. C – Full recovery mode  
    *Full recovery mode is necessary to ensure that all transactions are fully logged, which is essential for recovery and backup integrity.*

1. C – Join Only  
    *When the database is manually restored, the Join Only option allows it to be added to the availability group without re-seeding.*

1. B – To verify that the failover mechanism is correctly configured and functional  
    *Testing failover confirms that the high availability setup works as expected in a live failover scenario.*

1. B – Executing a T‑SQL query on sys.dm_hadr_availability_replica_states  
    *This DMV provides detailed information on the synchronization and health status of each replica.*

1. B – It detects replication delays, resource bottlenecks, and failures in real time  
    *Continuous monitoring is crucial to promptly identify and resolve issues, ensuring ongoing high availability.*

------------------------- -------------------------
------------------------- -------------------------
------------------------- -------------------------

## Azure

Azure SQL Database and Managed Instance come with built‑in high availability features. Instead of manually configuring clusters, replication, and certificate‑based endpoints, Azure provides automated failover, built‑in replication, and a simplified failover group mechanism. This means less manual configuration and a lower operational overhead while still meeting the key HA requirements.

### Fundamentals

Even though Azure SQL Database provides built‑in redundancy within a region (for example, by automatically maintaining multiple copies of your data within that same region), failover is still relevant for scenarios where:

* Region‑Level Outages: If an entire Azure region becomes unavailable (e.g., due to a major power or network disruption), local redundancy won’t protect you. A secondary server in a different region, configured through active geo‑replication or failover groups, ensures you can fail over to a completely different location.
* Disaster Recovery (DR) Requirements: Many organizations need cross‑region replication to meet compliance or RTO/RPO objectives. Setting up a failover group or active geo‑replication to another region lets you restore operations quickly if the primary region fails.
* Read Scale‑Out: Even if you’re not worried about regional failures, a failover group or geo‑replication can provide a readable secondary in another region. This offloads read workloads from your primary server and reduces latency for users in other geographical locations.

So while Azure’s built‑in redundancy does protect you against many common hardware or availability‑zone failures within a single region, configuring a secondary in a different region and enabling a failover plan is still the recommended best practice for comprehensive disaster recovery and minimal downtime.

Geo‑redundant backups and high availability serve two different, complementary purposes:

Geo‑redundant backups (via the backup storage redundancy setting) protect you at the backup level. If your primary region goes down, you can restore your database from these backups in another region. However:

You must manually restore the database, which can mean a longer Recovery Time Objective (RTO) (i.e., more downtime).

You might lose data that was committed after the last backup (so your Recovery Point Objective (RPO) may be higher).

High availability (via failover groups or active geo‑replication) provides an actively synchronized secondary database in another region. If the primary region fails, you can fail over quickly to the secondary with minimal downtime and data loss. This approach typically:

Allows near real‑time replication, so your RPO is close to zero.

Enables automatic or manual failover, so you can resume operations quickly (low RTO).

In short, geo‑redundant backups alone help you recover from a catastrophic event but at the cost of potentially more downtime and data loss, since you’d have to restore from backups. High availability (failover groups) ensures a live secondary that can take over with minimal disruption. Most mission‑critical workloads use both: they set up a high availability solution for immediate failover and also rely on geo‑redundant backups for an extra layer of disaster recovery.

------------------------- -------------------------

### Exercise

#### Servers

* Azure SQL Server `{prefix}ss-primary` in "West US" with database `{prefix}sd`
* Azure SQL Server `{prefix}ss-secondary` in "Central US"

#### Failover Group

* Navigate to the SQL Server then "Data Management" >> "Failover groups"

* Click on "Add group" and then complete the "Failover group" form:

    * **Failover group name**: `{prefix}failovergroup`

    * **Server**: `{prefix}ss-secondary`

    * **Database within the group**: Click "Configure database" and complete the resulting "Databases" form:

        * **`{prefix}sd` row**: `checked`

        * **Create standby replica**: `Yes` (and then consent to use of the secondary replica as a standby replica)

            > **If you DO select this option**:
            >
            > **Standby Replica Creation**: Azure will automatically create a read-only replica of your primary database on the secondary server. This replica will be kept in sync with the primary using active geo‑replication.
            >
            > **Role in Failover**: This standby replica is what takes over (or becomes the new primary) when you perform a failover. It ensures that your database remains available with minimal downtime and data loss.
            >
            > **Read-Only Workloads**: In addition to being the failover target, the standby replica can serve read-only queries via a dedicated listener endpoint if you choose. This offloads read workloads from the primary.
            >
            > **No Pre-existing Replica Needed**: If you already have a replica configured (for example, if you set up active geo‑replication manually), you might not need to use this option. But if not, “Create standby replica” automates that step for you.

            > **If you DO NOT select this option**:
            >
            > **No automatic replication is initiated**: The failover group will be created without automatically creating or linking a secondary copy of your database.
            >
            > **Existing Replication is Required**: If you already have an active geo‑replication relationship set up manually, the failover group might use that existing standby. Otherwise, your database remains only on the primary server until you manually configure replication.
            >
            > **Potential Impact on Failover**: Without a standby replica, a failover event won’t have a ready secondary to switch to, meaning you’d have to set up replication later or risk downtime.

            > **In short, if you don’t check this option, you’re opting out of the automated creation of a standby replica—so ensure you either already have one in place or plan to add it manually for proper high availability.**

           * Click "Select" to finalize the "Databases" form
       
       * Back on the "Failover group" page, you should now see "1 databases selected" under "Database within the group"... if so, click "Create" and monitor deployment












1. Verify High Availability:
   * After the failover group is created, use the Azure Portal to view the status and health of the failover group.
   * Optionally, connect via SSMS to simulate a failover scenario by forcing a manual failover through the portal (if testing is supported).
   * Check that the secondary server takes over with minimal disruption.

1. Implement Security Measures:
   * Review and adjust firewall rules to limit access to only authorized IP addresses.
   * Configure Virtual Network rules if necessary.
   * Enable auditing and threat detection to monitor access and any suspicious activities.

1. Set Up Monitoring:
   * Open Azure Monitor and add metrics relevant to your SQL Database—such as failover events, DTU/vCore consumption, and replication latency.
   * Configure alerts to notify you of potential issues that could affect high availability.

1. Document & Review:
   * Capture screenshots or export reports from Azure Monitor.
   * Document the configuration steps and any test results. This documentation will be useful for both internal review and customer meetings.

------------------------- -------------------------
------------------------- -------------------------

### Quiz

1. What is the primary mechanism for ensuring high availability in Azure SQL Database?  
   A) On‑prem clustering techniques  
   B) Custom T‑SQL scripts for replication  
   C) Built‑in replication and failover groups  
   D) Manual replication between servers

1. What benefit do failover groups provide in Azure SQL Database?  
   A) They require manual intervention during outages  
   B) They reduce the overall storage cost  
   C) They automatically manage regional failover  
   D) They are used solely for backup scheduling

1. How does Azure SQL Database simplify high availability compared to on‑premises solutions?  
   A) By integrating replication, failover, and monitoring into the service  
   B) By eliminating the need for any backup processes  
   C) By requiring manual configuration of clustering  
   D) By using on‑premises DMVs for status checks

1. Which aspect of the service tier (DTU/vCore) directly affects high availability performance?  
   A) The geographic region of the data center  
   B) Database size only  
   C) The color of the user interface  
   D) The level of performance impacting replication speed and failover time

1. When configuring a failover group in the Azure Portal, what is the role of the secondary server?  
   A) It serves as a standby that automatically takes over in the event of an outage  
   B) It acts as a development server only  
   C) It is used solely for running maintenance tasks  
   D) It provides additional storage for backups

1. Which tool is recommended for monitoring high availability in Azure SQL Database?  
   A) Windows Event Viewer  
   B) On‑premises DMVs  
   C) SQL Server Agent  
   D) Azure Monitor and Query Performance Insight

1. Which security best practice is recommended for Azure SQL Database HA setups?  
   A) Using default credentials for simplicity  
   B) Allowing unrestricted access for rapid deployment  
   C) Disabling firewall rules to improve performance  
   D) Configuring firewall and VNet rules, and enabling auditing

1. What does “automatic failover” mean in the context of Azure SQL Database?  
   A) It is only enabled during off-peak hours  
   B) The secondary server automatically takes over if the primary becomes unavailable  
   C) The secondary server only becomes active after manual approval  
   D) It requires a scheduled restart of the primary server

1. How does service tier selection (DTU vs vCore) impact high availability?  
   A) It controls the geographic location of the secondary server  
   B) It only changes the database pricing  
   C) It has no effect on high availability  
   D) It determines performance levels that can affect replication and failover times

1. Which step is essential when configuring high availability for Azure SQL Database through the portal?  
    A) Installing on‑premises clustering software  
    B) Writing complex PowerShell scripts  
    C) Creating a failover group that links the primary and secondary servers  
    D) Manually syncing databases via T‑SQL

1. What does “built‑in high availability” imply in Azure SQL Database?  
    A) It is achieved through third‑party add‑ons  
    B) It only applies to premium service tiers  
    C) HA features must be configured manually  
    D) High availability is integrated and managed by Azure with minimal setup

1. What is a primary advantage of using the Azure Portal for HA configuration?  
    A) It lacks monitoring capabilities  
    B) It mirrors on‑premises clustering exactly  
    C) It provides a guided and simplified configuration process  
    D) It requires extensive manual scripting

1. In Azure SQL Database, who manages the replication and failover processes?  
    A) The Azure service automatically  
    B) The database administrator manually  
    C) A third‑party vendor  
    D) On‑premises management tools

1. How is minimal downtime achieved in Azure SQL Database high availability?  
    A) Through daily scheduled maintenance windows  
    B) By disabling security features temporarily  
    C) Through automatic replication and the failover group mechanism  
    D) By requiring frequent manual intervention

1. What is the significance of configuring firewall rules in an Azure HA setup?  
    A) They are optional and not recommended  
    B) They are managed entirely by the operating system  
    C) They slow down the connection to the database  
    D) They ensure only authorized IP addresses can access the database

1. Which configuration step is simpler in Azure compared to on‑premises?  
    A) Manually running T‑SQL scripts for replication  
    B) Configuring certificate‑based endpoint authentication  
    C) Setting up failover groups via the Azure Portal  
    D) Installing a Windows Server Failover Cluster

1. How does Azure secure its high availability infrastructure?  
    A) By leaving all ports open for maximum connectivity  
    B) By requiring customers to build their own security layers  
    C) By disabling automated backups  
    D) By integrating security features such as firewall settings, VNets, and auditing

1. Which tool would you use to set alerts for HA issues in Azure SQL Database?  
    A) DMVs from on‑prem installations  
    B) Windows PowerShell only  
    C) SQL Server Management Studio exclusively  
    D) Azure Monitor

1. What is the purpose of a failover group in Azure SQL Database?  
    A) To enable read-only replicas for development  
    B) To manually copy databases between servers  
    C) To reduce database size  
    D) To automatically manage failover between the primary and secondary databases

1. Which statement best describes the overall high availability strategy in Azure SQL Database?  
    A) It is a streamlined, built‑in solution that automates replication, failover, and monitoring  
    B) It exactly replicates on‑premises clustering methods  
    C) It is a complex, manually configured solution  
    D) It only supports low‑availability configurations

------------------------- -------------------------

#### Answers

1. C – Built‑in replication and failover groups  
   *Azure SQL Database uses built‑in replication and managed failover groups to ensure high availability without manual clustering.*

1. C – They automatically manage regional failover  
   *Failover groups in Azure automatically manage regional failover, minimizing downtime during outages.*

1. A – By integrating replication, failover, and monitoring into the service  
   *Azure SQL Database simplifies HA by embedding these features directly into the service, reducing manual configuration.*

1. D – The level of performance impacting replication speed and failover time  
   *Service tier performance directly affects replication and failover times, which are critical for high availability.*

1. A – It serves as a standby that automatically takes over in the event of an outage  
   *The secondary server in a failover group is maintained as a standby that takes over automatically if the primary fails.*

1. D – Azure Monitor and Query Performance Insight  
   *These tools are designed to track performance metrics and alerts for high availability in Azure SQL Database.*

1. D – Configuring firewall and VNet rules, and enabling auditing  
   *Implementing these security measures is essential to protect your database and ensure only authorized access.*

1. B – The secondary server automatically takes over if the primary becomes unavailable  
   *Automatic failover means that the secondary server is ready to assume the primary role without manual intervention.*

1. D – It determines performance levels that can affect replication and failover times  
   *Choosing the appropriate service tier impacts the speed and reliability of replication and failover processes.*

1. C – Creating a failover group that links the primary and secondary servers  
    *This is the critical step in setting up HA in Azure, as it connects your servers for automatic failover.*

1. D – High availability is integrated and managed by Azure with minimal setup  
    *Azure’s HA features are built into the platform, reducing the need for manual configuration.*

1. C – It provides a guided and simplified configuration process  
    *The Azure Portal offers an intuitive interface that simplifies the setup of high availability.*

1. A – The Azure service automatically  
    *Azure manages replication and failover processes automatically, reducing administrative overhead.*

1. C – Through automatic replication and the failover group mechanism  
    *This integrated approach minimizes downtime by ensuring a seamless switch to the standby server.*

1. D – They ensure only authorized IP addresses can access the database  
    *Proper firewall configuration is essential to restrict access and enhance security in an HA setup.*

1. C – Setting up failover groups via the Azure Portal  
    *This step is much simpler in Azure compared to on‑premises clustering or manual configurations.*

1. D – By integrating security features such as firewall settings, VNets, and auditing  
    *Azure’s infrastructure includes robust, built‑in security measures that protect the high availability environment.*

1. D – Azure Monitor  
    *Azure Monitor is the recommended tool to set alerts and track performance for HA issues.*

1. D – To automatically manage failover between the primary and secondary databases  
    *Failover groups are designed to ensure automatic management of failover in case of an outage.*

1. A – It is a streamlined, built‑in solution that automates replication, failover, and monitoring  
    *This answer best describes Azure’s overall strategy, which simplifies high availability through automation.*
