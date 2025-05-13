# Power BI: Security

> This is a draft outline for a possible curriculum

# Source Data Security

| Topic | Description | Technology Focus |
| :--- | :--- | :--- |
| Azure AD authentication & RBAC for data sources | Use Entra ID tokens and security groups to grant least-privilege access to source systems (e.g. Azure SQL) | SQL |
| Source data protection policies | Classify sensitive columns and apply masking or client-side encryption before data reaches Power BI | SQL |

# Ingestion & Storage Security

| Topic | Description | Technology Focus |
| :--- | :--- | :--- |
| Staging zones—bronze, silver, gold | Segregate raw, cleansed, and curated data into layers with distinct access controls | Fabric |
| Secure pipelines & dataflows | Use managed identities or service principals and Azure AD OAuth for all ETL connections and secret management | ADF / Fabric |
| Data lakehouse security | Enforce OneLake hierarchical ACLs and workspace permissions on Fabric Lakehouses or Gen2 storage | Fabric |
| On-premises data gateway protection | Restrict gateway admins, encrypt stored credentials, keep gateway software patched and clustered | Gateway |

# Transformation Security

| Topic | Description | Technology Focus |
| :--- | :--- | :--- |
| Controlled & audited transformations | Limit and track who can create or modify Power Query, dataflow, or notebook logic | Power BI / Fabric |
| Sensitive data handling in ETL | Mask, hash, remove, or aggregate PII during transformation so only necessary fields enter the BI model | Power BI / Fabric |

# Data Model Security

| Topic | Description | Technology Focus |
| :--- | :--- | :--- |
| Row-level security | Define filter roles in Power BI Desktop tied to Entra ID groups so users see only their permitted rows | Power BI |
| Object-level security | Hide tables or columns from unauthorized viewers by assigning deny-access roles | Power BI |
| Customer-managed keys | (Premium only) Bring your own key from Azure Key Vault to control encryption of imported models | Power BI |
| Dataset credential management | Store and rotate source credentials securely, preferring Azure AD OAuth over stored passwords | Power BI |

# Presentation Security

| Topic | Description | Technology Focus |
| :--- | :--- | :--- |
| Workspace & app access control | Assign Entra ID groups to workspace and app roles to manage who can view or edit content | Power BI |
| Secure sharing practices | Share via security groups, disable publish-to-web, enforce tenant-level export restrictions | Power BI |
| Service principal embedding | Use an AAD application identity for app-owns-data scenarios with controlled embed tokens | Power BI |
| Sensitivity labels & data protection | Apply Purview labels to reports/datasets so exports carry encryption and usage policies | Power BI / Purview |
| Custom visuals governance | Allow only certified or approved visuals to prevent unauthorized data exfiltration | Power BI |
| Tenant settings & conditional access | Harden the tenant by disabling risky features and enforcing MFA or IP restrictions | Power BI |

# Governance & Monitoring

| Topic | Description | Technology Focus |
| :--- | :--- | :--- |
| Audit logs & monitoring | Enable and review Power BI audit logs in M365 or Azure Monitor to detect anomalous activities | Power BI |
| Data loss prevention policies | Define Purview DLP rules to automatically detect and remediate sensitive data exposure | Purview |
| Unified governance & catalog | Integrate with Purview for end-to-end lineage, classification, and impact analysis | Purview |

# Future Topics

| Topic | Description | Technology Focus |
| :--- | :--- | :--- |
| External sharing & B2B collaboration | Securely share with guests via Azure AD B2B and manage external user permissions | Power BI |
| Network isolation & private endpoints | Use private endpoints or VNet integration to restrict Power BI/Fabric service access | Fabric / Networking |
