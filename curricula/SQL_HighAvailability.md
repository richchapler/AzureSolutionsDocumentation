# SQL: High Availability

## Use Case

The database administration team at a mid-sized organization must ensure data availability. Key requirements:

* **Continuous Operations**: Support 24/7 database availability
* **Data Integrity**: Prevent data loss caused by disruptions
* **Security**: Data must be secure at all times
* **Proactive Monitoring**: Provide for quick issue detection

------------------------- -------------------------

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

------------------------- -------------------------
------------------------- -------------------------

### Troubleshooting

Consider the following framework a starting point for troubleshooting; there will undoubtedly be items not yet included, but these are the most common:

#### What's Happening?
Start by identifying symptoms using logs, dashboards, monitoring tools, and user reports:

* **Error messages**: connection timeout, “database unavailable”, login failures  
* **Application impact**: data appears stale, ETL or replication jobs stalled  
* **Dashboard indicators**: "Not Synchronizing", "Restoring", "Disconnected"  
* **Failover attempts**: stuck in transition, replica not joining, config incomplete  
* **General behaviors**: primary unreachable, failover not triggering, listener not resolving  

------------------------- -------------------------

#### SQL Server Issues?

SQL Server problems are often the root of synchronization or connectivity failures in an availability group. Check each of the following areas:

##### *SQL Server service not started or unresponsive*
* Confirm that the SQL Server service is running on both primary and secondary nodes  
* Use SQL Server Configuration Manager or `Get-Service` in PowerShell to check service status  
* If the service fails to start, review the Windows Event Log for startup errors  
* Validate that the correct instance name is used when connecting  

##### *Database stuck in `Restoring` state*
* Indicates that the secondary database hasn’t fully joined the availability group  
* Check that the database was restored using `WITH NORECOVERY`  
* Validate that the database is in FULL recovery mode  
* Verify recent log backups were restored prior to the join  
* If using automatic seeding, confirm seeding mode is set correctly and no errors are present in error logs  

##### *AG status shows "Not Synchronizing"*
* Run this query to check replica health and synchronization state:

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

* `SyncState` values:  
  * `0` = NOT SYNCHRONIZING  
  * `1` = SYNCHRONIZING  
  * `2` = SYNCHRONIZED  
  * `3` = REVERTING  
  * `4` = INITIALIZING  

* Common causes of "Not Synchronizing":  
  * Network or endpoint connectivity issues  
  * Secondary node not joined correctly  
  * Database not in FULL recovery mode  
  * Automatic seeding misconfigured  

##### *Causes: log backup missing, endpoint unreachable, high network latency*
* Missing log backups can prevent secondaries from catching up  
* Use `sys.fn_hadr_backup_is_preferred_replica` to confirm correct log backup target  
* Endpoint failures (e.g., invalid certs, misconfigured ports) can break synchronization  
* Run network tests (ping, port scans) to confirm connectivity between nodes  
* Check for unusual latency or packet loss — this can degrade performance in synchronous mode  

------------------------- -------------------------

#### Windows Server Failover Cluster?

Issues at the cluster level can interrupt failover operations or cause replicas to drop out of synchronization. Validate each of the following:

##### *Cluster service not running on one or more nodes*
* Open Failover Cluster Manager and check node status  
* Run `Get-ClusterNode` in PowerShell to verify each node is `Up`  
* If stopped, attempt to restart the Cluster Service and check for underlying issues (event logs, permissions, services dependencies)  

##### *Node unavailable due to reboot, NIC failure, or network isolation*
* Confirm all nodes are online and can ping each other  
* Review NIC settings and verify the cluster network is marked for "Cluster and Client" communication  
* Check if a recent reboot or patch cycle impacted availability  

##### *Quorum not met or misconfigured*
* If quorum is lost, the cluster will go offline or enter degraded mode  
* Validate quorum configuration in Failover Cluster Manager  
* Use `Get-ClusterQuorum` to confirm current mode and witness status  
* Review cluster logs for events related to node voting or witness failure  

