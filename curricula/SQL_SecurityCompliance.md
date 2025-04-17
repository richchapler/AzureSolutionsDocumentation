# SQL: Security and Compliance

## Use Case

The database administration team at a mid‑sized organization must ensure that critical data is protected from unauthorized access, breaches, and non‑compliance with regulatory standards. Key requirements include:

- **Data Protection**: Shield sensitive information from internal and external threats  
- **Network Isolation**: Restrict connectivity so that only approved users and systems access the databases  
- **Auditing and Monitoring**: Track security events, classify sensitive data, and monitor user activity to detect anomalies  
- **Operational Continuity**: Ensure systems remain secure and available during maintenance and emergencies  
- **Regulatory Compliance**: Meet legal and regulatory requirements; examples:  
  - **General Data Protection Regulation (GDPR)**: European Union regulation protecting personal data and privacy  
  - **Health Insurance Portability and Accountability Act (HIPAA)**: United States law safeguarding patient health information  
  - **Payment Card Industry Data Security Standard (PCI‑DSS)**: Standards to secure credit and debit card data  
  - **Sarbanes‑Oxley Act (SOX)**: United States law enforcing strict financial reporting and internal controls to prevent fraud

<!-- ------------------------- ------------------------- -->

## Fundamentals

### Authentication
...confirms an identity

- **Windows**: Leverage domain credentials for secure, integrated access  
- ****SQL Server****: Authenticate using **SQL Server**–specific credentials (username and password), suitable when Windows Authentication is not used  
- **Multi‑Factor**: Enforce a second verification step via Windows or Active Directory using smart cards or Entra ID multi‑factor authentication; note that multi‑factor authentication is not supported for **SQL Server** Authentication  
- **Entra ID Authentication**: centralize identity management using Entra ID for database access  
  - **SQL Server**: requires federation with Entra ID via Active Directory Federation Services (no native support)  
  - **Azure SQL**: configure an Entra ID administrator, create contained database users with FROM EXTERNAL PROVIDER, and enforce conditional access and multi‑factor policies  

<!-- ------------------------- ------------------------- -->

### Authorization
...controls what an identity can do

- **Role-Based**: A role is a **named group of permissions**
  - ****SQL Server****: Has built‑in roles such as data **reader**, data **writer**, and database **owner*-  
  - **Windows**: Create a SQL Login corresponding to a Windows user or group
    - Assign that login to the appropriate **SQL Server** role
    - Upshot: Active Directory manages membership 
- **Row Level Security**: restrict access to table rows based on the executing user’s context  
  - **SQL Server**: create an inline table‑valued function and enforce it via CREATE SECURITY POLICY … ADD FILTER PREDICATE to apply row‑level filters  
  - **Azure SQL**: use the same T‑SQL security policy approach, leveraging Entra ID principals or SESSION_CONTEXT to drive the filter logic

<!-- ------------------------- ------------------------- -->

### Scope  
...defines what is secured by authorization  

- **Server**: the entire SQL instance (logins, server roles)  
  - **SQL Server**: physical instance with full server roles such as "sysadmin" and "serveradmin"  
  - **Azure SQL**: logical server with limited built‑in roles and an Entra ID administrator

- **Database**: a single database (database roles, settings)  
  - **SQL Server**: supports Windows Authentication and **SQL Server** Authentication for logins  
  - **Azure SQL**: uses contained database users and database‑scoped roles only  

- **Schema**: a group of objects in a database  
  - **SQL Server** and **Azure SQL**: identical behavior for schema‑scoped permissions  

- **Object**: a table, view, or stored procedure  
  - **SQL Server** and **Azure SQL**: identical behavior for object‑level permissions  

- **Column**: a field in a table (for example encryption or masking)  
  - **SQL Server** and **Azure SQL**: identical behavior for column‑level permissions  

- **Row**: individual records (row‑level security)  
  - **SQL Server** and **Azure SQL**: both support security policies with FILTER PREDICATE definitions  

- **Network**: client connections  
  - **SQL Server**: relies on operating‑system or network firewall rules and TCP port configuration  
  - **Azure SQL**: uses **Azure SQL** firewall rules, virtual network service endpoints, and private endpoints with TLS enforced by default

<!-- ------------------------- ------------------------- -->

### Connectivity 
...restrict client connectivity using network controls  

