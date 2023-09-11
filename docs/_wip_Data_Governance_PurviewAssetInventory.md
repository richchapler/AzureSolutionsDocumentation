# Data Governance: Purview Asset Inventory

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/710b6484-af25-473d-ab87-3252ba299bf2" width="1000" />

## Use Case
This documentation considers the following requirements and goals:
* "Purview Asset, Export as CSV functionality does not produce a full inventory; we have more than 1,000 assets"

## Proposed Solution
* Generate SQL sample data
* Catalog SQL sample data using Purview
* Use Postman to prepare PurviewAPI request logic
* Use Logic App to iteratively query Purview and write results to Azure Blob Storage

## Solution Requirements
The proposed solution requires:
* [**Application Registration**](Infrastructure_ApplicationRegistration.md)
  * ...with Purview Role Assignment "owner" (&#x1F536;QUESTION OPEN WITH SUPPORT... CAN LOWER ROLE COULD BE USED?&#x1F536;)
  * ...with Purview [Collection Role Assignments](Infrastructure_Purview_CollectionRoleAssignment.md):
    * `Collection Admin` role to access Account Data Plane and Metadata policy Data Plane
    * `Data Curator` role to access Catalog Data plane
    * `Data Source Administrator` role to access Scanning Data plane
      <br>_Note: Be patient when setting permissions for the first time... replication can take some time_
      
* Key Vault
  * ...with secret for SQL admin password
  * ...with Access Policy, Secret "Get" & "List" permissions for Purview system-assigned managed identity
  <br> _NOT CLEAR IF THIS IS REQUIRED_
* [**Logic App**](https://learn.microsoft.com/en-us/azure/logic-apps/)

* [**Postman**](https://www.postman.com/product/workspaces/)
* [**Purview**](Infrastructure_Purview.md)
  * ...with Credential for Azure SQL
* _SQL_
  * _...with both SQL and AD authentication??_
  * _Serverless, Public Endpoint, Allow Azure Services, Add Current Client IP Address, no Defender, No existing data_

-----

### Exercise 1: Generate SQL sample data
In this exercise, we will LOREM IPSUM.

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

-----

### Exercise 2: Catalog SQL sample data using Purview
In this exercise, we will LOREM IPSUM.

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

-----

### Exercise 3: Use Postman to prepare PurviewAPI request logic
In this exercise, we will LOREM IPSUM.

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

`POST https://rchaplerp.purview.azure.com/catalog/api/search/query?api-version=2022-08-01-preview`
Header: `Authorization` | `Bearer eyJ0eXAiOiJKV1QiLCJh...`
```
{
  "filter": {
    "and": [
      {
        "entityType": "azure_sql_table"
      }
    ]
  },
  "orderby": [
    {
      "name": "ASC"
    }
  ],
  "limit":3,
  "offset":1
}
```

-----

### Exercise 4: Use Logic App to iteratively query Purview and write results to Azure Blob Storage
In this exercise, we will LOREM IPSUM.

### Step 1: Create Workflow and Add Trigger

Navigate to Logic App, add and navigate to a new "**Stateful**" workflow.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/688e81f5-b1b1-4258-84ed-cb5c544a06ba" width="800" title="Snipped: Sep 11, 2023" />

Navigate to Designer and click "**Add a trigger**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/ed3da0bd-22e5-43fe-9661-4b3f1da7eea9" width="800" title="Snipped: Sep 11, 2023" />

Click "**Add a trigger**" and on the resulting "**Add a trigger**" pop-out, search for and select "**Recurrence**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/75314acf-ba05-44cf-a902-fd9e2a0b83c6" width="800" title="Snipped: Sep 11, 2023" />

Complete the "**Recurrence**" pop-out form, and then click "**Save**".

-----

### Step 2: HTTP, Bearer Token

Click "+" to insert a step below "**Recurrence**", and then "**Add an action**" on the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a8bb0eda-9e15-4bd0-8241-0d8fe916ff19" width="800" title="Snipped: Sep 11, 2023" />

On the resulting "**Add an action**" pop-out, search for and then select "**HTTP**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/9b37dc87-2a68-4735-a283-8f626c793be0" width="800" title="Snipped: Sep 11, 2023" />

  Prompt | Entry
  :----- | :-----
  **URI** | `https://login.microsoftonline.com/{TenantId}/oauth2/token`
  **Method** | `POST`
  **Headers** | `content-type` :: `application/x-www-form-urlencoded`
  **Body** | `grant_type=client_credentials&client_id={ApplicationRegistration_ClientId}&client_secret={ApplicationRegistration_ClientSecret}& resource=https://purview.azure.net`

Complete the form and click "**Save**"

-----

LOREM IPSUM!

-----

## Reference
* https://learn.microsoft.com/en-us/purview/tutorial-using-rest-apis
* https://learn.microsoft.com/en-us/rest/api/purview/catalogdataplane/discovery/query?tabs=HTTP