##### *Listener DNS not resolving or IP not reachable*
* Use `nslookup` to verify the listener name resolves correctly on all nodes and client machines  
* Confirm the listener IP address is not in conflict and is present in the cluster resource list  
* Ensure all subnets used by the cluster have been properly added to the cluster configuration  

------------------------- -------------------------

#### Listener / Connectivity?

Connectivity issues are often external to SQL Server but critical to the success of failover and client redirection. Investigate the following:

##### *Listener name not resolving in DNS*
* Validate DNS record exists for the listener name and maps to the correct IP  
* Flush and re-register DNS on affected servers or clients using `ipconfig /flushdns` and `registerdns`  
* Confirm there's no name resolution conflict with another object (e.g., a stale A record or CNAME collision)  

##### *IP unreachable due to firewall, subnet config, or route issues*
* Check that firewall rules allow inbound traffic on TCP 1433 and any listener IPs  
* Validate subnets are routable across nodes, especially if using `MultiSubnetFailover`  
* Ensure no recent changes to NSGs, routing tables, or firewall appliances are blocking traffic  

##### *Application uses incorrect connection string or missing `MultiSubnetFailover`*
* Confirm connection strings in applications and scripts are correct  
* For multi-subnet configurations, the connection string should include `MultiSubnetFailover=True`  
* Missing this option may cause long timeouts during failover scenarios  

------------------------- -------------------------
------------------------- -------------------------

### Quiz

1. What is the primary goal of a high availability configuration in SQL Server?  
   A) Minimizing storage requirements  
   B) Enhancing data compression  
   C) Ensuring continuous operations and minimal downtime  
   D) Reducing licensing costs  

2. Which statement best describes the primary replica in a high availability setup?  
   A) A backup server that remains inactive until needed  
   B) A server used exclusively for log backups  
   C) The active server that handles all read/write operations  
   D) A replica dedicated to read-only queries  

3. What distinguishes synchronous replication from asynchronous replication?  
   A) Asynchronous replication guarantees zero latency  
   B) Synchronous replication waits for confirmation from the secondary before committing  
   C) Synchronous replication is only used for backups  
   D) Asynchronous replication uses encryption by default  

4. What is a key benefit of synchronous replication?  
   A) Increases write performance dramatically  
   B) Requires less network bandwidth  
   C) Ensures near-perfect data consistency with minimal risk of data loss  
   D) Eliminates all latency  

5. What is one tradeoff of using synchronous replication?  
   A) Inconsistent data across replicas  
   B) Higher risk of data loss  
   C) Slight latency because the primary waits for the secondary’s confirmation  
   D) Incompatibility with Windows Server environments  

6. During the setup process, why is establishing connectivity between servers crucial?  
   A) It speeds up disk defragmentation  
   B) It allows for automatic antivirus updates  
   C) It enables remote desktop functionality  
   D) It ensures reliable data exchange and replication  

7. Which of the following is an essential security measure in a high availability configuration?  
   A) Allowing unrestricted access between all network nodes  
   B) Disabling auditing to improve performance  
   C) Implementing strict access controls and using encryption for data in transit  
   D) Leaving firewall ports open for convenience  

8. Which statement best describes the role of the secondary replica in a high availability configuration?  
   A) It continuously synchronizes data with the primary and takes over operations during a failover event  
   B) It actively processes both read and write transactions during normal operations  
   C) It is solely used for storing backups and log files  
   D) It remains completely offline until manually activated  

9. Which Windows Server feature must be installed to support failover clustering?  
   A) Internet Information Services  
   B) Remote Desktop Services  
   C) Failover Clustering feature and management tools  
   D) Active Directory Domain Services  

10. Which PowerShell command is used to validate the prerequisites for a cluster before creation?  
    A) New-Cluster  
    B) Get-Cluster  
    C) Test-Cluster  
    D) Test-NetConnection  

11. Why might you configure static entries in the hosts file during a high availability lab exercise?  
    A) To bypass firewall restrictions  
    B) To enhance data encryption  
    C) To ensure reliable name resolution when DNS is unavailable  
    D) To accelerate internet browsing  

12. What is the purpose of having SQL Server Agent running in the high availability exercise?  
    A) To serve as the primary data backup mechanism  
    B) To manage SQL Server services  
    C) To schedule and automate maintenance and monitoring tasks  
    D) To facilitate real-time data replication  

