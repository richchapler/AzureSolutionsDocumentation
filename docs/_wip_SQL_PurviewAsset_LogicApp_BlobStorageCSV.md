Microsoft Purview >> "Export as CSV" result >> monthly pull to blob storage as CSV using Logic Apps?

Resources required:
* Azure SQL
  * ...with both SQL and AD authentication
  * Serverless, Public Endpoint, Allow Azure Services, Add Current Client IP Address, no Defender, No existing data
  * Generate 2,100 random tables to scan with Purview

    ```
    DECLARE @i INT = 1;
    WHILE @i <= 2100
    BEGIN
        DECLARE @randomName VARCHAR(255) = 'Table' + CAST(ABS(CHECKSUM(NEWID())) AS VARCHAR(255));
        DECLARE @sql NVARCHAR(MAX) = N'CREATE TABLE ' + @randomName + ' (id INT);';
        EXEC sp_executesql @sql;
        SET @i = @i + 1;
    END
    ```
* Key Vault
  * ...with secret for SQL admin password
  * ...with Access Policy, Secret "Get" and "List" permissions for Purview system-assigned managed identity
* Application Registration
  * ...with Purview Role Assignment "owner"
  * ...with Purview [Collection Role Assignments](Infrastructure_Purview_CollectionRoleAssignment.md):
    * `Collection Admin` role to access Account Data Plane and Metadata policy Data Plane
    * `Data Curator` role to access Catalog Data plane
    * `Data Source Administrator` role to access Scanning Data plane
      <br>_Note: Be patient when setting permissions for the first time... replication can take some time_
* Microsoft Purview
  * ...with Credential for Azure SQL
  * Register and Scan 2,100 random tables generated in SQL

To produce the "match this" report using the Purview UI:
* Click Data Estate Insights
* Click Assets

  ![image](https://github.com/richchapler/AzureSolutions/assets/44923999/dbf63ba0-3097-47e0-a4c4-a65600534d33)

* Click "View details"
* Select your collection on the resulting page

  ![image](https://github.com/richchapler/AzureSolutions/assets/44923999/3c3f44f7-a1cb-4d83-b941-519a7bf87e70)

* Click "Export as CSV" in the upper-right corner, then Export on the resulting pop-up

You can expect a result like:

```
"Asset ID","Asset name","Asset fully qualified name","Object type","Collection","Classification display name","Classification formal name","Asset data owner","Data owner email","Classification source","Reason for unclassified"
"d0b5cddd-501d-4509-8c01-e5923cba5a27","rchaplerdec2.westus3","https://rchaplerdec2.westus3.kusto.windows.net","Instances","rchaplerp","","","","","","Not applicable."
"11b9d5c0-7b10-4a08-9989-1e315297bd81","rchaplerded2","https://rchaplerdec2.westus3.kusto.windows.net/rchaplerded2","Databases","rchaplerp","","","","","","Not applicable."
```

Customer reports that the limit of 200 pages (* 50 items per page... i.e., 1000 items) is insufficient and wants to export the body of data to CSV.



Microsoft Purview API Request Body to search all assets:
```
{
  "keywords": null,
  "limit": 10,
  "filter": {
    "or": [
      {
        "assetType": "SQL Server"
      },
      {
        "assetType": "Azure SQL Server"
      },
      {
        "assetType": "Azure SQL Database"
      },
      {
        "assetType": "Azure SQL Data Warehouse"
      },
      {
        "assetType": "Azure SQL Managed Instance"
      },
      {
        "assetType": "Azure Storage Account"
      },
      {
        "assetType": "Azure Blob Storage"
      },
      {
        "assetType": "Azure Files"
      },
      {
        "assetType": "Azure Table Storage"
      },
      {
        "assetType": "Azure Data Lake Storage Gen1"
      },
      {
        "assetType": "Azure Data Lake Storage Gen2"
      },
      {
        "assetType": "Azure Cosmos DB"
      },
      {
        "assetType": "Azure Data Factory"
      },
      {
        "assetType": "Azure Cognitive Search"
      },
      {
        "assetType": "Power BI"
      },
      {
        "assetType": "Azure Data Explorer"
      },
      {
        "assetType": "Amazon S3"
      },
      {
        "assetType": "Azure Data Share"
      },
      {
        "assetType": "Teradata"
      },
      {
        "assetType": "SAP S4HANA"
      },
      {
        "assetType": "SAP ECC"
      },
      {
        "assetType": "SQL Server Integration Services"
      },
      {
        "assetType": "hive"
      },
      {
        "assetType": "Azure Database for MySQL"
      },
      {
        "assetType": "Azure Database for MariaDB"
      },
      {
        "assetType": "Azure Database for PostgreSQL"
      },
      {
        "assetType": "Azure Synapse Analytics"
      }
    ]
  }
}
```

## Reference
* https://learn.microsoft.com/en-us/purview/tutorial-using-rest-apis

## REST API Call
`POST https://rchaplerp.purview.azure.com/catalog/api/search/query?api-version=2022-08-01-preview`
Header: `Authorization` | `Bearer eyJ0eXAiOiJKV1QiLCJh...`
`{ "filter": { "and": [ { "entityType": "azure_sql_table" } ] } }`
