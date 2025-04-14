# SQL: Security and Compliance

## Use Case

The database administration team at a mid‑sized organization must ensure that critical data is protected from unauthorized access, breaches, and non‑compliance with regulatory standards. Key requirements include:

* **Data Protection**: Shield sensitive information from internal and external threats  
* **Network Isolation**: Restrict connectivity so that only approved users and systems access the databases  
* **Auditing and Monitoring**: Track security events, classify sensitive data, and monitor user activity to detect anomalies  
* **Operational Continuity**: Ensure systems remain secure and available during maintenance and emergencies  
* **Regulatory Compliance**: Meet legal and regulatory requirements; examples:  
  * **General Data Protection Regulation (GDPR)**: European Union regulation protecting personal data and privacy  
  * **Health Insurance Portability and Accountability Act (HIPAA)**: United States law safeguarding patient health information  
  * **Payment Card Industry Data Security Standard (PCI‑DSS)**: Standards to secure credit and debit card data  
  * **Sarbanes‑Oxley Act (SOX)**: United States law enforcing strict financial reporting and internal controls to prevent fraud

<!-- ------------------------- ------------------------- -->

## On‑Prem

### Fundamentals

#### Authentication
...confirms an identity

* **Windows**: Leverage domain credentials for secure, integrated access  
* **SQL Server**: Authenticate using SQL Server–specific credentials (username and password), suitable when Windows Authentication is not used  
* **Multi‑Factor**: Enforce a second verification step via Windows or Active Directory using smart cards or Azure Active Directory multi‑factor authentication; note that multi‑factor authentication is not supported for SQL Server Authentication  

<!-- ------------------------- ------------------------- -->

#### Authorization
...controls what an identity can do

* **Role-Based**: A role is a **named group of permissions**
  * **SQL Server**: Has built‑in roles such as data **reader**, data **writer**, and database **owner**  
  * **Windows**: Create a SQL Login corresponding to a Windows user or group
    * Assign that login to the appropriate SQL Server role
    * Upshot: Active Directory manages membership 
* **Row Level Security**: Configure filters so each user only sees the rows they are allowed to view based on their identity  

<!-- ------------------------- ------------------------- -->

#### Scope  
...defines what is secured by authorization  

* **Server**: the entire SQL Server instance (logins, server roles)  
* **Database**: a single database (database roles, settings)  
* **Schema**: a group of objects in a database  
* **Object**: a table, view, or stored procedure  
* **Column**: a field in a table (encryption, masking)  
* **Row**: individual records (row‑level security)  
* **Network**: client connections (firewalls, network rules)

<!-- ------------------------- ------------------------- -->

#### Encryption  
...ensures data remains unreadable by unauthorized parties  

* **At Rest**: data stored on disk or in backups  
  * **Transparent Data Encryption**: SQL Server feature that encrypts the entire database on disk  
  * **Column‑Level Encryption**: encrypts specific sensitive columns  
* **In Transit**: data moving across the network  
  * **Transport Layer Security (TLS)**: encrypts network traffic between client and server  
  * **Secure Sockets Layer (SSL)**: legacy protocol that also encrypts network traffic (superseded by TLS)  
* **In Use**: data loaded into memory for processing  
  * **Always Encrypted**: encrypts sensitive columns on the client so **SQL Server only ever sees encrypted data**

<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Data Governance (RESUME HERE)
…policies and controls to identify and protect sensitive data  
* **Sensitivity Labels**: mark tables and columns (for example, personal or financial data) so you know exactly what needs protection  
* **Dynamic Data Masking**: automatically obscure values in those labeled columns for users who shouldn’t see the real data  
* 
<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->
<!-- ------------------------- ------------------------- -->

#### Monitoring  
…collect and review logs and metrics to understand normal activity  
* **Continuous Monitoring**: use built‑in tools like dynamic management views or Azure Monitor to track server and database activity  
* **Scheduled Reviews**: audit logs and configurations on a regular cadence to catch misconfigurations or gaps  
* **Key Metrics**: track how quickly issues are detected, how fast they’re addressed, how often audits run, and overall compliance rates  

#### Threat Detection  
…identify and alert on suspicious or malicious behavior  
* **Advanced Threat Protection**: built‑in analytics that flag unusual activities before they become incidents  
* **Anomaly Detection**: spot patterns such as repeated login failures or unexpected queries  
* **Alerting**: configure notifications so the right teams are informed immediately when a threat is detected