13. What primary function does the "Always On Availability Groups" feature serve?  
    A) Improving query performance through caching  
    B) Compressing data to save storage space  
    C) Encrypting database backups automatically  
    D) Providing automated high availability and disaster recovery  

14. Why is certificate‑based endpoint authentication important in a high availability setup?  
    A) It encrypts data at rest on the server  
    B) It monitors SQL query performance  
    C) It authenticates users for the primary database  
    D) It secures the communication between primary and secondary replicas  

15. Which T‑SQL command is used to create a new database as part of the exercise?  
    A) ADD DATABASE  
    B) NEW DATABASE  
    C) CREATE DATABASE  
    D) INIT DATABASE  

16. Before performing backups in the lab, the database should be set to which recovery mode?  
    A) Minimal recovery mode  
    B) Simple recovery mode  
    C) Full recovery mode  
    D) Bulk-logged recovery mode  

17. When manually restoring a database for an Always On Availability Group, which seeding mode is most appropriate?  
    A) Full backup seeding  
    B) Automatic seeding  
    C) Join Only  
    D) Skip Initial Data Synchronization  

18. What is the purpose of testing failover in a high availability configuration?  
    A) To assess network bandwidth utilization  
    B) To validate the backup schedule  
    C) To measure the speed of the primary server  
    D) To verify that the failover mechanism is correctly configured and functional  

19. Which method is recommended for checking the synchronization status of availability replicas?  
    A) Rebooting the secondary server  
    B) Executing a T‑SQL query on sys.dm_hadr_availability_replica_states  
    C) Running a disk defragmentation check  
    D) Updating SQL Server to the latest version  

20. Why is continuous monitoring critical in a high availability environment?  
    A) It adds unnecessary complexity to the system  
    B) It forces constant configuration changes  
    C) It disables automatic updates for security reasons  
    D) It detects replication delays, resource bottlenecks, and failures in real time  

------------------------- -------------------------

#### Answers

1. **C** – Ensuring continuous operations and minimal downtime  
   *High availability aims to keep the system operational 24/7 and prevent data loss.*

2. **C** – The active server that handles all read/write operations  
   *The primary replica is responsible for processing transactions and ensuring service continuity.*

3. **B** – Synchronous replication waits for confirmation from the secondary before committing  
   *This process guarantees data consistency, albeit with a potential slight delay.*

4. **C** – Ensures near-perfect data consistency with minimal risk of data loss  
   *By confirming writes on the secondary, synchronous replication minimizes potential data loss.*

5. **C** – Slight latency because the primary waits for the secondary’s confirmation  
   *The added wait time during transaction commits can lead to minor delays in performance.*

6. **D** – It ensures reliable data exchange and replication  
   *Proper connectivity is fundamental for continuous communication between nodes.*

7. **C** – Implementing strict access controls and using encryption for data in transit  
   *Securing data channels is critical to protect sensitive information in transit.*

8. **A** – The secondary replica continuously synchronizes with the primary and is poised to assume operations during a failover, ensuring minimal downtime  
   *Maintaining constant synchronization allows the secondary replica to seamlessly take over when needed, minimizing service interruptions.*

9. **C** – Failover Clustering feature and management tools  
   *These tools are necessary to build and manage clusters that underpin high availability solutions.*

10. **C** – Test-Cluster  
    *This command verifies that the environment meets all prerequisites for a successful cluster setup.*

11. **C** – To ensure reliable name resolution when DNS is unavailable  
    *Static hosts file entries act as a fallback for name resolution in lab environments.*

12. **C** – To schedule and automate maintenance and monitoring tasks  
    *SQL Server Agent helps automate key tasks, reducing manual overhead in High Availability setups.*

13. **D** – Providing automated high availability and disaster recovery  
    *Always On Availability Groups enable seamless failover and data replication for continuous operations.*

14. **D** – It secures the communication between primary and secondary replicas  
    *Certificate‑based authentication creates a trusted, encrypted channel between servers.*

15. **C** – CREATE DATABASE  
    *The CREATE DATABASE command is standard for initializing a new SQL Server database.*

