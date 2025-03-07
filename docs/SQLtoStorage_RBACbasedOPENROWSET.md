# SQL >> Storage Connectivity: RBAC-based OPENROWSET

This guide shows how to query a JSON file in Azure Storage from Azure SQL using a system-assigned managed identity and RBAC. It uses a **container-level** external data source so that the container name doesn’t need to be repeated in the file path.

---

## Steps

1 Enable the system-assigned managed identity  
   - In the Azure portal, open your SQL resource (server or managed instance)  
   - Go to Identity  
   - Switch the system-assigned identity to On  

2 Grant the Storage Blob Data Reader role  
   - In the portal, open your storage account  
   - Go to Access Control (IAM)  
   - Add a role assignment for Storage Blob Data Reader  
   - Assign access to Managed identity and choose your SQL resource  

3 Create a database scoped credential  
   - Connect to your Azure SQL database  
   - Run  

   ```sql
   CREATE DATABASE SCOPED CREDENTIAL ManagedIdentityCredential
   WITH IDENTITY = 'Managed Identity';
   ```

4 Create a container-level external data source  
   - Point LOCATION to your container instead of the entire storage account  

   ```sql
   CREATE EXTERNAL DATA SOURCE MyBlobStorage
   WITH (
       TYPE = BLOB_STORAGE,
       LOCATION = 'https://<storage_account>.blob.core.windows.net/<container>',
       CREDENTIAL = ManagedIdentityCredential
   );
   ```

5 Use OPENROWSET to query the JSON file  
   - Supply only the file name, since the container is already in the LOCATION  

   ```sql
   SELECT TOP 1 BulkColumn AS JsonData
   FROM OPENROWSET(
       BULK '<myfile.json>',
       DATA_SOURCE = 'MyBlobStorage',
       SINGLE_CLOB
   ) AS j;
   ```

This returns the JSON as a single text column (JsonData). You can parse it further using SQL JSON functions if needed.

---

## Additional Notes

- **Firewall**  
  If the storage account has network restrictions, ensure your SQL resource is allowed.  
- **Propagation Delay**  
  Role assignments can take a few minutes to become active.  
- **Troubleshooting**  
  Check that the container name, file name, and URL are spelled correctly, and confirm the managed identity is enabled and assigned the correct RBAC role.  