#### Compliance Requirements  
* **General Data Protection Regulation**: European regulation protecting personal data and privacy  
* **Health Insurance Portability and Accountability Act**: United States law safeguarding patient health information  
* **Payment Card Industry Data Security Standard**: Standards to secure credit and debit card data  
* **Sarbanes‑Oxley Act**: United States law enforcing strict financial reporting and internal controls to prevent fraud  

#### Policy Management  
* **Policy Documentation**: Maintain and update security policies to support audits and regulatory oversight  
* **Automated Enforcement**: Use tools such as Azure Policy to ensure configurations remain compliant with evolving standards  

#### Security Baselines and Hardening  
* **Industry Benchmarks**: Apply standard hardening guides such as those from the Center for Internet Security  
* **Surface Area Reduction**: Disable unused features, services, and ports  
* **Patch Management**: Keep operating systems and SQL Server up to date  

#### Best Practices  
* **Least‑Privilege Principle**: Grant only the permissions each user needs.  
  - Use built‑in roles (e.g., data reader, data writer) and assign users or AD groups to them.  
* **Authentication Choice**:  
  - Prefer Windows Authentication for domain users.  
  - Use SQL Server Authentication for service/non‑domain accounts.  
* **Regular Permission Reviews**:  
  - Audit role memberships and login mappings on a schedule.  
  - Revoke or adjust access as roles and responsibilities change.  
* **Encryption Everywhere**: Enforce encryption at rest, in transit, and in use.  
* **Regular Audits & Reviews**: Schedule log and policy reviews.  
* **Incident Response Planning**: Develop and test clear recovery and response plans.  
* **Periodic Security Assessments**: Run penetration tests and vulnerability scans.

Recurring Maintenance
Audit Reviews: schedule regular reviews of audit logs and role memberships

Permission Reviews: periodically verify user and group access

Patch Scheduling: apply security updates on a defined cadence

Backup & Restore Tests: routinely test backups and recovery procedures

Security Drills: conduct tabletop exercises and vulnerability scans regularly*  

<!-- ------------------------- ------------------------- -->

### Exercise

##### Prepare Environment

Set up a SQL Server instance with the following:
  * **SQL Server Management Studio (SSMS)**
  * **AdventureWorks** sample database (for testing various security configurations)

##### Configure Security

* **Step 1: Enable Auditing**  
  Execute the following T‑SQL to begin auditing:
  ```sql
  CREATE SERVER AUDIT SQLSecurityAudit
    TO FILE (FILEPATH = 'C:\AuditLogs\', MAXSIZE = 10 MB);
  ALTER SERVER AUDIT SQLSecurityAudit WITH (STATE = ON);
  GO
  ```

* **Step 2: Implement Encryption**  
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

* **Step 3: Set Up Role‑Based Access**  
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

1. What is the primary purpose of Transparent Data Encryption (TDE) in SQL Server?  
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

5. Which SQL Server feature is used to log security‑related events?  
   A) Database Mail  
   B) SQL Server Audit  
   C) SQL Server Agent  
   D) Dynamic Management Views (DMVs)

6. What network-level security measure should be implemented on‑premises?  
   A) VPN and firewall configurations  
   B) Dynamic data masking  
   C) Always Encrypted  
   D) Transparent Data Encryption

------------------------- -------------------------

#### Answers

1. **B** – TDE encrypts data at rest to protect physical storage  
2. **C** – Windows Authentication leverages domain credentials for secure access  
3. **B** – Always Encrypted ensures encryption occurs on the client side  
4. **B** – Sensitivity labels classify data and dynamic data masking obscures sensitive data  
5. **B** – SQL Server Audit logs security‑related events  
6. **A** – Using VPNs, IP whitelisting, and firewalls helps limit network access

------------------------- -------------------------

## Azure

### Fundamentals

Azure SQL environments come with built‑in security and compliance features that differ from on‑premises implementations. They leverage a shared responsibility model, where Microsoft manages certain aspects of security while customers focus on configuration and access control.

#### Encryption

* **At Rest**:  
  * **TDE**: Enabled by default on Azure SQL Database and Managed Instances  
  * **Storage Encryption**: Backups and Azure Blob storage are encrypted automatically  
* **In Transit**:  
  * **TLS/SSL**: All client connections enforce TLS for secure data transfer  
* **In Use (Always Encrypted)**:  
  * **Implementation**: Always Encrypted is supported for protecting sensitive data so that even administrators cannot view it  
  * **Key Management**: Integration with Azure Key Vault allows both auto‑managed and customer‑managed keys  
  * **Considerations**: Be mindful of limitations such as query restrictions and potential performance impacts