16. **C** – Full recovery mode  
    *Full recovery mode is essential for maintaining comprehensive transaction logs needed for backup and recovery.*

17. **C** – Join Only  
    *The Join Only option integrates a manually restored database into the availability group without re-seeding.*

18. **D** – To verify that the failover mechanism is correctly configured and functional  
    *Testing failover confirms that the high availability setup will operate as expected during an outage.*

19. **B** – Executing a T‑SQL query on sys.dm_hadr_availability_replica_states  
    *This query provides detailed insights into the synchronization and health of each replica.*

20. **D** – It detects replication delays, resource bottlenecks, and failures in real time  
    *Continuous monitoring is critical for proactively identifying and resolving issues in a High Availability environment.*

------------------------- -------------------------
------------------------- -------------------------
------------------------- -------------------------

## Azure

Azure SQL Database and Managed Instance come with **built‑in high availability features**.
Instead of manually configuring clusters, replication, and certificate‑based endpoints, Azure provides automated failover, built‑in replication, and a simplified failover group mechanism.
This means **less manual configuration** and a **lower operational overhead** while still meeting the key High Availability requirements.

Azure SQL already provides **built‑in redundancy within a region** by **automatically maintaining multiple copies of your data within that same region**, but failover is still relevant for specific scenarios:

* **Region‑Level Outages**: If an entire Azure region becomes unavailable (e.g., due to a major power or network disruption), local redundancy won’t protect you. A secondary server in a different region, configured through active geo‑replication or failover groups, ensures you can fail over to a completely different location.

* **Disaster Recovery**: Many organizations need cross‑region replication to meet compliance or RTO/RPO objectives. Setting up a failover group or active geo‑replication to another region lets you restore operations quickly if the primary region fails.

* **Read Scale‑Out**: Even if you’re not worried about regional failures, a failover group or geo‑replication can provide a readable secondary in another region. This **offloads read workloads from your primary server** and **reduces latency** for users in other geographical locations.

* **"Backup"**: High availability (via failover groups or active geo‑replication) provides an **actively synchronized secondary database in another region**. If the primary region fails, you can fail over quickly to the secondary with minimal downtime and data loss. This approach typically:

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
            > * **Standby Replica Creation**: Azure will automatically create a read-only replica of your primary database on the secondary server. This replica will be kept in sync with the primary using active geo‑replication.
            >
            > * **Role in Failover**: This standby replica is what takes over (or becomes the new primary) when you perform a failover. It ensures that your database remains available with minimal downtime and data loss.
            >
            > * **Read-Only Workloads**: In addition to being the failover target, the standby replica can serve read-only queries via a dedicated listener endpoint if you choose. This offloads read workloads from the primary.
            >
            > * **No Pre-existing Replica Needed**: If you already have a replica configured (for example, if you set up active geo‑replication manually), you might not need to use this option. But if not, “Create standby replica” automates that step for you.

            > **If you DO NOT select this option**:
            >
            > * **No automatic replication is initiated**: The failover group will be created without automatically creating or linking a secondary copy of your database.
            >
            > * **Existing Replication is Required**: If you already have an active geo‑replication relationship set up manually, the failover group might use that existing standby. Otherwise, your database remains only on the primary server until you manually configure replication.
            >
            > * **Potential Impact on Failover**: Without a standby replica, a failover event won’t have a ready secondary to switch to, meaning you’d have to set up replication later or risk downtime.

        In short, if you don’t check this option, you’re opting out of the automated creation of a standby replica—so ensure you either already have one in place or plan to add it manually for proper high availability.

##### Verify High Availability
   * After the failover group is created, use the Azure Portal to view the status and health of the failover group
   * Optionally, connect via SSMS to simulate a failover scenario by forcing a manual failover through the portal (if testing is supported)
   * Check that the secondary server takes over with minimal disruption

##### Implement Security Measures
   * Review and adjust firewall rules to limit access to only authorized IP addresses
   * Configure Virtual Network rules if necessary
   * Enable auditing and threat detection to monitor access and any suspicious activities