- **Network Isolation**: configure firewalls, service endpoints, private endpoints, virtual network rules and VPNs  
  - **SQL Server**: manage Windows firewall rules or network ACLs to limit access to the database port  
  - **Azure SQL**: define server‑level firewall rules, enable virtual network service endpoints or private endpoints  

<!-- ------------------------- ------------------------- -->

### Encryption  
...ensures data remains unreadable by unauthorized parties  

- **At Rest**: data stored on disk or in backups

  - **Transparent Data Encryption**: encrypts the entire database on disk  
    - **SQL Server**: create a database master key and certificate, then run ALTER DATABASE … SET ENCRYPTION ON  
    - **Azure SQL**: enabled by default; manage keys via Azure Key Vault or use service‑managed keys in the Azure portal

  - **Column‑Level Encryption**: encrypts specific sensitive columns  
    - **SQL Server**: create a database master key, certificate, and symmetric key; use the ENCRYPTBYKEY and DECRYPTBYKEY functions in T‑SQL to secure column data  
    - **Azure SQL**: use the same T‑SQL encryption functions with keys managed in Azure Key Vault or service‑managed keys, configured via the Azure portal or PowerShell  

- **In Transit**: data moving across the network  

  - **Transport Layer Security (TLS)**: encrypts network traffic between client and server  
    - **SQL Server**: install a valid server certificate, enable “Force Encryption” in **SQL Server** Configuration Manager, and require clients to use Encrypt=True in their connection strings  
    - **Azure SQL**: TLS is enforced by default for all connections (minimum TLS 1.2); configure the minimum TLS version in the Azure portal under your SQL server’s Security settings

  - **Secure Sockets Layer (SSL)**: legacy protocol that also encrypts network traffic (superseded by TLS)  
    - **SQL Server**: older versions support SSL 3.0; enable or disable via **SQL Server** Configuration Manager or Windows registry, though it is disabled by default in recent releases for security  
    - **Azure SQL**: does not support SSL; any legacy SSL connection attempts are rejected and only TLS 1.2 or higher is allowed
 
- **In Use**: data loaded into memory for processing

  - **Always Encrypted**: encrypts sensitive columns on the client so **SQL Server** only ever sees encrypted data  
    - **SQL Server**: configure column master keys and column encryption keys using **SQL Server** Management Studio or PowerShell; enable client‑side encryption by specifying Column Encryption Setting=Enabled in the connection string  
    - **Azure SQL**: integrate with Azure Key Vault for key management; enable Always Encrypted through the Azure portal or PowerShell and set Column Encryption Setting=Enabled in the connection string

- **Storage Encryption**: apply encryption at the storage layer to protect database files and backups at rest  
  - **SQL Server**: use BitLocker to encrypt volumes hosting database files and ensure backup targets reside on encrypted media  
  - **Azure SQL**: rely on Azure Storage Service Encryption, which automatically encrypts database files and backups using Microsoft‑managed or customer‑managed keys in Azure Key Vault  

<!-- ------------------------- ------------------------- -->

### Key Management  
…secure and rotate cryptographic keys that underpin encryption  

- **Key Management**: manage keys for database encryption features  
  - **SQL Server**: store keys in a database master key protected by a certificate or hardware security module via Extensible Key Management; rotate keys using ALTER MASTER KEY and certificate commands  
  - **Azure SQL**: integrate with Azure Key Vault for service‑managed or customer‑managed keys; configure rotation policies and access controls for Transparent Data Encryption and Always Encrypted  

<!-- ------------------------- ------------------------- -->

### Governance
...policies and controls to identify and protect sensitive data

- **Sensitivity Classification**: assign metadata to columns indicating their sensitivity  
  - **SQL Server**: use the sys.sp_add_sensitivity_classification stored procedure or SSMS Data Classification pane to label columns; query sys.sensitivity_classifications to review existing labels  
  - **Azure SQL**: run “Data discovery & classification” scans in the Azure portal or call sys.sp_add_sensitivity_classification via T‑SQL; view and manage labels in the portal or via sys.sensitivity_classifications  

- **Dynamic Data Masking**: define masking rules to obscure sensitive column data **at query time**  
  - **SQL Server**: use ALTER TABLE … ALTER COLUMN … ADD MASKED WITH (FUNCTION = 'default()' or custom functions); manage masks in SSMS under Security > Dynamic Data Masking  
  - **Azure SQL**: configure masks in the Azure portal “Dynamic Data Masking” blade or via the same T‑SQL ADD MASKED statements; supports default, email, and partial masks  

