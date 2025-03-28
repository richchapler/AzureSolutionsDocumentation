# SQL: High Availability

## Basics

### Use Case

The database administration team at a mid-sized organization must ensure data availability. Key Requirements:

- Continuous Operations: Support 24/7 database availability
- Data Integrity: Prevent data loss caused by disruptions
- Security: Data must be secure at all times
- Proactive Monitoring: Provide for quick issue detection

------------------------- ------------------------- -------------------------

### On‑Prem

#### Discussion

##### Fundamentals

- Concept
  - Ensure a database is always available to users
  - Set up multiple copies on different servers, possibly in separate geographic locations
  - Allow a secondary server to take over quickly if the primary fails
  - Provide a seamless user experience with minimal disruption
- Components
  - Primary Replica: The active database handling all read/write operations
  - Secondary Replica(s): Up-to-date copies that can serve as failover targets
- Benefits
  - Minimal downtime during failures
  - Reduced risk of data loss through continuous data synchronization
- Real-World Analogy
  - Similar to keeping a backup copy of a file on another computer, but with automatic updates and immediate availability

##### Setup and Configuration

- Preparation
  - Set up at least two servers that can communicate with each other
  - Install and configure the necessary SQL Server features for high availability
- Step-by-Step Process
  - Designate a Primary: Choose one server to handle all primary transactions
  - Configure Secondaries: Set up additional servers to receive data updates from the primary
  - Establish Connectivity: Ensure network and security settings allow the servers to exchange data
- Outcome
  - A working environment where the primary server’s data is continuously replicated to secondary servers
- Why It Matters
  - Establishes the foundation for continuous operations and lays the groundwork for advanced features later

##### Synchronous vs. Asynchronous

- Synchronous Replication
  - Definition: Data is copied to the secondary server at the same time it is written to the primary
  - Advantages:
    - Near-perfect data consistency
    - Minimal risk of data loss in the event of a failure
  - Tradeoffs:
    - May introduce slight latency, as the primary waits for confirmation from the secondary
- Asynchronous Replication
  - Definition: Data is sent to the secondary server with a delay
  - Advantages
    - Lower latency on the primary server since it doesn't wait for secondary confirmation
  - Tradeoffs
    - A small risk of data loss if a failure occurs before the secondary catches up
- Choosing the Right Method
  - Depends on the organization’s need for data consistency versus system performance

##### Security Considerations

- Data Protection
  - Encryption: Use encryption to secure data as it travels between the primary and secondary servers
  - Access Controls: Configure strict permissions so that only authorized users can access and manage the databases
- Compliance and Auditing
  - Ensure that the configuration meets any regulatory or organizational policies
  - Set up auditing to monitor any access or changes to the configuration
- Best Practices
  - Regularly update security patches and review access permissions
  - Use secure communication protocols to safeguard data integrity

##### Monitoring Considerations

- Importance of Monitoring: Early detection of issues such as replication delays or server failures is crucial.

- Tools and Techniques:
  - Dynamic Management Views (DMVs): Use DMVs to query system health and performance statistics
  - Performance Dashboards: Visual tools that display real-time metrics on replication status, CPU usage, and memory consumption
  
- What to Monitor
  - Replication latency between primary and secondary replicas
  - Server resource usage to detect potential bottlenecks
  - Error logs for any signs of communication failures or synchronization issues
  
- Outcome: Continuous assurance that the high availability setup is functioning optimally and that issues are addressed promptly

------------------------- -------------------------

#### Exercise

##### Pre-Requisites