##### Set Up Monitoring
   * Open Azure Monitor and add metrics relevant to your SQL Database—such as failover events, DTU/vCore consumption, and replication latency
   * Configure alerts to notify you of potential issues that could affect high availability

------------------------- -------------------------
------------------------- -------------------------

### Quiz

1. What is the primary mechanism for ensuring high availability in Azure SQL Database?  
   A) On‑prem clustering techniques  
   B) Custom T‑SQL scripts for replication  
   C) Manual replication between servers  
   D) Built‑in replication and failover groups  

2. What benefit do failover groups provide in Azure SQL Database?  
   A) They automatically manage regional failover  
   B) They require manual intervention during outages  
   C) They are used solely for backup scheduling  
   D) They reduce the overall storage cost  

3. How does Azure SQL Database simplify high availability compared to on‑premises solutions?  
   A) By integrating replication, failover, and monitoring into the service  
   B) By requiring manual configuration of clustering  
   C) By eliminating the need for any backup processes  
   D) By using on‑premises DMVs for status checks  

4. Which aspect of the service tier (DTU/vCore) directly affects high availability performance?  
   A) The geographic region of the data center  
   B) The level of performance impacting replication speed and failover time  
   C) Database size only  
   D) The color of the user interface  

5. When configuring a failover group in the Azure Portal, what is the role of the secondary server?  
   A) It is used solely for running maintenance tasks  
   B) It provides additional storage for backups  
   C) It serves as a standby that automatically takes over in the event of an outage  
   D) It acts as a development server only  

6. Which tool is recommended for monitoring high availability in Azure SQL Database?  
   A) Windows Event Viewer  
   B) Azure Monitor and Query Performance Insight  
   C) SQL Server Agent  
   D) On‑premises DMVs  

7. Which security best practice is recommended for Azure SQL Database High Availability setups?  
   A) Using default credentials for simplicity  
   B) Disabling firewall rules to improve performance  
   C) Allowing unrestricted access for rapid deployment  
   D) Configuring firewall and VNet rules, and enabling auditing  

8. What does “automatic failover” mean in the context of Azure SQL Database?  
   A) It requires a scheduled restart of the primary server  
   B) The secondary server automatically takes over if the primary becomes unavailable  
   C) The secondary server only becomes active after manual approval  
   D) It is only enabled during off-peak hours  

9. How does service tier selection (DTU vs vCore) impact high availability?  
   A) It controls the geographic location of the secondary server  
   B) It only changes the database pricing  
   C) It has no effect on high availability  
   D) It determines performance levels that can affect replication and failover times  

10. Which step is essential when configuring high availability for Azure SQL Database through the portal?  
    A) Creating a failover group that links the primary and secondary servers  
    B) Writing complex PowerShell scripts  
    C) Installing on‑premises clustering software  
    D) Manually syncing databases via T‑SQL  

11. What does “built‑in high availability” imply in Azure SQL Database?  
    A) High Availability features must be configured manually  
    B) It is achieved through third‑party add‑ons  
    C) High availability is integrated and managed by Azure with minimal setup  
    D) It only applies to premium service tiers  

12. What is a primary advantage of using the Azure Portal for High Availability configuration?  
    A) It provides a guided and simplified configuration process  
    B) It mirrors on‑premises clustering exactly  
    C) It requires extensive manual scripting  
    D) It lacks monitoring capabilities  

13. In Azure SQL Database, who manages the replication and failover processes?  
    A) The database administrator manually  
    B) The Azure service automatically  
    C) On‑premises management tools  
    D) A third‑party vendor  

14. How is minimal downtime achieved in Azure SQL Database high availability?  
    A) Through daily scheduled maintenance windows  
    B) By disabling security features temporarily  
    C) Through automatic replication and the failover group mechanism  
    D) By requiring frequent manual intervention  

15. What is the significance of configuring firewall rules in an Azure High Availability setup?  
    A) They are optional and not recommended  
    B) They slow down the connection to the database  
    C) They ensure only authorized IP addresses can access the database  
    D) They are managed entirely by the operating system  