- **Microsoft Purview**: catalog and govern data assets with automated scanning and classification  
  - **SQL Server**: register on‑premises instances with Purview and deploy the scanning integration to discover and classify sensitive data; review scan results and export compliance reports in the Purview portal  
  - **Azure SQL**: enable Purview scanning directly against **Azure SQL** servers in the portal; schedule recurring scans, validate or update classifications, and generate built‑in compliance dashboards

- **Sensitivity labels integration**: leverage Microsoft Information Protection to apply and enforce labels across databases and tables  
  - **SQL Server**: install and configure the Azure Information Protection scanner or use MIP PowerShell cmdlets to label on‑premises databases registered with Purview  
  - **Azure SQL**: integrate with Microsoft Information Protection in the Azure portal or via PowerShell to apply sensitivity labels at scale and ensure metadata flows through downstream services  

<!-- ------------------------- ------------------------- -->

### Lifecycle Management  
…manage data retention, purging, and archival for compliance  

- **Retention Policies**: implement temporal tables or partition switching to expire data after a set period  
  - **SQL Server**: configure system‑versioned temporal tables with HISTORY_RETENTION_PERIOD or use partition switching to archive old data  
  - **Azure SQL**: use temporal tables plus an Azure Automation runbook to clean up history or export snapshots to Blob storage  

- **Purge Jobs**: schedule deletion tasks to remove stale or expired data  
  - **SQL Server**: create **SQL Server** Agent jobs that run DELETE or EXEC stored procedures on a cadence  
  - **Azure SQL**: use Elastic Jobs or Azure Automation runbooks to execute purge scripts  

- **Archival**: move historical data to lower‑cost storage for long‑term retention  
  - **SQL Server**: export data via BCP or PolyBase into Azure Blob or Data Lake  
  - **Azure SQL**: use Azure Data Factory or Elastic Query to offload data to Data Lake  

<!-- ------------------------- ------------------------- -->

### Auditing  
…record security‑relevant events for accountability  

- **Auditing**: configure audit logs to capture actions such as schema changes, logins, and security policy modifications  
  - **SQL Server**: create a Server Audit and Server Audit Specification, target FILE or Windows Application log, then enable both  
  - **Azure SQL**: enable **Azure SQL** Auditing at the server or database scope in the portal or via PowerShell, and send logs to Log Analytics, Storage, or Event Hubs  

<!-- ------------------------- ------------------------- -->

### Monitoring  
...logs, metrics, alerts used to understand activity and exceptions

- **Monitoring**: collect and review logs and metrics to understand normal activity  
  - **SQL Server**: query dynamic management views (for example, sys.dm_exec_sessions and sys.dm_os_wait_stats) to track server and session activity  
  - **Azure SQL**: use Azure Monitor (Log Analytics) to collect logs, metrics, and configure alerts

<!-- ------------------------- ------------------------- -->

### Threat Detection  
...identify and alert on suspicious or malicious behavior  

- **Advanced Threat Protection**: built‑in analytics that flag unusual activities before they become incidents  
  - **SQL Server**: deploy Extended Events and **SQL Server** Audit to capture and analyze suspect events  
  - **Azure SQL**: enable "Microsoft Defender for SQL" to flag unusual activities before they become incidents

- **Anomaly Detection**: automated identification of irregular patterns that may indicate threats  
  - **SQL Server**: run custom T‑SQL queries against dynamic management views or audit logs to spot patterns such as repeated login failures or unexpected queries  
  - **Azure SQL**: leverage anomaly detection in "Microsoft Defender for SQL" to automatically surface unusual behavior
 
- **Alerting**: mechanisms to notify stakeholders when potential threats are detected  
  - **SQL Server**: configure Database Mail and **SQL Server** Agent alerts to send notifications on threat events  
  - **Azure SQL**: create Azure Monitor alerts on threat detection logs to send emails, SMS, or trigger automated responses

<!-- ------------------------- ------------------------- -->

### Recurring Maintenance  
...regular activities to ensure continued security and compliance  

- **Audit Reviews**: schedule regular examination of audit logs and role memberships  
  - **SQL Server**: schedule **SQL Server** Agent jobs to export and review audit data via sys.fn_get_audit_file and check server role mappings  
  - **Azure SQL**: use Azure Automation runbooks or Logic Apps to retrieve audit logs from Azure Monitor and review database‑scoped role assignments  

