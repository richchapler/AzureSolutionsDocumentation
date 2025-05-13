# Power BI: Security

> This is a draft outline for a possible curriculum

# Source Data Security

| Topic | Description | Technology Focus |
| :--- | :--- | :--- |
| Azure AD authentication & RBAC for data sources | Use Entra ID tokens and security groups to grant least-privilege access to source systems (e.g. Azure SQL) | SQL |
| Static data masking | Replace sensitive data at rest in the database so non-privileged users only see masked values | SQL |
| Dynamic data masking | Configure Azure SQL to mask column values at query time based on user roles, without altering underlying data | SQL |

# Ingestion & Storage Security

| Topic | Description | Technology Focus |
| :--- | :--- | :--- |
| Staging zones—bronze, silver, gold | Segregate raw, cleansed, and curated data into layers with distinct access controls | Fabric |
| Secure pipelines & dataflows | Use managed identities or service principals and Azure AD OAuth for all ETL connections and secret management | ADF / Fabric |
| Data lakehouse security | Enforce OneLake hierarchical ACLs and workspace permissions on Fabric Lakehouses or Gen2 storage | Fabric |
| On-premises data gateway protection | Restrict gateway admins, encrypt stored credentials, and keep gateway software patched and clustered | Gateway |

# Transformation Security

| Topic | Description | Technology Focus |
| :--- | :--- | :--- |
| Controlled & audited transformations | Limit and track who can create or modify Power Query, dataflow, or notebook logic | Power BI / Fabric |
| Sensitive data handling in ETL | Mask, hash, remove, or aggregate PII during transformation so only necessary fields enter the BI model | Power BI / Fabric |

# Data Model Security

| Topic | Description | Technology Focus |
| :--- | :--- | :--- |
| Row-level security (RLS) | Define filter roles in Power BI Desktop tied to Entra ID groups so users see only their permitted rows | Power BI |
| Object-level security (OLS) | Hide entire tables or columns from unauthorized viewers by assigning deny-access roles | Power BI |
| In-model data masking | Use DAX formulas to mask or truncate values in the report layer based on user role | Power BI |
| Sensitivity labels | Apply Microsoft Purview labels to datasets so exports and downloads carry persistent protection | Power BI / Purview |
| Customer-managed keys (BYOK) | (Premium only) Bring your own key from Azure Key Vault to control encryption of imported models | Power BI |
| Dataset credential management | Store and rotate source credentials securely, preferring Azure AD OAuth over passwords | Power BI |

# Presentation Security

| Topic | Description | Technology Focus |
| :--- | :--- | :--- |
| Workspace & app access control | Assign Entra ID groups to Admin, Member, Contributor, and Viewer roles in workspaces and Apps | Power BI |
| Granular dashboard access | Enforce RLS/OLS on datasets and use workspace roles so dashboards display only authorized data | Power BI |
| Secure sharing practices | Share via security groups, disable Publish-to-Web, and enforce tenant-level export restrictions | Power BI |
| Service principal embedding | Use an AAD application identity for app-owns-data scenarios with controlled embed tokens | Power BI |
| Custom visuals governance | Allow only certified or approved organizational visuals to prevent unauthorized data exfiltration | Power BI |
| Tenant settings & conditional access | Harden the tenant by disabling risky features, enforcing MFA, and limiting IP ranges | Power BI |

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