#### Auditing, Monitoring & Data Classification

* **Azure SQL Auditing**:  
  * **Built‑in Auditing**: Automatically logs activity to Azure Storage, Log Analytics, or Event Hubs  
  * **Configuration**: Can be managed through the Azure portal with minimal setup  
* **Sensitivity Labels**:  
  * **Integration with MIP**: Leverage Microsoft Information Protection to apply and manage sensitivity labels that classify data and ensure compliance  
* **Advanced Threat Protection**:  
  * **Built‑in Features**: Azure Advanced Threat Protection (now part of Microsoft Defender for SQL) monitors for anomalous activities and potential security breaches

#### Network Isolation & Access Control

* **Network-Level Restrictions**:  
  * **Service Endpoints & Private Endpoints**: Use these features to restrict connectivity so that only approved virtual networks or systems can access Azure SQL databases  
  * **Firewall Rules**: Configure database and server firewalls to limit access to specific IP ranges

#### Azure Policy and Compliance Management

* **Azure Policy Initiatives**:  
  * **Enforcement**: Use initiatives to automatically apply and enforce security configurations across all Azure SQL deployments  
  * **Remediation**: Run remediation tasks to ensure existing resources meet defined compliance standards  
  * **Consistency**: Helps manage and monitor compliance across deployments with minimal administrative overhead

#### Azure AD Authentication

* **Identity-Based Access Control**:  
  * **Azure AD Integration**: Use Azure Active Directory (Azure AD) for centralized identity management, single sign‑on (SSO), and multi‑factor authentication  
  * **Configuration**: Create contained database users using the FROM EXTERNAL PROVIDER clause and configure an Azure AD administrator for the SQL server  
  * **Conditional Access**: Leverage conditional access policies to enforce additional verification based on risk factors such as location and device

#### Platform Differences

#### Azure SQL Database
* **Managed Security**:  
  * Built‑in threat detection and automated patching  
  * Limited direct configuration of underlying infrastructure  
* **Azure AD Integration**:  
  * Native support for Azure AD authentication simplifies identity management

#### Azure SQL Managed Instance
* **Hybrid Control**:  
  * Combines traditional SQL Server security features with Azure’s managed environment  
  * Supports both TDE and Always Encrypted with advanced key management via Azure Key Vault  
* **Enhanced Auditing**:  
  * Offers robust audit capabilities that integrate with Azure Monitor and Log Analytics

#### SQL Server on Azure VMs
* **Full Control**:  
  * Apply traditional on‑premises security hardening techniques  
  * Custom configuration of auditing, encryption, and network isolation  
* **Custom Security Solutions**:  
  * Integrate third‑party tools or tailored configurations as required

------------------------- -------------------------

### Exercise

* **Step 1**: Log in to the Azure Portal and navigate to your SQL Server instance  
* **Step 2**: Enable Advanced Threat Protection from the Security settings  
* **Step 3**: Configure auditing for your Azure SQL Database via the portal (e.g., directing logs to Azure Storage or Log Analytics)  
* **Step 4**: Review and apply network isolation settings using service or private endpoints  
* **Step 5**: Use Azure Policy to assign compliance initiatives and run remediation tasks to ensure all deployments meet required standards

------------------------- -------------------------

### Quiz

1. Which Azure SQL Database feature helps automatically detect potential security threats?  
   A) Automated Backups  
   B) Advanced Threat Protection  
   C) Log Shipping  
   D) Dynamic Data Masking  

2. What advantage does Azure SQL Managed Instance offer for security?  
   A) No need for patching  
   B) A hybrid approach with both managed and traditional security controls  
   C) Full direct access to hardware  
   D) Limited auditing capabilities  

3. For SQL Server on Azure VMs, which statement is true regarding security?  
   A) They automatically apply all security settings  
   B) They allow full control over security configurations  
   C) They have no built‑in security features  
   D) They cannot integrate third‑party security tools  

4. Which authentication method centralizes identity management for Azure environments?  
   A) SQL Authentication  
   B) Windows Authentication  
   C) Azure AD Authentication  
   D) Certificate Authentication

------------------------- -------------------------

#### Answers

1. **B** – Advanced Threat Protection helps identify and remediate security threats automatically  
2. **B** – Azure SQL Managed Instance combines managed security with traditional features for greater control  
3. **B** – SQL Server on Azure VMs allows full control over security configurations  
4. **C** – Azure AD Authentication provides centralized identity management and integrated conditional access