- **Permission Reviews**: periodically verify user and group access  
  - **SQL Server**: query sys.server_principals, sys.database_principals, and sp_helprotect with scheduled T‑SQL scripts  
  - **Azure SQL**: run PowerShell scripts against Entra ID and query contained database users to confirm assignments  

- **Patch Scheduling**: apply security updates on a defined cadence  
  - **SQL Server**: coordinate Windows Update or WSUS schedules with maintenance windows to apply **SQL Server** patches  
  - **Azure SQL**: configure maintenance windows and update control in the Azure portal  

- **Backup & Restore Tests**: validate backup and recovery procedures  
  - **SQL Server**: use maintenance plans or custom T‑SQL jobs to run BACKUP, RESTORE VERIFYONLY and test restores to sandbox instances  
  - **Azure SQL**: perform point‑in‑time restores or database copies to test recovery workflows  

- **Security Drills**: conduct tabletop exercises and vulnerability scans regularly  
  - **SQL Server**: run the built‑in vulnerability assessment in **SQL Server** Management Studio and simulate attack scenarios  
  - **Azure SQL**: leverage Microsoft Defender for SQL vulnerability assessment and simulate failover or breach exercises

<!-- ------------------------- ------------------------- -->

### Best Practices 
...recommended guidelines to establish and maintain a secure and compliant SQL environment

- **Access and Identity Management**  
  - **Least‑Privilege Principle**: grant **only the permissions each user needs** by using built‑in roles and assigning them to users or Active Directory groups  
  - **Authentication Choice**: prefer **Windows Authentication** for domain users <s>and SQL Authentication for service or non‑domain accounts</s>

- **Data Protection**  
  - **Encryption Everywhere**: enforce encryption at rest, in transit, and in use  
  - **Surface Area Reduction**: disable unused features, services, and ports  

- **Maintenance and Compliance**  
  - **Patch Management**: schedule and apply operating system and **SQL Server** security updates on a defined cadence  
  - **Regular Permission Reviews**: audit role memberships and login mappings on a schedule to verify access remains appropriate  
  - **Regular Audits & Reviews**: schedule log and policy reviews to ensure ongoing compliance  
  - **Periodic Security Assessments**: run penetration tests and vulnerability scans regularly  
  - **Incident Response Planning**: develop and test clear recovery and response plans  

- **Governance and Automation**  
  - **Automated Enforcement**: use “Azure Policy” to automatically remediate non‑compliant SQL resources  
  - **Policy Documentation**: maintain and update security policies to support audits and regulatory oversight  
  - **Industry Benchmark Hardening**: apply guides from the Center for Internet Security and other standards

<!-- ------------------------- ------------------------- -->

#### Platform Differences

#### **Azure SQL** Database
- **Managed Security**:  
  - Built‑in threat detection and automated patching  
  - Limited direct configuration of underlying infrastructure  
- **Entra ID Integration**:  
  - Native support for Entra ID authentication simplifies identity management

#### **Azure SQL** Managed Instance
- **Hybrid Control**:  
  - Combines traditional **SQL Server** security features with Azure’s managed environment  
  - Supports both TDE and Always Encrypted with advanced key management via Azure Key Vault  
- **Enhanced Auditing**:  
  - Offers robust audit capabilities that integrate with Azure Monitor and Log Analytics

#### **SQL Server** on Azure VMs
- **Full Control**:  
  - Apply traditional on-prem security hardening techniques  
  - Custom configuration of auditing, encryption, and network isolation  
- **Custom Security Solutions**:  
  - Integrate third‑party tools or tailored configurations as required

<!-- ------------------------- ------------------------- -->

## On‑Prem

### Exercise

##### Prepare Environment

Set up a **SQL Server** instance with the following:
  - ****SQL Server** Management Studio (SSMS)**
  - **AdventureWorks*- sample database (for testing various security configurations)

##### Configure Security

- **Step 1: Enable Auditing*-  
  Execute the following T‑SQL to begin auditing:
  ```sql
  CREATE SERVER AUDIT SQLSecurityAudit
    TO FILE (FILEPATH = 'C:\AuditLogs\', MAXSIZE = 10 MB);
  ALTER SERVER AUDIT SQLSecurityAudit WITH (STATE = ON);
  GO
  ```

