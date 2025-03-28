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

* Windows Server named `SERVER_PRIMARY` with SQL Server, SQL Server Management Studio, and PowerShell 7.x.
* Windows Server named `SERVER_SECONDARY` with SQL Server, SQL Server Management Studio, and PowerShell 7.x.



-------------------------

##### Windows Server

###### Failover Clustering

Perform these steps on both SERVER_PRIMARY and SERVER_SECONDARY:

- Run the following PowerShell script to install the Failover Clustering feature and management tools:

  ```powershell
  Install-WindowsFeature Failover-Clustering -IncludeManagementTools
  ```

  Expected Result:

  ```text
  Success Restart Needed Exit Code      Feature Result
  ------- -------------- ---------      --------------
  True    No             Success        {Failover Clustering, Remote Server Admini...
  ```

  

- Run the following PowerShell script to verify the installation on each server:

  ```powershell
  Get-WindowsFeature Failover-Clustering
  ```

  Expected Result:

  ```text
  Display Name                                            Name                       Install State
  ------------                                            ----                       -------------
  [X] Failover Clustering                                 Failover-Clustering            Installed
  ```



-------------------------

###### Cluster

Perform these steps on both SERVER_PRIMARY and SERVER_SECONDARY:

- Run the following PowerShell script to run cluster validation tests before creating the cluster on both servers:

  ```powershell
  Test-Cluster -Node localhost -Verbose
  ```

- On `SERVER_PRIMARY`, run the following PowerShell script to create a new cluster:

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

* On `SERVER_PRIMARY`, run the following PowerShell script to get the IP address for use in the next step:

  ```powershell
  ipconfig
  ```



-------------------------

###### Cluster Node

- On `SERVER_SECONDARY`, run the following PowerShell script to test connectivity between servers:

  ```powershell
  Test-Connection -Count 2 -ComputerName "<SERVER_PRIMARY_IPADDRESS>"
  ```

  Expected result:

  ```text
     Destination: 172.30.209.74
  
  Ping Source           Address                   Latency BufferSize Status
                                                     (ms)        (B)
  ---- ------           -------                   ------- ---------- ------
     1 WIN-JH6VAICNUJH  172.30.209.74                   1         32 Success
     2 WIN-JH6VAICNUJH  172.30.209.74                   1         32 Success
  ```

- On `SERVER_SECONDARY`, run the following PowerShell script to test connectivity between servers:

  ```powershell
  Get-Service -Name ClusSvc
  ```

  Expected result:

  ```text
  Status   Name               DisplayName
  ------   ----               -----------
  Stopped  ClusSvc            Cluster Service
  ```

  > Note: If Status = "Stopped", then blah

  

- On `SERVER_SECONDARY`, run the following PowerShell script to add a node to the cluster:

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

###### Prepare Databases

* Create "c:\Temp" directories on both Windows Servers

* On `SERVER_PRIMARY`, open SQL Server Management Studio and connect to the database engine

  - Execute the following T‑SQL to create a database:

    ```sql
    CREATE DATABASE myDatabase ON (NAME = myDatabase_Data, FILENAME = 'C:\Temp\myDatabase.mdf')
    ```
  
  
    - Execute the following T‑SQL to verify the database was created and is online:
  
      ```sql
      SELECT name FROM sys.databases WHERE name = 'myDatabase'
      ```
  
  
    - Execute the following T‑SQL to ensure the database is in full recovery mode
  
      ```sql
      ALTER DATABASE myDatabase SET RECOVERY FULL
      ```
  
  


-------------------------

###### Configure the Always On Availability Group

> Note: Server requirements: 1) SQL Authentication `sa` enabled, 2) Windows Firewall + Inbound Rule `SQL Server TCP 1433`

- On `SERVER_PRIMARY`, open SQL Server Management Studio and connect to the database engine

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
      * "Replicas" tab >> "Availability Replicas" matrix: No changes required
        * Automatic Failover: `Disabled`
  


* Enabled
  
          * Enables the cluster to automatically switch the primary role to a secondary replica without manual intervention when a failure is detected, which minimizes downtime
          * Requires synchronous commit mode to ensure data consistency, and it may add slight latency as the system waits for confirmation from the secondary
    
        * Disabled (default)
        
          * Requires manual intervention to initiate the failover, offering more control over the process and potentially avoiding unintended failovers during transient issues
          * May lead to increased downtime if the primary fails and manual actions are delayed, but can be beneficial in environments where failover decisions need careful review before execution
        
      * Availability Mode: `Asynchronous-commit mode`
      
        * Synchronous-commit mode
        
          * Ensures that transactions on the primary server wait for confirmation from the secondary replica before committing, which minimizes the risk of data loss during a failover
          * However, it may introduce additional latency as the primary waits for network round-trips, which can impact performance in high-latency environments
        
        * Asynchronous-commit mode (default)
        
          * Allows the primary server to commit transactions immediately without waiting for the secondary to confirm, which improves performance by reducing latency
          * This mode comes with a risk of data loss if the primary fails before the secondary has received and applied the latest changes
      
      * Readable Secondary: `No`
      
        * No (default)
        
          * Keeps all read/write operations on the primary, ensuring that all data is accessed from the most current, committed source
          * May be preferred in scenarios where strict consistency is required or when the workload does not justify offloading read operations to a secondary replica
        * Yes
        
          * Allows the secondary replica to accept read-only workloads, offloading query processing from the primary and potentially improving overall system scalability and performance
          * Useful for reporting and analytics, as it can help balance workload and reduce the load on the primary server
        
        * Read-intent only
        
          * Only client connections that explicitly specify an intent to read (using `ApplicationIntent=ReadOnly` in the connection string) are allowed on the secondary replica
          * Helps prevent unintentional read connections on the secondary while still leveraging its resources for appropriate workloads

  * "Select Data Synchronization": `Automatic Seeding`


    * Automatic Seeding
      - Automatically creates the database on each secondary replica, then streams the data directly from the primary replica
      - Simplifies the setup process because you do not have to manually perform backups and restores on the secondaries
      - May be limited by your network bandwidth or other environmental constraints, especially for very large databases
    * Full Database and Log Backup
      - Wizard takes a full backup and log backup of each database on the primary and restores them to the secondaries
      - Requires a valid file path (e.g., a shared folder) that all replicas can access
      - Ensures that each secondary database is properly prepared in NORECOVERY mode, after which the wizard finalizes the join to the availability group
    * Join Only
      - Assumes you have already manually backed up and restored the databases to the secondary replicas in NORECOVERY mode
      - The wizard merely joins those restored databases to the availability group, rather than creating or restoring them automatically
      - Useful when you have complex restore requirements, large databases, or prefer to handle the backup/restore process yourself
    * Skip Initial Data Synchronization
      - Creates the availability group without performing any data movement or backup/restore steps
      - You must handle all backup/restore operations and manually join each secondary database to the availability group later
      - Often chosen for highly customized workflows or environments where you need to control every step of the data synchronization process manually

  * "Validation": Confirm "Success" results

  * "Summary": Review and then click "Finish"

  * "Results": Monitor progress and confirm success

  

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
