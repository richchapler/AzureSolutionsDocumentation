# Data Governance: Purview Asset Inventory

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/710b6484-af25-473d-ab87-3252ba299bf2" width="1000" />

## Use Case
This documentation considers the following requirements and goals:
* "Purview Asset, Export as CSV functionality does not produce a full inventory; we have more than 1,000 assets"

## Proposed Solution
* Generate SQL sample data
* Catalog SQL sample data using Purview
* Use Postman to prepare PurviewAPI request logic
* Automate Process: Use an Azure Logic App to iteratively query Purview and write results to Azure Blob Storage

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

-----

### Exercise 4: Automate Process
In this exercise, we will use an Azure Logic App to iteratively query Purview and write results to Azure Blob Storage.

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

Complete the form and click "**Save**".

-----

### Step 3: Initialize Variable, `BearerToken`

Click "+" to insert a step below "**HTTP, Bearer Token**", and then "**Add an action**" on the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/07a99791-f1cc-4f6a-934b-fdbc41e80604" width="800" title="Snipped: Sep 11, 2023" />

On the resulting "**Add an action**" pop-out, search for and then select "**Initialize Variable**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/56b7f385-7a24-4227-a563-7760e0a6c1a6" width="800" title="Snipped: Sep 11, 2023" />

Prompt | Entry
:----- | :-----
**Name** | `BearerToken`
**Type** | `String`
**Value** | `concat('Bearer ',body('HTTP,_Bearer_Token').access_token)`

Complete the form and click "**Save**".

-----

### Step 4: HTTP, Query `Count`

Click "+" to insert a step below "**Initialize Variable, BearerToken**", and then "**Add an action**" on the resulting menu.
<br>On the resulting "**Add an action**" pop-out, search for and then select "**HTTP**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/f2a9fab3-3175-4e5a-96da-5dbb784e66da" width="800" title="Snipped: Sep 11, 2023" />

Prompt | Entry
:----- | :-----
**URI** | `https://{PurviewAccountName}.purview.azure.com/catalog/api/search/query?api-version=2022-08-01-preview`
**Method** | `POST`
**Headers** | `content-type` :: `application/json` and `authorization` :: `@{variables('BearerToken')}`
**Body** | `{"limit":0}`

-----

### Step 5: Initialize Variable, `AssetCount`

Click "+" to insert a step below "**HTTP, Query Count**", and then "**Add an action**" on the resulting menu.
<br>On the resulting "**Add an action**" pop-out, search for and then select "**Initialize Variable**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/bcc3f0ea-70d8-454d-a314-0916ff32c9a3" width="800" title="Snipped: Sep 11, 2023" />

Prompt | Entry
:----- | :-----
**Name** | `AssetCount`
**Type** | `Integer`
**Value** | `body('HTTP,_Query_Count')?['@search.count']`

Complete the form and click "**Save**".

-----

### Step 6: For Each Asset Batch

Click "+" to insert a step below "**Initialize Variable, AssetCount**", and then "**Add an action**" on the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/03e9bfc1-a50d-4522-acc8-b03970ec09b1" width="800" title="Snipped: Sep 11, 2023" />

On the resulting "**Add an action**" pop-out, search for and then select "**For Each**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/54a31821-8223-49ea-8877-83f620462ea8" width="800" title="Snipped: Sep 11, 2023" />

Prompt | Entry
:----- | :-----
**Select An Output From Previous Steps** | `range(0, add(div(variables('AssetCount'), 1000), 1))`

Complete the form and click "**Save**".

-----

### Step 7: HTTP, Query Assets

Click "+" inside the "**For Each Asset Batch**" action and then "**Add an action**" on the resulting menu.
<br>On the resulting "**Add an action**" pop-out, search for and then select "**HTTP**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/3429738f-9299-48b6-a974-b20f16bec90a" width="800" title="Snipped: Sep 11, 2023" />

Prompt | Entry
:----- | :-----
**URI** | `https://{PurviewAccountName}.purview.azure.com/catalog/api/search/query?api-version=2022-08-01-preview`
**Method** | `POST`
**Headers** | `content-type` :: `application/json` and `authorization` :: `@{variables('BearerToken')}`
**Body** | `{"orderby":[{"name":"ASC"}],"limit":@{string(mul(item(),1000))},"offset":1}`

Complete the form and click "**Save**".

-----

### Step 8: Create Blob

Click "+" to insert a step below "**HTTP, Query Assets**", and then "**Add an action**" on the resulting menu.
<br>On the resulting "**Add an action**" pop-out, search for and then select "**Create Blob (v2)**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/897b1262-cdfc-4ad7-8524-f28dc5aa9ded" width="800" title="Snipped: Sep 11, 2023" />

Prompt | Entry
:----- | :-----
**Connection Name** | `{StorageAccount_Name}`
**Authentication Key** | Access Key
**Azure Storage Account Name**... | `{StorageAccount_Name}`
**Azure Storage Account Access Key** | `{StorageAccount_AccessKey}`

Click "**Create New**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b368b0ab-a2c8-4530-b4bb-ba90972244cb" width="800" title="Snipped: Sep 11, 2023" />

Prompt | Entry
:----- | :-----
**Storage Account Name**... | `{StorageAccount_Name}`
**Folder Path**... | `{StorageAccount_ContainerName}`
**Blob Name**... | `@{concat(formatDateTime(utcNow(), 'yyyyMMddHHmmss'), '.json')}`
**Blob Content**... | `@{body('HTTP,_Query_Assets').value}`




Complete the form and click "**Save**".





LOREM IPSUM!

-----

## Reference
* https://learn.microsoft.com/en-us/purview/tutorial-using-rest-apis
* https://learn.microsoft.com/en-us/rest/api/purview/catalogdataplane/discovery/query?tabs=HTTP