16. Which configuration step is simpler in Azure compared to on‑premises?  
    A) Manually running T‑SQL scripts for replication  
    B) Configuring certificate‑based endpoint authentication  
    C) Installing a Windows Server Failover Cluster  
    D) Setting up failover groups via the Azure Portal  

17. How does Azure secure its high availability infrastructure?  
    A) By leaving all ports open for maximum connectivity  
    B) By requiring customers to build their own security layers  
    C) By disabling automated backups  
    D) By integrating security features such as firewall settings, VNets, and auditing  

18. Which tool would you use to set alerts for High Availability issues in Azure SQL Database?  
    A) Azure Monitor  
    B) Windows PowerShell only  
    C) SQL Server Management Studio exclusively  
    D) DMVs from on‑prem installations  

19. What is the purpose of a failover group in Azure SQL Database?  
    A) To reduce database size  
    B) To enable read-only replicas for development  
    C) To automatically manage failover between the primary and secondary databases  
    D) To manually copy databases between servers  

20. Which statement best describes the overall high availability strategy in Azure SQL Database?  
    A) It exactly replicates on‑premises clustering methods  
    B) It is a streamlined, built‑in solution that automates replication, failover, and monitoring  
    C) It is a complex, manually configured solution  
    D) It only supports low‑availability configurations  

------------------------- -------------------------

#### Answers

1. **D** – Built‑in replication and failover groups  
   *Azure SQL Database leverages built‑in replication and managed failover groups to ensure high availability without manual clustering.*

2. **A** – They automatically manage regional failover  
   *Failover groups in Azure enable automatic regional failover, reducing downtime during outages.*

3. **A** – By integrating replication, failover, and monitoring into the service  
   *The service consolidates these functions, simplifying high availability compared to on‑premises solutions.*

4. **B** – The level of performance impacting replication speed and failover time  
   *Service tier performance directly influences the speed and reliability of replication and failover processes.*

5. **C** – It serves as a standby that automatically takes over in the event of an outage  
   *The secondary server is maintained as a ready backup to assume the primary role when needed.*

6. **B** – Azure Monitor and Query Performance Insight  
   *These tools provide comprehensive monitoring and alerting for high availability metrics.*

7. **D** – Configuring firewall and VNet rules, and enabling auditing  
   *Proper security measures help ensure that only authorized access occurs in a high availability setup.*

8. **B** – The secondary server automatically takes over if the primary becomes unavailable  
   *Automatic failover minimizes downtime by transferring operations seamlessly to the secondary server.*

9. **D** – It determines performance levels that can affect replication and failover times  
   *Choosing the right service tier is crucial as it directly impacts replication speed and the overall failover process.*

10. **A** – Creating a failover group that links the primary and secondary servers  
    *This step is fundamental to configuring automated high availability in Azure SQL Database.*

11. **C** – High availability is integrated and managed by Azure with minimal setup  
    *Azure’s built‑in High Availability features eliminate much of the manual configuration required on‑premises.*

12. **A** – It provides a guided and simplified configuration process  
    *The Azure Portal streamlines High Availability setup, making it accessible and user-friendly.*

13. **B** – The Azure service automatically  
    *Azure manages the replication and failover processes, reducing the administrative burden.*

14. **C** – Through automatic replication and the failover group mechanism  
    *This integration minimizes downtime by ensuring that replication and failover occur without manual intervention.*

15. **C** – They ensure only authorized IP addresses can access the database  
    *Proper firewall configuration is critical for protecting the database from unauthorized access.*

16. **D** – Setting up failover groups via the Azure Portal  
    *This approach is significantly simpler in Azure compared to traditional on‑premises clustering methods.*

17. **D** – By integrating security features such as firewall settings, VNets, and auditing  
    *Azure secures its High Availability infrastructure through a comprehensive suite of built‑in security features.*

18. **A** – Azure Monitor  
    *Azure Monitor is the recommended tool for setting alerts and tracking high availability issues.*

19. **C** – To automatically manage failover between the primary and secondary databases  
    *Failover groups ensure that failover happens seamlessly and automatically when needed.*

1. A – It is a streamlined, built‑in solution that automates replication, failover, and monitoring  
    *This answer best describes Azure’s overall strategy, which simplifies high availability through automation.*
