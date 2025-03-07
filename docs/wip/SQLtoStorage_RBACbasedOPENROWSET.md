# SQL >> Storage Connectivity: RBAC-based OPENROWSET

This documentation walks you through the steps needed to enable Azure SQL to securely query files stored in an Azure Storage account using a managed identity and RBAC. This eliminates the need for SAS tokens and leverages Azure’s native role-based access control.

---

## Overview

By using a managed identity, your Azure SQL instance can authenticate directly with your Azure Storage account. This configuration involves:
- Enabling a managed identity on your SQL instance.
- Granting the managed identity appropriate RBAC permissions on the Storage account.
- Creating a database scoped credential in SQL that references the managed identity.
- Defining an external data source that points to the Storage account.
- Using OPENROWSET to query data from a file (e.g., CSV or JSON).

---

## Prerequisites

Before you begin, ensure you have:

- **Azure Subscription:** Active subscription with permissions to create and configure Azure SQL and Storage accounts.
- **Azure SQL Database or Managed Instance:** Running SQL instance with support for external data sources.
- **Azure Storage Account:** Contains the files you intend to query.
- **Managed Identity:** Either system-assigned or user-assigned identity enabled on your SQL instance.
- **Permissions:** Ability to assign RBAC roles (via the Azure Portal or CLI).

---

## Step-by-Step Configuration

### 1. Enable Managed Identity on the Azure SQL Instance

- **For Azure SQL Managed Instance:**  
  In the Azure portal, navigate to your SQL Managed Instance and enable the system-assigned managed identity under the "Identity" blade.

- **For Azure SQL Database:**  
  If using Azure SQL Database in a managed instance scenario, ensure your service supports managed identity (or consider using Azure Synapse Analytics which supports managed identities for external data sources).

### 2. Grant RBAC Permissions on the Storage Account

- **Assign the Role:**
  1. Go to your Azure Storage account in the Azure Portal.
  2. Open the **Access Control (IAM)** section.
  3. Click on **+ Add > Add role assignment**.
  4. Select the **Storage Blob Data Reader** role.
  5. Under **Assign access to**, choose **Managed Identity** and select your SQL instance’s managed identity.
  6. Click **Save**.

This grants your SQL instance permission to read blobs from the storage account.

### 3. Create a Database Scoped Credential in Azure SQL

Connect to your Azure SQL instance using SQL Server Management Studio (SSMS) or Azure Data Studio, then run:

```sql
CREATE DATABASE SCOPED CREDENTIAL ManagedIdentityCredential
WITH IDENTITY = 'Managed Identity';
```

*Note:* The keyword `'Managed Identity'` signals SQL to use the managed identity of the Azure SQL instance. No secret is required.

### 4. Create an External Data Source

Define an external data source that points to your Storage account URL. Typically, this URL is the storage account endpoint (e.g., `https://<your_storage_account>.blob.core.windows.net/`).

```sql
CREATE EXTERNAL DATA SOURCE MyBlobStorage
WITH (
    TYPE = BLOB_STORAGE,
    LOCATION = 'https://<your_storage_account>.blob.core.windows.net/',
    CREDENTIAL = ManagedIdentityCredential
);
```

Replace `<your_storage_account>` with your actual storage account name.

### 5. Use OPENROWSET to Query Files

#### Querying a CSV File

Use the following syntax to query a CSV file stored in a specific container:

```sql
SELECT *
FROM OPENROWSET(
    BULK 'mycontainer/myfile.csv',
    DATA_SOURCE = 'MyBlobStorage',
    FORMAT = 'CSV'
) AS Data;
```

- **BULK:** Specify the path relative to the storage account endpoint.
- **DATA_SOURCE:** References the external data source created above.
- **FORMAT:** Defines the file format; in this case, CSV.

#### Querying a JSON File

Since there isn’t a built-in JSON format, load the JSON content as a single large text (using `SINGLE_CLOB`) and then parse it:

1. **Load the JSON Data:**

   ```sql
   SELECT BulkColumn AS JsonData
   FROM OPENROWSET(
       BULK 'mycontainer/myfile.json',
       DATA_SOURCE = 'MyBlobStorage',
       SINGLE_CLOB
   ) AS j;
   ```

2. **Parse the JSON Content:**

   You can use SQL Server’s JSON functions (e.g., `OPENJSON`) to parse the JSON:

   ```sql
   SELECT *
   FROM OPENJSON((
       SELECT BulkColumn
       FROM OPENROWSET(
            BULK 'mycontainer/myfile.json',
            DATA_SOURCE = 'MyBlobStorage',
            SINGLE_CLOB
       ) AS j
   ));
   ```

---

## Troubleshooting Tips

- **Managed Identity Issues:**  
  Verify that the managed identity is correctly enabled on your SQL instance.

- **RBAC Permission:**  
  Double-check that the identity has the **Storage Blob Data Reader** role on the appropriate scope (storage account, container, or blob).

- **External Data Source URL:**  
  Ensure that the `LOCATION` in the external data source is correct. Typically, it points to the storage account endpoint. If you want to limit the connection to a single container, you can specify that container in the URL.

- **Network Restrictions:**  
  Make sure your Storage account’s firewall settings allow access from your SQL instance. Consider enabling "Allow trusted Microsoft services to access this storage account" if needed.

- **Propagation Delays:**  
  Remember that role assignments and managed identity changes might take a few minutes to propagate.

---

## Conclusion

By following this guide, you set up an end-to-end RBAC connection between your Azure SQL instance and Azure Storage account. This configuration leverages managed identities to securely access data files via OPENROWSET without the need for SAS tokens. The process enhances security and simplifies credential management by using Azure’s native RBAC.

If you encounter issues, revisit each step to confirm configuration details and permissions, and consult the Azure documentation for further troubleshooting assistance.
