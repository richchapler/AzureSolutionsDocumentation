# SQL >> Storage Connectivity: RBAC-based OPENROWSET

This documentation walks you through the steps needed to enable Azure SQL to securely query files stored in an Azure Storage account using a managed identity and RBAC. This eliminates the need for SAS tokens and leverages Azure’s native role-based access control

---

## Overview

By using a managed identity, your Azure SQL instance can authenticate directly with your Azure Storage account This configuration involves:
- Enabling a managed identity on your SQL instance
- Granting the managed identity appropriate RBAC permissions on the Storage account
- Creating a database scoped credential in SQL that references the managed identity
- Defining an external data source that points to the Storage account
- Using OPENROWSET to query data from a file (e.g. CSV or JSON)

---

## Prerequisites

Before you begin, ensure you have:

- Azure Subscription (active subscription with permissions to create and configure Azure SQL and Storage accounts)
- Azure SQL Database or Managed Instance (running SQL instance with support for external data sources)
- Azure Storage Account (contains the files you intend to query)
- Managed Identity (either system-assigned or user-assigned identity enabled on your SQL instance)
- Permissions (ability to assign RBAC roles via the Azure Portal or CLI)

---

## Step-by-step configuration

### 1. Enable managed identity on the Azure SQL instance

For Azure SQL Managed Instance  
In the Azure portal, navigate to your SQL Managed Instance and enable the system-assigned managed identity under the Identity blade

For Azure SQL Database  
If using Azure SQL Database, ensure your service supports managed identity (or consider using Azure Synapse Analytics which supports managed identities for external data sources)

### 2. Grant RBAC permissions on the storage account

Assign the role
- Go to your Azure Storage account in the Azure Portal
- Open the Access Control (IAM) section
- Click on + Add > Add role assignment
- Select the Storage Blob Data Reader role
- Under Assign access to, choose Managed Identity and select your SQL instance’s managed identity
- Click Save

This grants your SQL instance permission to read blobs from the storage account

### 3. Create a database scoped credential in Azure SQL

Connect to your Azure SQL instance using SQL Server Management Studio (SSMS) or Azure Data Studio, then run:

```sql
CREATE DATABASE SCOPED CREDENTIAL ManagedIdentityCredential
WITH IDENTITY = 'Managed Identity';
```

Note  
The keyword 'Managed Identity' signals SQL to use the managed identity of the Azure SQL instance No secret is required

### 4. Create an external data source

Define an external data source that points to your storage account URL Typically, this URL is the storage account endpoint (for example, https://<your_storage_account>.blob.core.windows.net/)

```sql
CREATE EXTERNAL DATA SOURCE MyBlobStorage
WITH (
    TYPE = BLOB_STORAGE,
    LOCATION = 'https://<your_storage_account>.blob.core.windows.net/',
    CREDENTIAL = ManagedIdentityCredential
);
```

Replace <your_storage_account> with your actual storage account name

If you want to avoid specifying the container name in your OPENROWSET path, you can point LOCATION directly to the container

```sql
CREATE EXTERNAL DATA SOURCE MyBlobStorage
WITH (
    TYPE = BLOB_STORAGE,
    LOCATION = 'https://<your_storage_account>.blob.core.windows.net/<your_container>',
    CREDENTIAL = ManagedIdentityCredential
);
```

In that case, your OPENROWSET file path only needs the file name

### 5. Use OPENROWSET to query files

#### Querying a CSV file

Use the following syntax to query a CSV file stored in a specific container

```sql
SELECT *
FROM OPENROWSET(
    BULK 'mycontainer/myfile.csv',
    DATA_SOURCE = 'MyBlobStorage',
    FORMAT = 'CSV'
) AS Data;
```

BULK specifies the path relative to the storage account endpoint  
DATA_SOURCE references the external data source created above  
FORMAT defines the file format (CSV in this case)

If you scoped your external data source to a container, remove the container name from the BULK path

#### Querying a JSON file

There isn’t a built-in JSON format Instead, load the JSON content as a single large text value (using SINGLE_CLOB) and then parse it

1. Load the JSON data

```sql
SELECT BulkColumn AS JsonData
FROM OPENROWSET(
    BULK 'mycontainer/myfile.json',
    DATA_SOURCE = 'MyBlobStorage',
    SINGLE_CLOB
) AS j;
```

2. Parse the JSON content

Use SQL Server’s JSON functions (for example, OPENJSON) to parse the JSON

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

## Troubleshooting tips

- Managed identity issues  
  Verify that the managed identity is correctly enabled on your SQL instance

- RBAC permission  
  Double-check that the identity has the Storage Blob Data Reader role on the appropriate scope (storage account or container)

- External data source URL  
  Ensure that the LOCATION in the external data source is correct If you prefer to avoid specifying the container name in the BULK path, scope the LOCATION to the container

- Network restrictions  
  Make sure your storage account’s firewall settings allow access from your SQL instance Consider enabling Allow trusted Microsoft services to access this storage account if needed

- Propagation delays  
  Role assignments and managed identity changes might take a few minutes to propagate

---

## Conclusion

By following these steps, you set up an RBAC-based connection between your Azure SQL instance and Azure Storage account This configuration uses managed identities to securely access data files via OPENROWSET without SAS tokens It enhances security and simplifies credential management by leveraging Azure’s native RBAC

If you encounter issues, revisit each step to confirm configuration details, permissions, and scope You can also consult Azure documentation for further troubleshooting assistance