* Two servers (`SERVER_PRIMARY` and `SERVER_SECONDARY`), each with:

  * Windows Server
    * PowerShell 7.x
    * Firewall + Inbound Rules:
      * SQL Server TCP 1433
      * SQL Server Availability Group Endpoint 5022

    * (Optional) Static entries in `C:\Windows\System32\drivers\etc\hosts` if DNS name resolution is unavailable

  * SQL Server
    * Configured for mixed authentication (since the demo environment won't be domain-joined)
    * Master Key (`CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<strongPassword>'`)
    * Certificate (`CREATE CERTIFICATE Hadr_cert WITH SUBJECT = 'AG endpoint certificate'`)
    * Backup Certificate (`BACKUP CERTIFICATE Hadr_cert TO FILE = 'C:\Temp\Hadr_cert.cer' WITH PRIVATE KEY ( FILE = 'C:\Temp\Hadr_cert.pvk', ENCRYPTION BY PASSWORD = '<strongPassword>')`)
    * SQL Server Agent started (`Start-Service -Name SQLSERVERAGENT`)
  * SQL Server Management Studio




-------------------------

##### Windows Server

###### Failover Clustering

Perform the following steps on both servers:

- Install Failover Clustering feature and management tools

  ```powershell
  Install-WindowsFeature Failover-Clustering -IncludeManagementTools
  ```

  Expected Result

  ```plaintext
  Success Restart Needed Exit Code      Feature Result
  ------- -------------- ---------      --------------
  True    No             Success        {Failover Clustering, Remote Server Admini...
  ```

- Verify installation

  ```powershell
  Get-WindowsFeature Failover-Clustering
  ```

  Expected Result

  ```plaintext
  Display Name                                            Name                       Install State
  ------------                                            ----                       -------------
  [X] Failover Clustering                                 Failover-Clustering            Installed
  ```



-------------------------

###### Cluster and Node

Perform the following steps on both servers:

- Validate prerequisites before creating the cluster on both servers

  ```powershell
  Test-Cluster -Node localhost -Verbose
  ```

- Create a new cluster on `SERVER_PRIMARY`

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

* Open Failover Cluster Manager on `SERVER_PRIMARY`, connect to your cluster, and confirm cluster status = "Online"

- Add a node to the cluster:

    ```powershell
    Add-ClusterNode -Name "myCluster"
    ```

- Verify that SERVER_SECONDARY has joined the cluster by running (on either server):

  ```powershell
  Get-ClusterNode
  ```



------------------------- -------------------------

##### SQL Server

###### Always On

- On `SERVER_PRIMARY`, open SQL Server Configuration Manager and then select "SQL Server Services"
  - Right-click on `SQL Server (MSSQLSERVER)` and select "Properties" from the resulting menu

  - Click the "Always On Availability Groups" tab and then check "Enable Always On Availability Groups"

  - Click "OK" and then restart `SQL Server (MSSQLSERVER)`


* Repeat the process for `SERVER_SECONDARY`



-------------------------

###### Certificate-Based Endpoint Authentication

* Create a master key on `SERVER_PRIMARY` and `SERVER_SECONDARY`

    ```sql
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'P@55word!'
    ```

* Create a certificate for the endpoint on `SERVER_PRIMARY` and `SERVER_SECONDARY`

    ```sql
    CREATE CERTIFICATE Hadr_cert WITH SUBJECT = 'HADR Endpoint Certificate'
    ```

* Backup the certificate on `SERVER_PRIMARY`

    ```sql
    BACKUP CERTIFICATE Hadr_cert TO FILE = 'C:\Temp\Hadr_cert.cer'
    WITH PRIVATE KEY (
        FILE = 'C:\Temp\Hadr_cert_privatekey.pvk',
        ENCRYPTION BY PASSWORD = 'P@55word!'
    )
    ```

* Copy `Hadr_cert.cer`, `Hadr_cert.pvk`, and `Hadr_cert_privatekey.pvk` to c:\Temp on `SERVER_SECONDARY`

* Restore the certificate on `SERVER_SECONDARY`

    ```sql
    CREATE CERTIFICATE Hadr_cert FROM FILE = 'C:\Temp\Hadr_cert.cer'
    WITH PRIVATE KEY (
        FILE = 'C:\Temp\Hadr_cert_privatekey.pvk',
        DECRYPTION BY PASSWORD = 'P@55word!'
    )
    ```

* Recreate the HADR endpoint on `SERVER_PRIMARY` and `SERVER_SECONDARY`

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




-------------------------

###### Prepare Databases

* Create "c:\Temp" directories on both Windows Servers

* On `SERVER_PRIMARY`, open SQL Server Management Studio and connect to the database engine

  - Create a database:

    ```sql
    CREATE DATABASE myDatabase ON (NAME = myDatabase_Data, FILENAME = 'C:\Temp\myDatabase.mdf')
    ```


    - Verify the database was created and is online:
    
      ```sql
      SELECT name FROM sys.databases WHERE name = 'myDatabase'
      ```


    - Ensure the database is in full recovery mode
    
      ```sql
      ALTER DATABASE myDatabase SET RECOVERY FULL
      ```

  * Backup the database

      ```sql
      BACKUP DATABASE myDatabase TO DISK = 'C:\Temp\myDatabase.bak' WITH INIT
      BACKUP LOG myDatabase TO DISK = 'C:\Temp\myDatabase_log.bak' WITH INIT;
      ```

* On `SERVER_SECONDARY`

  * Copy `myDatabase.bak` and `myDatabase_log.bak` to c:\Temp

  * Restore the backup

    ```sql
    RESTORE DATABASE myDatabase FROM DISK = 'C:\Temp\myDatabase.bak' WITH NORECOVERY,
        MOVE 'myDatabase_Data' TO 'C:\Temp\myDatabase.mdf',
        MOVE 'myDatabase_Log' TO 'C:\Temp\myDatabase.ldf';
    
    RESTORE LOG myDatabase FROM DISK = 'C:\Temp\myDatabase_log.bak' WITH NORECOVERY;
    ```



-------------------------

###### Always On Availability Group

On `SERVER_PRIMARY`, open SQL Server Management Studio and connect to the database engine:

- "Object Explorer": Right-click "Always On High Availability" and then "New Availability Group Wizard" in the resulting menu

- Complete the "New Availability Group" wizard:

  - "Introduction": Skip

  - "Specify Options"

    - Enter "Availability group name" `myAvailabilityGroup` 

    - Choose "Cluster type": `Windows Server Failover Cluster`

      - Default value... for health checks and failovers  
    
      - Other options:
        - EXTERNAL: For third-party cluster managers (e.g., Pacemaker on Linux)... useful if WSFC isn’t feasible  
    
        - NONE: Creates a read-scale, cluster-less Availability Group... no automatic failover, suitable for offloading reads to secondaries
    
    - "Database Level Health Detection": `unchecked`  
    
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


        * Automatic Seeding: Streams the database directly from primary to secondary. Easiest option but may silently fail in lab setups without DNS, proper permissions, or shared services.
        * Full Database and Log Backup: Wizard handles backup and restore using a shared location. Reliable, but requires a common file path accessible by all replicas.

         * Join Only: Use when you've already manually restored the database with `NORECOVERY`. The wizard just joins it to the availability group — ideal for custom or large database setups.
            
         * Skip Initial Data Synchronization: Creates the AG without any data movement. You’re fully responsible for preparing and joining each secondary manually.

     * "Validation": Confirm "Success" results
       
     * "Summary": Review and then click "Finish"
       
     * "Results": Monitor progress and confirm success




- After creating the availability group and adding replicas, recreate the endpoints on both `SERVER_PRIMARY` and `SERVER_SECONDARY`. This step ensures consistent endpoint configuration across both replicas, especially if endpoints were created implicitly or are misaligned.

  ```sql
  DROP ENDPOINT Hadr_endpoint;
  GO
  CREATE ENDPOINT Hadr_endpoint  
      STATE = STARTED  
      AS TCP (LISTENER_PORT = 5022)  
      FOR DATA_MIRRORING (ROLE = ALL, AUTHENTICATION = WINDOWS NEGOTIATE, ENCRYPTION = REQUIRED ALGORITHM AES);
  ```

- After adding the secondary replica, ensure its seeding mode is set to `AUTOMATIC` (required for automatic seeding):

  ```sql
  ALTER AVAILABILITY GROUP myAvailabilityGroup 
  MODIFY REPLICA ON 'SERVER_SECONDARY' 
  WITH (SEEDING_MODE = AUTOMATIC);
  ```



-------------------------

###### Verify Configuration  

- On `SERVER_PRIMARY`, execute the following T‑SQL to verify configuration and status:

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

  - `0` = NOT SYNCHRONIZING: Replica isn't synchronizing data.
  - `1` = SYNCHRONIZING: Replica is in the process of synchronizing.
  - `2` = SYNCHRONIZED: Replica is synchronized (healthy, fully synchronized with the primary).
  - `3` = REVERTING: Replica reverting state (rarely encountered).
  - `4` = INITIALIZING: Replica is initializing data synchronization.

- On `SERVER_SECONDARY`, execute the following T‑SQL to LOREM IPSUM



-------------------------

##### Test Failover

###### Initiate Failover

- On the secondary replica, execute the following T-SQL command to initiate a manual failover:

  ```sql
  ALTER AVAILABILITY GROUP [myAvailabilityGroup] FAILOVER;
  ```

###### Confirm Failover

- On the former secondary replica, confirm it is now acting as the primary replica by executing the verification query provided previously:

  ```sql
  SELECT
      ag.name AS AGName,
      ar.replica_server_name AS ReplicaName,
      rs.role_desc AS ReplicaRole,
      rs.operational_state_desc AS OperationalState,
      rs.connected_state_desc AS ConnectedState,
      rs.synchronization_state_desc AS SyncState
  FROM sys.availability_groups AS ag
  JOIN sys.availability_replicas AS ar
      ON ag.group_id = ar.group_id
  JOIN sys.dm_hadr_availability_replica_states AS rs
      ON ar.replica_id = rs.replica_id;
  ```

- If an availability group listener is configured, verify client connections continue uninterrupted



-------------------------

##### Security

- Encryption and Access Controls:
  - Ensure communication endpoints use SSL/TLS for encryption
  - Verify that access is restricted to authorized administrators only

- Policy Review:
  - Check that the configuration meets organizational or regulatory security standards
  
    

-------------------------

##### Monitoring

- Using DMVs and Dashboards:

  - Query system health and replication status using DMVs such as:

    ```sql
    SELECT * FROM sys.dm_hadr_database_replica_states;
    ```

  - Use SSMS performance dashboards to view real-time metrics on CPU usage, memory, and replication latency


- Log Monitoring:

  - Review SQL Server error logs and the Windows Event Viewer for any issues related to communication or synchronization



------------------------- -------------------------

#### Quiz

Lorem Ipsum

##### Answers

Lorem Ipsum

------------------------- ------------------------- -------------------------

### Azure

- Built‑in High Availability Features in Azure SQL Database:
  - Explore how failover groups and built‑in replication ensure seamless connectivity.
- Impact of Service Tiers (DTU/vCore) on HA:
  - Understand how service tier selection affects performance and availability.
- Initial Configuration Using the Azure Portal:
  - Guided steps for configuring HA settings through the portal.
- Security Considerations:
  - Implement best practices such as firewall configurations, network isolation, and auditing within Azure HA setups.
- Monitoring Considerations:
  - Leverage Azure Monitor and Query Performance Insight to track HA performance and health.
- Quiz:
  - Short assessments to verify your knowledge of service tiers and initial HA configuration.
  - Answers: Detailed answer keys for each quiz question.

------------------------- ------------------------- ------------------------- -------------------------

## Advanced

### Use Case

The database administration team now needs to improve performance and reliability during high-demand periods and complex failover scenarios.

Key Requirements:

- Optimized Failover: Minimize downtime during peak loads
- Enhanced Performance: Continuously monitor and adjust replication
- Scalability: Support dynamic changes to meet varying workloads
- Advanced Security: Strengthen access controls and encryption
- Robust Disaster Recovery: Enable geo‑replication and automated backups for rapid recovery

### On‑Prem

- Advanced Always On Settings:
  - Configure advanced options like read‑only routing and backup offloading.
- Fine‑Tuning Failover Policies and Replica Synchronization:
  - Optimize failover thresholds and ensure robust synchronization.
- Integration with Windows Server Failover Clustering (WSFC):
  - Understand the role of WSFC in supporting high availability.
- Security Considerations:
  - Enhance security by applying encryption, auditing, and secure configuration practices during advanced setups.
- Monitoring Considerations:
  - Employ performance monitoring tools and DMVs to track synchronization and failover metrics.
- Exercise:
  - A lab to integrate WSFC and adjust advanced settings.
- Quiz:
  - Assessments focused on troubleshooting and optimizing failover configurations.
  - Answers: Detailed answer keys for each quiz question.

### Azure

- Advanced Service Tier Configurations and Scaling Options:
  - Learn how to adjust and scale HA settings to reduce downtime.
- Customizing HA Behavior with Performance Monitoring Tools:
  - Use Azure’s native tools to fine‑tune and validate high availability configurations.
- Managing Geo‑Replication and Automated Backups:
  - Configure and manage geo‑replication to support disaster recovery.
- Security Considerations:
  - Implement advanced security measures including role‑based access control, encryption, and continuous auditing.
- Monitoring Considerations:
  - Utilize Azure Monitor, Log Analytics, and other tools to continuously assess performance and HA readiness.
- Quiz:
  - Assessments to test your knowledge of scaling options and advanced configuration tasks.
  - Answers: Detailed answer keys for each quiz question.

## Hard Topic 1: Designing Quorum for Failover

### Use Case

The database administration team now faces complex clustering challenges. They need a robust failover design that clearly defines how decisions are made when some nodes fail.

Key Requirements:

- Clear Quorum Models: Understand and apply the best quorum model for their environment.
- Reliable Decision-Making: Ensure that the system continues operating even when some nodes are unavailable.
- Troubleshooting: Quickly identify and resolve quorum-related issues.
- Secure Configurations: Protect the quorum setup from unauthorized changes.
- Continuous Monitoring: Track quorum health to maintain system stability.

### On‑Prem

- Understanding Quorum Models:
  - Study models such as Node Majority and Node and File Share Majority.
- Best Practices for Configuring and Testing Quorum:
  - Develop robust quorum configurations.
- Troubleshooting Common Quorum Issues:
  - Techniques for diagnosing and resolving quorum-related problems.
- Security Considerations:
  - Integrate secure access and controls to prevent unauthorized modifications in quorum configurations.
- Monitoring Considerations:
  - Apply monitoring tools to verify quorum status and identify issues promptly.
- Exercise:
  - A lab simulating quorum configuration and testing scenarios.
- Quiz:
  - Short tests to reinforce your knowledge of quorum models and best practices.
  - Answers: Detailed answer keys for each quiz question.

### Azure

- Managing HA in a PaaS Environment:
  - Understand how high availability is achieved without manual quorum settings.
- Impact of Service Tiers on Failover and Resiliency:
  - Explore how different service tiers influence failover behavior.
- Disaster Recovery Considerations:
  - Plan for disaster scenarios using built‑in Azure features.
- Security Considerations:
  - Ensure service tier configurations include security measures to protect against unauthorized access.
- Monitoring Considerations:
  - Use Azure monitoring tools to assess resiliency and performance under varying service tiers.
- Quiz:
  - Assess your understanding of HA management in a PaaS environment.
  - Answers: Detailed answer keys for each quiz question.

## Hard Topic 2: Troubleshooting and Performance Tuning in HA Scenarios

### Use Case

The database administration team is facing intermittent performance issues and occasional delays during failovers. They need to quickly diagnose problems, tune performance, and maintain stable operations under varying workloads.

Key Requirements:

- Effective Troubleshooting: Quickly identify and resolve replication or failover issues.
- Performance Tuning: Optimize replication and synchronization for better efficiency.
- Real-Time Monitoring: Use advanced tools to track performance metrics.
- Secure Adjustments: Ensure that troubleshooting and tuning do not compromise security.
- Resilience: Maintain stable operations even under heavy workloads.

### On‑Prem

- Diagnosing and Resolving Synchronization and Failover Issues:
  - Identify common issues and apply effective fixes, including post‑failover steps.
- Tools and Techniques for Performance Tuning:
  - Leverage diagnostic tools and dynamic management views.
- Real‑World Best Practices:
  - Analyze scenarios where troubleshooting improved operational performance.
- Security Considerations:
  - Maintain security integrity during troubleshooting and tuning.
- Monitoring Considerations:
  - Continuously monitor performance metrics to ensure issues are promptly detected.
- Exercise:
  - A lab simulating failover and performance tuning scenarios.
- Quiz:
  - Assessments focused on troubleshooting strategies and performance optimization.
  - Answers: Detailed answer keys for each quiz question.

### Azure

- Utilizing Azure Monitoring Tools for HA Troubleshooting:
  - Use tools like Azure Metrics and Query Performance Insight to diagnose issues.
- Tuning Performance by Adjusting Service Tiers and Scaling Options:
  - Learn to balance performance and cost in a managed environment.
- Maintaining Resilience Under Load:
  - Strategies to ensure continued availability during high-demand periods.
- Security Considerations:
  - Verify that performance adjustments maintain or enhance security measures.
- Monitoring Considerations:
  - Use Azure native monitoring to continuously assess HA readiness.
- Quiz:
  - Short assessments to test your understanding of troubleshooting and tuning techniques.
  - Answers: Detailed answer keys for each quiz question.

## Hard Topic 3: Disaster Recovery Planning and Operational Best Practices

### Use Case

The database administration team must be ready for unexpected catastrophic events. They need a disaster recovery plan that minimizes downtime and restores data quickly and securely.

Key Requirements:

- Rapid Recovery: Enable quick restoration of services after an outage.
- Comprehensive Backup Strategies: Implement automated backups and geo‑replication.
- Defined RTO and RPO: Establish clear recovery time and data loss objectives.
- Ongoing Testing: Regularly test failover procedures and maintain DR readiness.
- Secure DR Processes: Ensure disaster recovery measures protect data security.

### On‑Prem

- Integrating Backup, Restore, and Geo‑Replication Strategies:
  - Develop comprehensive disaster recovery (DR) plans that minimize downtime.
- Defining RTO and RPO:
  - Set realistic Recovery Time Objectives (RTO) and Recovery Point Objectives (RPO).
- Regular Failover Testing, Patching, and Maintenance Routines:
  - Implement best practices for continuous operational readiness.
- Security Considerations:
  - Ensure that DR processes are secure and compliant.
- Monitoring Considerations:
  - Use monitoring tools to validate backup integrity and DR plan effectiveness.
- Exercise:
  - A lab to simulate backup and restore operations and perform failover testing.
- Quiz:
  - Assessments on DR planning and execution.
  - Answers: Detailed answer keys for each quiz question.

### Azure

- Leveraging Automated Backups and Geo‑Replication:
  - Utilize Azure’s native features for rapid disaster recovery.
- Operational Strategies for Minimal Downtime:
  - Optimize DR plans in a managed environment.
- Proactive Monitoring and Maintenance:
  - Continuously monitor DR readiness using Azure tools.
- Security Considerations:
  - Secure DR processes with encryption and automated auditing.
- Monitoring Considerations:
  - Regularly review DR performance and readiness metrics.
- Quiz:
  - Assessments to verify your understanding of operational best practices for DR.
  - Answers: Detailed answer keys for each quiz question.

## Hard Topic 4: Security, Compliance, and Comparative Analysis of HA Technologies

### Use Case

The database administration team is evaluating various high availability solutions to find one that meets strict security and compliance standards without sacrificing performance. They need to compare different HA approaches to determine the best fit for their secure and efficient operations.

Key Requirements:

- Secure Communication: Ensure all data is encrypted and access is tightly controlled.
- Compliance: Meet regulatory standards with robust auditing and reporting features.
- Comparative Analysis: Evaluate different HA technologies (e.g., Always On, clustering, log shipping, mirroring) to select the optimal solution.
- Ongoing Security Monitoring: Continuously monitor for potential threats without impacting performance.
- Operational Efficiency: Choose a solution that balances high security with ease of management and reliable performance.

### On‑Prem

- Securing Communication Between Replicas and Ensuring Compliance:
  - Implement encryption, auditing, and access controls within HA configurations.
- Impact on Data Encryption, Auditing, and Regulatory Requirements:
  - Understand the broader security implications of HA setups.
- Review of Alternative HA Technologies:
  - Compare Always On with clustering, log shipping, and mirroring.
- Security Considerations:
  - Continually assess and update security policies as part of HA operations.
- Monitoring Considerations:
  - Use monitoring to ensure that alternative HA solutions meet performance and security requirements.
- Exercise:
  - A lab focused on configuring secure HA environments and comparing alternative technologies.
- Quiz:
  - Assessments to test your knowledge of security controls and compliance impacts in various HA technologies.
  - Answers: Detailed answer keys for each quiz question.

### Azure

- Ensuring Secure High Availability in a PaaS Environment:
  - Leverage Azure’s built‑in security features to maintain robust HA.
- Built‑In Compliance Features:
  - Understand how Azure HA solutions support regulatory and compliance requirements.
- Comparative Insights on HA Approaches:
  - Analyze differences between on‑prem and cloud HA solutions in terms of security and operational efficiency.
- Security Considerations:
  - Integrate continuous security monitoring and compliance reviews into HA practices.
- Monitoring Considerations:
  - Utilize Azure’s monitoring tools to ensure that security and compliance metrics are met.
- Quiz:
  - Assessments focused on security best practices and compliance in Azure HA environments.
  - Answers: Detailed answer keys for each quiz question.