- **Step 2: Implement Encryption*-  
  Configure Transparent Data Encryption on the AdventureWorks database:
  ```sql
  USE master;
  GO
  CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'YourStrongPassword!';
  GO
  CREATE CERTIFICATE TDECert WITH SUBJECT = 'TDE Certificate';
  GO
  ALTER DATABASE AdventureWorks SET ENCRYPTION ON;
  GO
  ```

- **Step 3: Set Up Role‑Based Access*-  
  Create a custom role and assign it to a test user:
  ```sql
  CREATE ROLE SecurityOperator;
  GRANT VIEW SERVER STATE TO SecurityOperator;
  GO
  EXEC sp_addrolemember 'SecurityOperator', 'YourUserName';
  GO
  ```

------------------------- -------------------------

### Quiz

1. What is the primary purpose of Transparent Data Encryption (TDE) in **SQL Server**?  
   A) To speed up data retrieval  
   B) To encrypt data at rest  
   C) To simplify backup processes  
   D) To enable point‑in‑time recovery  

2. Which authentication method integrates with the operating system for secure access?  
   A) Mixed Mode  
   B) SQL Authentication  
   C) Windows Authentication  
   D) Certificate Authentication  

3. What additional measure encrypts data on the client side before transmission to the server?  
   A) Column‑Level Encryption  
   B) Always Encrypted  
   C) TDE  
   D) TLS/SSL  

4. How do sensitivity labels and dynamic data masking assist in compliance?  
   A) They automate patching  
   B) They classify and partially obscure sensitive data  
   C) They encrypt backup files  
   D) They restrict network access  

5. Which **SQL Server** feature is used to log security‑related events?  
   A) Database Mail  
   B) **SQL Server** Audit  
   C) **SQL Server** Agent  
   D) Dynamic Management Views (DMVs)

6. What network-level security measure should be implemented on-prem?  
   A) VPN and firewall configurations  
   B) Dynamic data masking  
   C) Always Encrypted  
   D) Transparent Data Encryption

------------------------- -------------------------

#### Answers

1. **B*- – TDE encrypts data at rest to protect physical storage  
2. **C*- – Windows Authentication leverages domain credentials for secure access  
3. **B*- – Always Encrypted ensures encryption occurs on the client side  
4. **B*- – Sensitivity labels classify data and dynamic data masking obscures sensitive data  
5. **B*- – **SQL Server** Audit logs security‑related events  
6. **A*- – Using VPNs, IP whitelisting, and firewalls helps limit network access

------------------------- -------------------------

## Azure

### Exercise

- **Step 1**: Log in to the Azure Portal and navigate to your **SQL Server** instance  
- **Step 2**: Enable Advanced Threat Protection from the Security settings  
- **Step 3**: Configure auditing for your **Azure SQL** Database via the portal (e.g., directing logs to Azure Storage or Log Analytics)  
- **Step 4**: Review and apply network isolation settings using service or private endpoints  
- **Step 5**: Use Azure Policy to assign compliance initiatives and run remediation tasks to ensure all deployments meet required standards

------------------------- -------------------------

### Quiz

1. Which **Azure SQL** Database feature helps automatically detect potential security threats?  
   A) Automated Backups  
   B) Advanced Threat Protection  
   C) Log Shipping  
   D) Dynamic Data Masking  

2. What advantage does **Azure SQL** Managed Instance offer for security?  
   A) No need for patching  
   B) A hybrid approach with both managed and traditional security controls  
   C) Full direct access to hardware  
   D) Limited auditing capabilities  

3. For **SQL Server** on Azure VMs, which statement is true regarding security?  
   A) They automatically apply all security settings  
   B) They allow full control over security configurations  
   C) They have no built‑in security features  
   D) They cannot integrate third‑party security tools  

4. Which authentication method centralizes identity management for Azure environments?  
   A) SQL Authentication  
   B) Windows Authentication  
   C) Entra ID Authentication  
   D) Certificate Authentication

------------------------- -------------------------

#### Answers

1. **B*- – Advanced Threat Protection helps identify and remediate security threats automatically  
2. **B*- – **Azure SQL** Managed Instance combines managed security with traditional features for greater control  
3. **B*- – **SQL Server** on Azure VMs allows full control over security configurations  
4. **C*- – Entra ID Authentication provides centralized identity management and integrated conditional access