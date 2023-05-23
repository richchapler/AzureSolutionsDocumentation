# Data Governance: Purview >> Alation Bridge (WiP)

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/40433723-dfa4-44b0-bbf3-7632d0278389" width="1000" />

## Use Case
This documentation considers the following requirements and goals:
* "Alation is our enterprise data catalog"
* "Alation lacks necessary connectors {e.g., Data Explorer} for metadata we want to include"
* "We want to get Azure resource metadata into Alation without manual effort"

## Proposed Solution
* Use Purview to gather metadata from Data Explorer
* Use Postman to prepare API requests for Purview and Alation
* Use Function / Logic App (???) to automate a metadata "bridge" from Purview to Alation

## Solution Requirements
The proposed solution requires:
* [**Alation**](https://www.alation.com/)
* [**Application Registration**](Infrastructure_ApplicationRegistration.md)
  * ...with Purview [collection role assignments](Infrastructure_Purview_CollectionRoleAssignment.md) for `Collection admins`, `Data source admins`, and `Data curators`
* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [Cluster and Database](https://learn.microsoft.com/en-us/azure/data-explorer/create-cluster-database-portal)
  * ...with [StormEvents](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data) sample data
  * ...with `Database User` permissions for the Purview System-Assigned Managed Identity
* [**Logic App**](https://learn.microsoft.com/en-us/azure/logic-apps/)
* [**Postman**](https://www.postman.com/product/workspaces/)
* [**Purview**](Infrastructure_Purview.md)

-----

## Exercise 1: Gather Metadata
In this exercise, we will register and scan Data Explorer.

### Step 1: Purview, Register Data Explorer

Open Microsoft Purview Governance Portal and navigate to "**Data map**" >> "**Data sources**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/31d804e0-dc6b-4752-b1d8-6d5329bf9a57" width="800" title="Snipped: May 18, 2023" />

Click "**Register**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2e263fd8-e4d2-45a7-a27c-5ef0fac600c2" width="800" title="Snipped: May 18, 2023" />

On the "**Register data source**" pop-out, search for and select "**Azure Data Explorer (Kusto)**", then click "**Continue**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0f6e0602-761a-4c1d-b40b-cfe1492c359b" width="800" title="Snipped: May 18, 2023" />

Complete the "**Register data source (Azure Data Explorer (Kusto))**" form, then click "**Register**".

-----

### Step 2: Purview, Scan Data Explorer

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d4f5a783-471c-4efc-862b-d80dec885a80" width="800" title="Snipped: May 18, 2023" />

On the "**Data sources**" page, click the "**New scan**" icon on your registered data source.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/f1142f3a-bc18-44a4-9392-870d64344fe1" width="800" title="Snipped: May 18, 2023" />

Complete the "**Scan**..." pop-out form, then click "**Continue**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/f7542d38-a911-4804-a513-f3f78ebd97d5" width="800" title="Snipped: May 18, 2023" />

Confirm selections on the "**Scope your scan**" pop-out form, then click "**Continue**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e8a6ea30-3ecc-4e0b-b955-f6ce6905ec38" width="800" title="Snipped: May 18, 2023" />

Confirm selection on the "**Select a scan rule set**" pop-out form, then click "**Continue**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/5626d2a3-df02-48bf-b224-c27df9e35219" width="800" title="Snipped: May 18, 2023" />

Complete the "**Set a scan trigger**" pop-out form, then click "**Continue**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/61d0a41a-ca51-4ea5-a6e0-4f1a6f2976d9" width="800" title="Snipped: May 18, 2023" />

Review selections on the "**Review your scan**" pop-out form, then click "**Scan and run**".

-----

### Step 3: Purview, Confirm Success

Navigate to the registered data source.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/64c8f4c8-f51d-4335-8784-d09e3a50ae18" width="800" title="Snipped: May 18, 2023" />

Monitor scan progress until "**Last run status**" equals "**Completed**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/ce6a4eee-ca73-4f95-86ca-e4415a455a8a" width="800" title="Snipped: May 18, 2023" />

Navigate to "**Data catalog**" >> "**Browse**" and confirm scan results on the "**Browse assets**" page.

-----

**Congratulations... you have successfully completed this exercise**

-----

## Exercise 2: Prepare Requests
In this exercise, we will prepare API Requests manually approximating the flow of data from Purview to Alation.

### Request Type 1: Authentication

#### Purview, OAuth2 Token

Navigate to Postman and create a new request.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b9e86ca9-09bc-4f47-8f3a-e83b542f98fe" width="800" title="Snipped: May 22, 2023" />

Prompt | Entry
:----- | :-----
**HTTP Method** | `POST`
**Enter URL**... | `https://login.microsoftonline.com/{TenantId}/oauth2/token`
**Authorization** >> Type | Select `No Auth`
**Body** | Select `form-data` and enter the following key-value pairs:<br>* `grant_type` :: `client_credentials`<br>* `client_id` :: `{ApplicationRegistration_ClientId}`<br>* `client_secret` :: `{ApplicationRegistration_ClientSecret}`<br>* `resource` :: `https://purview.azure.net`

Click "**Send**"

##### Expected Response
Status: `200 OK`<br>
```
{
    "token_type": "Bearer",
    "expires_in": "3599",
    "ext_expires_in": "3599",
    "expires_on": "1684446114",
    "not_before": "1684442214",
    "resource": "https://purview.azure.net",
    "access_token": "{Purview_AccessToken}"
}
```

#### Alation, Refresh Token

Navigate to Postman and create a new request.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/4edfff14-2aeb-436b-8071-d031d1dd7fbd" width="800" title="Snipped: May 22, 2023" />

Prompt | Entry
:----- | :-----
**HTTP Method** | `POST`
**Enter URL**... | `https://{Alation_InstanceName}.alationcatalog.com/integration/v1/createRefreshToken/`
**Authorization** >> Type | `No Auth`
**Body** | Select `form-data` and enter the following key-value pairs:<br>* `username` :: `{Alation_UserName}`<br>* `password` :: `{Alation_Password}`<br>* `name` :: `rt` (abbreviation for Refresh Token, but can be anything)

Click "**Send**"

##### Sample Response
Status: `201 Created`<br>
```
{
    "user_id": {Alation_UserId},
    "created_at": "2023-05-19T15:06:35.830643Z",
    "token_expires_at": "2023-07-18T15:06:35.830178Z",
    "token_status": "ACTIVE",
    "last_used_at": null,
    "name": "rt",
    "refresh_token": "{Alation_RefreshToken}"
}
```

#### Alation, API Access Token

Navigate to Postman and create a new request.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/3459a440-8e0e-4a31-b68e-5de48aec753b" width="800" title="Snipped: May 22, 2023" />

Prompt | Entry
:----- | :-----
**HTTP Method** | `POST`
**Enter URL**... | `https://{Alation_InstanceName}.alationcatalog.com/integration/v1/createAPIAccessToken/`
**Authorization** >> Type | `No Auth`
**Body** | Select `form-data` and enter the following key-value pairs:<br>* `refresh_token` :: `{Alation_RefreshToken}`<br>* `user_id` :: `{Alation_UserId}`

Click "**Send**"

##### Expected Response
Status: `201 Created`<br>
```
{
    "user_id": {user_id},
    "created_at": "2023-05-19T15:16:10.225053Z",
    "token_expires_at": "2023-05-20T15:16:10.222161Z",
    "token_status": "ACTIVE",
    "api_access_token": "{Alation_APIAccessToken}"
}
```

-----

### Request Type 2: Purview Query `azure_data_explorer_cluster` >> Alation "Virtual Data Source"

#### Purview Query `azure_data_explorer_cluster`

Navigate to Postman and create a new request.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/c580a21e-5363-4525-a1c3-1b00a786bec1" width="800" title="Snipped: May 22, 2023" />

Prompt | Entry
:----- | :-----
**HTTP Method** | `POST`
**Enter URL**... | `https://{Purview_AccountName}.purview.azure.com/catalog/api/search/query?api-version=2022-08-01-preview`
**Authorization** >> Type | `No Auth`
**Headers** | `Authorization` :: `Bearer {Purview_AccessToken}`
**Body** | `{ "filter": { "and": [ { "entityType": "azure_data_explorer_cluster" } ] } }`

Click "**Send**"

##### Expected Response
Status: `200 OK`<br>
```
{
    "@search.count": 1,
    "value": [
        {
            "updateBy": "ServiceAdmin",
            "id": "f2f00357-afcb-448f-b7c6-620960fb1421",
            "collectionId": "rchaplerp",
            "isIndexed": true,
            "qualifiedName": "https://rchaplerdec.westus3.kusto.windows.net",
            "entityType": "azure_data_explorer_cluster",
            "updateTime": 1684431659499,
            "assetType": [
                "Azure Data Explorer"
            ],
            "createBy": "ServiceAdmin",
            "createTime": 1684431659499,
            "name": "{Purview_AssetName}",
            "@search.score": 3.2775774
        }
    ],
    "@search.facets": null
}
```

-----

#### Alation, Create Data Source

Navigate to Postman and create a new request.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/04351026-fa83-4d02-9890-ed769f4bcaba" width="800" title="Snipped: May 22, 2023" />

Prompt | Entry
:----- | :-----
**HTTP Method** | `POST`
**Enter URL**... | `https://{Alation_InstanceName}.alationcatalog.com/integration/v1/datasource/`
**Authorization** >> Type | `No Auth`
**Headers** | `token` :: `{Alation_APIAccessToken}`
**Body** | Select `form-data` and enter the following key-value pairs:<br>* `dbtype` :: `customdb`<br>* `is_virtual` :: `true`<br>* `title` :: `{Purview_AssetName}`<br>* `deployment_setup_complete` :: `true`

Click "**Send**"

##### Expected Response
Status: `201 Created`

```
{
    "host": null,
    "port": null,
    "deployment_setup_complete": true,
    "db_username": null,
    "dbname": null,
    "latest_extraction_successful": false,
    "is_presto_hive": false,
    "disable_auto_lineage": false,
    "is_hive": false,
    "has_hdfs_based_qli": false,
    "otype": "data",
    "private": false,
    "has_aws_s3_based_qli": false,
    "has_previewable_qli": true,
    "obfuscate_literals": null,
    "enable_default_schema_extraction": false,
    "supports_compose": true,
    "title": "rchaplerdec.westus3",
    "profiling_tip": null,
    "uri": "",
    "unresolved_mention_fingerprint_method": 0,
    "can_data_upload": false,
    "url": "/data/31/",
    "latest_extraction_time": null,
    "exclude_additional_columns_in_qli": false,
    "metastore_type": 0,
    "has_metastore_uri": false,
    "webhdfs_username": null,
    "is_gone": false,
    "negative_filter_words": null,
    "metastore_uri": null,
    "aws_region": null,
    "deleted": false,
    "owner_ids": [
        65
    ],
    "builtin_datasource": null,
    "all_schemas": null,
    "id": {Alation_DataSourceId},
    ...
}
```

-----

### Request Type 3: Purview Query `azure_data_explorer_database` >> Alation "**Schema**"

#### Purview Query `azure_data_explorer_database`

Navigate to Postman and create a new request.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6402a8f6-d7cd-4657-8205-2e6d8d62aff7" width="800" title="Snipped: May 22, 2023" />

Prompt | Entry
:----- | :-----
**HTTP Method** | `POST`
**Enter URL**... | `https://{Purview_AccountName}.purview.azure.com/catalog/api/search/query?api-version=2022-08-01-preview`
**Authorization** >> Type | `No Auth`
**Headers** | `Authorization` :: `Bearer {Purview_AccessToken}`
**Body** | `{ "filter": { "and": [ { "entityType": "azure_data_explorer_database" } ] } }`

Click "**Send**"

##### Expected Response
Status: `200 OK`<br>
```
{
    "@search.count": 1,
    "value": [
        {
            "updateBy": "ServiceAdmin",
            "id": "88a6bfb6-58c6-4cd3-9fdf-a4b3f79b7057",
            "collectionId": "rchaplerp",
            "isIndexed": true,
            "qualifiedName": "https://rchaplerdec.westus3.kusto.windows.net/rchaplerded",
            "entityType": "azure_data_explorer_database",
            "updateTime": 1684432574026,
            "assetType": [
                "Azure Data Explorer"
            ],
            "createBy": "ServiceAdmin",
            "createTime": 1684431661266,
            "name": "{Purview_DatabaseName}",
            "@search.score": 3.496634
        }
    ],
    "@search.facets": null
}
```

-----

#### Alation, Create Schema

Navigate to Postman and create a new request.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/c43cc6a1-bdb8-434b-99c2-623f567f1785" width="800" title="Snipped: May 23, 2023" />

Prompt | Entry
:----- | :-----
**HTTP Method** | `POST`
**Enter URL**... | `https://{Alation_InstanceName}.alationcatalog.com/integration/v2/schema/`
**Params** | `ds_id` :: `{Alation_DataSourceId}`
**Authorization** >> Type | `No Auth`
**Headers** | `token` :: `{Purview_APIAccessToken}`
**Body** |  `[ { "key": "{Purview_APIAccessToken}.{Purview_DatabaseName}", "title": "{Purview_DatabaseName}" } ]`

Click "**Send**"

##### Expected Response
Status: `201 Created`

```
{
    "job_id": 9898
}
```

Open Alation to confirm that the Schema has been added to the Virtual Data Source.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0a491a41-e1e8-4d51-b631-17b9f6cd12a7" width="800" title="Snipped: May 23, 2023" />

-----

### Request Type 4: Purview Query `azure_data_explorer_table` >> Alation "Table"

#### Purview Query `azure_data_explorer_table`

Navigate to Postman and create a new request.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2f8436eb-9556-4d8c-9089-80fb3baac9ad" width="800" title="Snipped: May 23, 2023" />

Prompt | Entry
:----- | :-----
**HTTP Method** | `POST`
**Enter URL**... | `https://{Purview_AccountName}.purview.azure.com/catalog/api/search/query?api-version=2022-08-01-preview`
**Authorization** >> Type | `No Auth`
**Headers** | `Authorization` :: `Bearer {Purview_AccessToken}`
**Body** | `{ "filter": { "and": [ { "entityType": "azure_data_explorer_table" } ] } }`

Click "**Send**"

##### Expected Response
Status: `200 OK`<br>
```
{
    "@search.count": 1,
    "value": [
        {
            "objectType": "Tables",
            "updateBy": "ServiceAdmin",
            "id": "{Purview_TableId}",
            "collectionId": "rchaplerp",
            "isIndexed": true,
            "qualifiedName": "https://rchaplerdec.westus3.kusto.windows.net/rchaplerded/StormEvents",
            "entityType": "azure_data_explorer_table",
            "updateTime": 1684432561179,
            "classification": [
                "MICROSOFT.PERSONAL.GEOLOCATION",
                "MICROSOFT.GOVERNMENT.US.STATE",
                "MICROSOFT.PERSONAL.PHYSICALADDRESS"
            ],
            "assetType": [
                "Azure Data Explorer"
            ],
            "createBy": "ServiceAdmin",
            "createTime": 1684432561179,
            "name": "{Purview_TableName}",
            "@search.score": 35.591343
        }
    ],
    "@search.facets": null
}
```

-----

#### Alation, Create Table

Navigate to Postman and create a new request.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/79127e2b-b1d7-43c2-9a54-0ded5f0d4925" width="800" title="Snipped: May 23, 2023" />

Prompt | Entry
:----- | :-----
**HTTP Method** | `POST`
**Enter URL**... | `https://{Alation_InstanceName}.alationcatalog.com/integration/v2/schema/`
**Params** | `ds_id` :: `{Alation_DataSourceId}`
**Authorization** >> Type | `No Auth`
**Headers** | `token` :: `{Purview_APIAccessToken}`
**Body** |  `[ { "key": "{Purview_APIAccessToken}.{Purview_DatabaseName}.{Purview_TableName}", "title": "{Purview_TableName}" } ]`

Click "**Send**"

##### Expected Response
Status: `201 Created`

```
{
    "job_id": 9901
}
```

Open Alation to confirm that the Table has been added to the Virtual Data Source >> Schema.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2cd742ad-4a8a-4f62-8146-1c22709c14be" width="800" title="Snipped: May 23, 2023" />

-----

### Request Type 5: Purview Entity `azure_data_explorer_column` >> Alation "Column"

Navigate to Postman and create a new request.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/42071e64-336c-4bec-a585-e7bbdf13ebca" width="800" title="Snipped: May 23, 2023" />

Prompt | Entry
:----- | :-----
**HTTP Method** | `GET`
**Enter URL**... | `https://{Purview_AccountName}.purview.azure.com/catalog/api/atlas/v2/entity/guid/{Purview_TableId}`
**Authorization** >> Type | `No Auth`
**Headers** | `Authorization` :: `Bearer {Purview_AccessToken}`

Click "**Send**"

##### Expected Response
Status: `200 OK`<br>
```
{
     ...
     "entity": {
        "typeName": "azure_data_explorer_table",
        ...
        "relationshipAttributes": {
            ...
            "columns": [
                {
                    "guid": "55b852c7-59d5-495c-88bb-33f6f6f6000f",
                    "typeName": "azure_data_explorer_column",
                    "entityStatus": "ACTIVE",
                    "displayText": "{Purview_ColumnName}",
                    "relationshipType": "azure_data_explorer_table_columns",
                    "relationshipGuid": "61a8cf20-6eeb-497a-bff9-336931f46577",
                    "relationshipStatus": "ACTIVE",
                    "relationshipAttributes": {
                        "typeName": "azure_data_explorer_table_columns"
                    }
                },
                ...
}
```

-----

#### Alation, Create Column

Navigate to Postman and create a new request.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/dca27a61-cb94-4544-b9d2-29e9cd5d78d8" width="800" title="Snipped: May 23, 2023" />

Prompt | Entry
:----- | :-----
**HTTP Method** | `POST`
**Enter URL**... | `https://{Alation_InstanceName}.alationcatalog.com/integration/v2/schema/`
**Authorization** >> Type | `No Auth`
**Headers** | `token` :: `{Purview_APIAccessToken}`
**Body** |  `[ { "key": "{Purview_APIAccessToken}.{Purview_DatabaseName}.{Purview_TableName}.{Purview_ColumnName}", "title": "{Purview_ColumnName}", "column_type": "{Purview_ColumnType}" } ]`

Click "**Send**"

##### Expected Response
Status: `202 Accepted`

```
{
    "job_id": 9906
}
```

Open Alation to confirm that the Table has been added to the Virtual Data Source >> Schema.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b5961b14-0c59-448d-9522-00e4a09670d6" width="800" title="Snipped: May 23, 2023" />

-----

**Congratulations... you have successfully completed this exercise**

-----

## Exercise 3: Automate Requests
In this exercise, we will automate the bridge between Purview and Alation.

### Step 1: Create Logic App Workflow

Navigate to Logic App and create a new workflow.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d8d7eac1-e3bf-4a66-bcde-a9e512110645" width="800" title="Snipped: May 23, 2023" />

Complete the "**New workflow**" form and then click "**Create**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/909a2843-4ae6-43b8-b083-48dd0c0f2443" width="800" title="Snipped: May 23, 2023" />

Click to open the new workflow and then click "**Get started**" in the "**Edit in designer**" rectangle.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/119cd5b1-6d1b-4dd3-bcb2-3a104b152632" width="800" title="Snipped: May 23, 2023" />

Click "**Add a trigger**" and on the resulting "**Add an action**" pop-out, search for and select "**Recurrence**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/36b45f08-44d7-4caa-b682-f1d09a73808d" width="800" title="Snipped: May 23, 2023" />

Complete the "**Recurrence**" pop-out form, and then click "**Save**".

#### Initialize Variable, `Purview_BearerToken`

Click "+" to insert a step below "**Recurrence**", and then "**Add an action**" on the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6947d211-dcea-458c-aad6-de56db71d52e" width="800" title="Snipped: May 23, 2023" />

On the resulting "**Add an action**" pop-out, search for and then select "**Initialize variable**".







  <img src="https://user-images.githubusercontent.com/44923999/192591237-71d8d320-7131-4e15-b535-fff6d43c763d.png" width="800" title="Snipped: September 27, 2022" />

* Complete the resulting "**Initialize variable**" pop-out form, **Parameters** tab, including:

  | Prompt    | Entry           |
  | --------- | --------------- |
  | **Name**  | Enter "Date"    |
  | **Type**  | Select "String" |
  | **Value** | {null}          |

  _Note: Logic Apps does not have a data type for DateTime, so we use string and handle usage in expressions_

#### Initialize Variable, Scope

<img src="https://user-images.githubusercontent.com/44923999/192591752-1252451c-17e5-44e5-8e01-9163214d0672.png" width="800" title="Snipped: September 27, 2022" />

* Repeat for string variable **Scope**

#### Initialize Variable, KQL

<img src="https://user-images.githubusercontent.com/44923999/192592437-11340df0-697a-4a13-9c6d-5948d07e30a9.png" width="800" title="Snipped: September 27, 2022" />

* Repeat for string variable **KQL**

#### Parameters

* Click **Parameters** in the menu bar

* On the resulting **Parameters** pop-out form, click "**+ Create parameter**"

  <img src="https://user-images.githubusercontent.com/44923999/192592869-da6ac1f7-19fe-4cf7-901c-6c43e35fa7ed.png" width="800" title="Snipped: September 27, 2022" />

* Complete the pop-out form, including:

  | Prompt    | Entry                                                        |
  | --------- | ------------------------------------------------------------ |
  | **Name**  | Enter "Subscriptions"                                        |
  | **Type**  | Select "Array"                                               |
  | **Value** | Your SubscriptionId values in form: `[ "{Subscription1_Id}","{Subscription2_Id}" ]` |

  <img src="https://user-images.githubusercontent.com/44923999/192593500-0d272e61-1125-428b-a67a-801c0d34debd.png" width="800" title="Snipped: September 27, 2022" />

* Repeat for parameter **StartDate** 

  | Prompt    | Entry                                                        |
  | --------- | ------------------------------------------------------------ |
  | **Name**  | Enter "StartDate"                                            |
  | **Type**  | Select "String"                                              |
  | **Value** | Enter `2022-01-01` (or a date value that is meaningful for you) |

  _Note: Date values will be required in ISO-8601 formatted strings {e.g., 2022-09-15T00:00:00.0000000}; abbreviated versions {e.g., 2022-09-15} work fine_

* Repeat for parameter **EndDate**

  | Prompt    | Entry                                                        |
  | --------- | ------------------------------------------------------------ |
  | **Name**  | Enter "EndDate"                                              |
  | **Type**  | Select "String"                                              |
  | **Value** | Enter `2022-08-31` (or a date value that is meaningful for you) |

  _Note: Parameters will be alphabetized regardless of the order in which you create them_

* Click **X** to close the pop-out form and then click **Save**

### Step 3: Get Bearer Token

In this step, we will request an access token from the Client Credentials Token URL and initialize a Token variable.

* Navigate to **Designer**

#### HTTP, Get Token

* Click the **+** icon underneath "**Recurrence**"

  <img src="https://user-images.githubusercontent.com/44923999/192594246-6a59769f-cd1b-440c-95e0-d82620a5ec0e.png" width="800" title="Snipped: September 27, 2022" />

* Click "**Add a parallel branch**" on the resulting pop-up menu

  <img src="https://user-images.githubusercontent.com/44923999/192594362-2a0deb2e-0f92-47dd-a178-cf3edbf3b839.png" width="800" title="Snipped: September 27, 2022" />

* On the resulting "**Add an action**" pop-out, search for and then select "**HTTP**"

  <img src="https://user-images.githubusercontent.com/44923999/192596099-785e7bc4-a984-481d-9aa3-f715afef92b3.png" width="800" title="Snipped: September 27, 2022" />

* Complete the resulting "**HTTP**" pop-out form, **Parameters** tab, including:

  | Prompt      | Entry                                                        |
  | ----------- | ------------------------------------------------------------ |
  | **Method**  | Select "POST"                                                |
  | **URI**     | Enter `https://login.microsoftonline.com/16b3c013-d300-468d-ac64-7eda0820b6d3/oauth2/token` |
  | **Headers** | Add `content-type` :: `application/x-www-form-urlencoded`    |
  | **Body**    | Enter  `grant_type=client_credentials&client_id={Client Identifier}&client_secret={Client Secret}&resource=https://management.azure.com/` |

  _Note: If you expect processing to take longer than an hour {i.e., the lifespan of a token}, you might consider moving token handling to the nested iteration_

#### Initialize Variable, Token

* Click the **+** icon under **HTTP** and then "**Add an action**" on the resulting pop-up menu

* On the resulting "**Add an action**" pop-out, search for and then select "**Initialize variable**"

  <img src="https://user-images.githubusercontent.com/44923999/192597248-aae13dce-04a6-4498-831c-59050c9cd278.png" width="800" title="Snipped: September 27, 2022" />

* Complete the resulting "**Initialize variable**" pop-out form, **Parameters** tab, including:

  | Prompt    | Entry                                                        |
  | --------- | ------------------------------------------------------------ |
  | **Name**  | Enter "Token"                                                |
  | **Type**  | Select "String"                                              |
  | **Value** | Enter expression:<br>`concat('Bearer ',body('HTTP,_Get_Token').access_token)`<br><br>...and then click **OK** |

  _Note: The parenthetic "body" value {i.e., `body('HTTP,_Get_Token')`} may need to change depending on what you named the component_ 

* Click **Save**

#### Confirm Success

* Navigate to **Overview**, click "**Run Trigger**", and then click **Run** in the resulting dropdown menu

  <img src="https://user-images.githubusercontent.com/44923999/192598688-069b7473-6a9c-4b5e-bc37-c77f3814dc4f.png" width="800" title="Snipped: September 27, 2022" />

* Confirm that all actions succeed and click on those you would like to understand better

### Step 4: Prepare Dates Array

In this step, we will iterate through dates between StartDate and EndDate and append to the Dates array.

* Navigate to **Designer**

#### Initialize Variable, Counter

* Click the **+** icon underneath "**Recurrence**" and then "**Add a parallel branch**" on the resulting pop-up menu

* On the resulting "**Add an action**" pop-out, search for and then select "**Initialize variable**"

  <img src="https://user-images.githubusercontent.com/44923999/192600226-de668e6a-fa41-44ea-85a3-032b67d9e88c.png" width="800" title="Snipped: September 27, 2022" />

* Complete the resulting "**Initialize variable**" pop-out form, **Parameters** tab, including:

  | Prompt    | Entry            |
  | --------- | ---------------- |
  | **Name**  | Enter "Counter"  |
  | **Type**  | Select "Integer" |
  | **Value** | Enter 0          |

#### Initialize Variable, Dates

  <img src="https://user-images.githubusercontent.com/44923999/192601399-1cf57b86-47cb-4678-a2d7-d8dcf36c7592.png" width="800" title="Snipped: September 27, 2022" />

* Repeat for array variable **Dates** (with no initial value)

#### Do..Until

* Click the **+** icon and then "**Add an action**" on the resulting pop-up menu

  <img src="https://user-images.githubusercontent.com/44923999/192621146-85ff665a-e906-4b07-b78e-68a04206fefb.png" width="800" title="Snipped: September 27, 2022" />

* On the resulting "**Add an action**" pop-out, search for and then select "**Until**"

  <img src="https://user-images.githubusercontent.com/44923999/192621503-61143154-dd17-43d2-aefe-62c0c18cc0d8.png" width="800" title="Snipped: September 27, 2022" />

* Complete the resulting **Until** pop-out form, **Parameters** tab, including:

  | Prompt             | Entry                                                        |
  | ------------------ | ------------------------------------------------------------ |
  | **Choose a value** | Enter expression:<br>`addDays(parameters('StartDate'), variables('Counter'))` |
  | **Type**           | Select "is greater than"                                     |
  | **Choose a value** | Enter expression:<br>`addDays(parameters('EndDate'), 0)`     |

#### Append to Array Variable, Date

* Click the **+** icon inside the "**Do..Until**" action and then "**Add an action**" on the resulting pop-up menu

  <img src="https://user-images.githubusercontent.com/44923999/192622104-b9b829e1-115a-465b-8188-5aa8eb7ce3ca.png" width="800" title="Snipped: September 27, 2022" />

* On the resulting "**Add an action**" pop-out, search for and then select "**Append to array variable**"

  <img src="https://user-images.githubusercontent.com/44923999/192623210-b34b6282-152f-4df1-a5d6-4637a1e3ff2c.png" width="800" title="Snipped: September 27, 2022" />

* Complete the resulting "**Append to array variable**" pop-out form, **Parameters** tab, including:

  | Prompt    | Entry                                                        |
  | --------- | ------------------------------------------------------------ |
  | **Name**  | Select "Dates"                                               |
  | **Value** | Enter expression:<br>`addDays(parameters('StartDate'),variables('Counter'))` |

#### Increment Variable, Counter

* Click the **+** icon inside the "**Do..Until**" action and then "**Add an action**" on the resulting pop-up menu

  <img src="https://user-images.githubusercontent.com/44923999/192623419-4816edf7-8eba-40ed-95ec-82f956688289.png" width="800" title="Snipped: September 27, 2022" />

* On the resulting "**Add an action**" pop-out, search for and then select "**Increment variable**"

  <img src="https://user-images.githubusercontent.com/44923999/192623566-0b7f5a84-155c-4f6d-8e9f-977d37805a53.png" width="800" title="Snipped: September 27, 2022" />

* Complete the resulting "**Append to array variable**" pop-out form, **Parameters** tab, including:

  | Prompt    | Entry            |
  | --------- | ---------------- |
  | **Name**  | Select "Counter" |
  | **Value** | Enter "1"        |

* Click **Save**

#### Confirm Success

* Navigate to **Overview**, click "**Run Trigger**", and then click **Run** in the resulting dropdown menu

* Click on the new "**Running**" item in the "**Run History**" list

  <img src="https://user-images.githubusercontent.com/44923999/192623862-67e1bccb-a259-43f9-b97c-304dba148971.png" width="800" title="Snipped: September 27, 2022" />

* Confirm that all actions succeed and click on those you would like to understand better

### Step 5: Iterate through Subscriptions

In this step, we will create a "For Each" action for Subscriptions.

* Navigate to **Designer**

#### For Each, Subscription

* Click the **+** icon at the bottom of the page and then "**Add an action**" on the resulting pop-up menu

  <img src="https://user-images.githubusercontent.com/44923999/192624266-b5b3a4d7-2d5a-4a7f-9064-70634dbf7290.png" width="800" title="Snipped: September 27, 2022" />

  _Note: Dependencies from all parallel branches will be added to the new action_

* On the resulting "**Add an action**" pop-out, search for and then select "**For each**"

  <img src="https://user-images.githubusercontent.com/44923999/192624716-e3e20dc5-ec5a-4614-a160-f3e6c310018c.png" width="800" title="Snipped: September 27, 2022" />

* Complete the resulting "**For each**" pop-out form, **Parameters** tab, including:

  | Prompt                                   | Entry                  |
  | ---------------------------------------- | ---------------------- |
  | **Select an output from previous steps** | Select "Subscriptions" |

#### HTTP, Get Resource Groups

* Click the **+** icon inside the "**For Each, Subscription**" action and then "**Add an action**" on the resulting pop-up menu

* On the resulting "**Add an action**" pop-out, search for and then select "**HTTP**"

  <img src="https://user-images.githubusercontent.com/44923999/192627282-3ac16593-cdfb-4319-b721-4434907849fa.png" width="800" title="Snipped: September 27, 2022" />

* Complete the resulting "**HTTP**" pop-out form, **Parameters** tab, including:

  | Prompt      | Entry                                                        |
  | ----------- | ------------------------------------------------------------ |
  | **Method**  | Select "GET"                                                 |
  | **URI**     | Enter expression:<br>`concat('https://management.azure.com/subscriptions/',item(),'/resourcegroups?api-version=2021-04-01')` |
  | **Headers** | Add headers:<br>`Authorization` :: `variables('Token')`<br>`content-type` :: `application/json;charset=utf-8` |

  _Note: Capitalization of the header key "Authorization" appears to be important to this API_

* Click **Save**

#### Confirm Success

* Navigate to **Overview**, click "**Run Trigger**", and then click **Run** in the resulting dropdown menu

* Click on the new "**Running**" item in the "**Run History**" list

  <img src="https://user-images.githubusercontent.com/44923999/192626982-a02a11f7-9ffb-4d82-af51-24b502cbb331.png" width="800" title="Snipped: September 27, 2022" />

* Confirm that all actions succeed and click on those you would like to understand better

### Step 6: Iterate through Resource Groups

In this step, we will create a "For Each" action for Resource Groups {aka Scopes}.

* Navigate to **Designer**

#### For Each, Resource Group

* Click the **+** icon inside the "**For Each, Subscription**" action and then "**Add an action**" on the resulting pop-up menu

* On the resulting "**Add an action**" pop-out, search for and then select "**For each**"

  <img src="https://user-images.githubusercontent.com/44923999/192627843-ff8a80b0-0422-4ff1-8017-a3925cfb0608.png" width="800" title="Snipped: September 27, 2022" />

* Complete the resulting "**For each**" pop-out form, **Parameters** tab, including:

  | Prompt                                   | Entry                                                       |
  | ---------------------------------------- | ----------------------------------------------------------- |
  | **Select an output from previous steps** | Enter expression: `body('HTTP,_Get_Resource_Groups').value` |

#### Set Variable, Scope

* Click the **+** icon inside the "**For Each, Resource Group**" action and then "**Add an action**" on the resulting pop-up menu

* On the resulting "**Add an action**" pop-out, search for and then select "**Set variable**"

  <img src="https://user-images.githubusercontent.com/44923999/192628042-c247947c-cd7c-45b1-ab6b-c52e4c2a22c1.png" width="800" title="Snipped: September 27, 2022" />

* Complete the resulting "**Set variable**" pop-out form, **Parameters** tab, including:

  | Prompt    | Entry                         |
  | --------- | ----------------------------- |
  | **Name**  | Select "**Scope**"            |
  | **Value** | Enter expression: `item().id` |

* Click **Save**

#### Confirm Success

* Navigate to **Overview**, click "**Run Trigger**", and then click **Run** in the resulting dropdown menu

* Click on the new "**Running**" item in the "**Run History**" list

  <img src="https://user-images.githubusercontent.com/44923999/192628455-6d963fd8-3e08-42a1-b925-b739fbdbd4f7.png" width="800" title="Snipped: September 27, 2022" />

* Confirm that all actions succeed and click on those you would like to understand better

### Step 7: Iterate through Dates

In this step, we will nest "For Each" actions for Dates.

* Navigate to **Designer**

#### For Each, Date

* Click the **+** icon inside the "**For Each, Resource Group**" action and then "**Add an action**" on the resulting pop-up menu

* On the resulting "**Add an action**" pop-out, search for and then select "**For each**"

  <img src="https://user-images.githubusercontent.com/44923999/192628876-d9b264c4-d347-496b-ac71-2b2f9c8f1798.png" width="800" title="Snipped: September 27, 2022" />

* Complete the resulting "**For each**" pop-out form, **Parameters** tab, including:

  | Prompt                                   | Entry                                  |
  | ---------------------------------------- | -------------------------------------- |
  | **Select an output from previous steps** | Enter expression: `variables('Dates')` |

#### Set Variable, Date

* Click the **+** icon inside the "**For Each, Date**" action and then "**Add an action**" on the resulting pop-up menu

* On the resulting "**Add an action**" pop-out, search for and then select "**Set variable**"

  <img src="https://user-images.githubusercontent.com/44923999/192629234-e12c2485-e04c-4102-9dea-fdba5d50ae8c.png" width="800" title="Snipped: September 27, 2022" />

* Complete the resulting "**Set variable**" pop-out form, **Parameters** tab, including:

  | Prompt    | Entry                      |
  | --------- | -------------------------- |
  | **Name**  | Select "Date"              |
  | **Value** | Enter expression: `item()` |

* Click **Save**

#### Confirm Success

* Navigate to **Overview**, click "**Run Trigger**", and then click **Run** in the resulting dropdown menu

* Click on the new "**Running**" item in the "**Run History**" list

  <img src="https://user-images.githubusercontent.com/44923999/192630220-9f3fbbd4-b60a-4871-923d-e05cbb8f42de.png" width="800" title="Snipped: September 27, 2022" />

* Confirm that all actions succeed and click on those you would like to understand better

### Step 8: Get Cost Data

In this step, we will request and process data from the Cost Management API.

* Navigate to **Designer**

#### HTTP, Get Costs

* Click the **+** icon inside the "**For Each, Date**" action and then "**Add an action**" on the resulting pop-up menu

* On the resulting "**Add an action**" pop-out, search for and then select "**HTTP**"

  <img src="https://user-images.githubusercontent.com/44923999/192630740-f07210f9-c467-4779-bddd-62bb69af05fe.png" width="800" title="Snipped: September 27, 2022" />

* Complete the resulting "**HTTP**" pop-out form, **Parameters** tab, including:

  | Prompt      | Entry                                                        |
  | ----------- | ------------------------------------------------------------ |
  | **Method**  | Select "POST"                                                |
  | **URI**     | Enter expression:<br>`https://management.azure.com/@{variables('Scope')}/providers/Microsoft.CostManagement/query?api-version=2021-10-01` |
  | **Headers** | Add headers:<br>`authorization` :: `variables('Token')`<br>`content-type` :: `application/json;charset=utf-8` |

* Finally, paste the following in **Body**:

  ```
  {
    "dataset": {
      "aggregation": {
        "totalCost": {
          "function": "Sum",
          "name": "PreTaxCost"
        }
      },
      "granularity": "Daily",
      "grouping": [
        {
          "name": "ResourceGroupName",
          "type": "Dimension"
        },
        {
          "name": "ResourceType",
          "type": "Dimension"
        },
        {
          "name": "ResourceId",
          "type": "Dimension"
        },
        {
          "name": "ResourceLocation",
          "type": "Dimension"
        },
        {
          "name": "MeterCategory",
          "type": "Dimension"
        },
        {
          "name": "MeterSubCategory",
          "type": "Dimension"
        },
        {
          "name": "Meter",
          "type": "Dimension"
        },
        {
          "name": "ServiceName",
          "type": "Dimension"
        },
        {
          "name": "PartNumber",
          "type": "Dimension"
        },
        {
          "name": "PricingModel",
          "type": "Dimension"
        },
        {
          "name": "ChargeType",
          "type": "Dimension"
        },
        {
          "name": "ReservationName",
          "type": "Dimension"
        },
        {
          "name": "Frequency",
          "type": "Dimension"
        }
      ]
    },
    "timePeriod": {
      "from": "@{variables('Date')}",
      "to": "@{variables('Date')}"
    },
    "timeframe": "Custom",
    "type": "Usage"
  }
  ```

  _Note: Scope ResourceGroup does not allow use of **BillingPeriod** and **ServiceTier** columns_

#### Parse JSON

* Click the **+** icon inside the "**For Each, Date**" action and then "**Add an action**" on the resulting pop-up menu

* On the resulting "**Add an action**" pop-out, search for and then select "**Parse JSON**"

  <img src="https://user-images.githubusercontent.com/44923999/192637507-42df62f5-22e8-4ae0-988d-2e7db949b631.png" width="800" title="Snipped: September 27, 2022" />

* On the resulting "**Parse JSON**" pop-out form, **Parameters** tab, **Content** textbox, select dynamic content "Body" from the "**HTTP, Get Costs**" grouping and then, paste the following in **Schema**:

  ```
  {
      "properties": {
          "eTag": {},
          "id": {
              "type": "string"
          },
          "location": {},
          "name": {
              "type": "string"
          },
          "properties": {
              "properties": {
                  "columns": {
                      "items": {
                          "properties": {
                              "name": {
                                  "type": "string"
                              },
                              "type": {
                                  "type": "string"
                              }
                          },
                          "required": [
                              "name",
                              "type"
                          ],
                          "type": "object"
                      },
                      "type": "array"
                  },
                  "nextLink": {},
                  "rows": {
                      "items": {
                          "type": "array"
                      },
                      "type": "array"
                  }
              },
              "type": "object"
          },
          "sku": {},
          "type": {
              "type": "string"
          }
      },
      "type": "object"
  }
  ```

* Click **Save**

#### Confirm Success

* Navigate to **Overview**, click "**Run Trigger**", and then click **Run** in the resulting dropdown menu

* Click on the new "**Running**" item in the "**Run History**" list

  <img src="https://user-images.githubusercontent.com/44923999/192638440-104b408d-96d8-4271-b304-a94c4c5b777c.png" width="800" title="Snipped: September 27, 2022" />

* Confirm that all actions succeed and click on those you would like to understand better

### Step 9: Ingest to Data Explorer

In this step, we will send the Cost Management API response to Data Explorer using an `.ingest inline` command.

* Navigate to **Designer**

#### For Each, Response Row

* Click the **+** icon inside the "**For Each, Date**" action and below the "**Parse JSON**" action

* Then click "**Add an action**" on the resulting pop-up menu

* On the resulting "**Add an action**" pop-out, search for and then select "**For each**"

  <img src="https://user-images.githubusercontent.com/44923999/192640350-baaa1637-a1c9-43c8-9ae3-89cd66917b53.png" width="800" title="Snipped: September 27, 2022" />

* On the resulting "**For each**" pop-out form, **Parameters** tab, "**Select an output from previous steps**" textbox, select dynamic content "**rows**" from the "**Parse JSON**" grouping

#### Set Variable, KQL

* Click the **+** icon inside the "**For Each, Resource Group**" action and then "**Add an action**" on the resulting pop-up menu

* On the resulting "**Add an action**" pop-out, search for and then select "**Set variable**"

  <img src="https://user-images.githubusercontent.com/44923999/192789432-8591ac03-e7b3-496d-a7cd-588cb5ad3be2.png" width="800" title="Snipped: September 28, 2022" />

* Complete the resulting "**Set variable**" pop-out form, **Parameters** tab, including:

  | Prompt    | Entry                                                        |
  | --------- | ------------------------------------------------------------ |
  | **Name**  | Select "**KQL**"                                             |
  | **Value** | Enter expression: `concat('.ingest inline into table CostManagement <| ', replace(replace(string(item()),'[',''),']',''))` |

* Click **Save**

#### ADX Command, Ingest

* Click the **+** icon inside the "**For Each, Response Row**" action and then "**Add an action**" on the resulting pop-up menu

* On the resulting "**Add an action**" pop-out, click the **Azure** tab, search for and then select "**Run control command and render a chart**"

  <img src="https://user-images.githubusercontent.com/44923999/192791316-11fd9dec-a0bb-467d-8c08-4acd340e4ca8.png" width="800" title="Snipped: September 28, 2022" />

* Complete the resulting "**Run control command and render a chart**" pop-out form, **Parameters** tab, including:

  | Prompt              | Entry                   |
  | ------------------- | ----------------------- |
  | **Control Command** | Select variable **KQL** |
  | **Chart Type**      | Select "**Html Table**" |

  _Note: the selected "**Chart Type**" value does not matter; it is required by the Operation, but the result will not be used_

* Click **Save**

#### Confirm Success

* Navigate to **Overview**, click "**Run Trigger**", and then click **Run** in the resulting dropdown menu

* Click on the new "**Running**" item in the "**Run History**" list

  <img src="https://user-images.githubusercontent.com/44923999/192792427-f3052503-5ea6-4924-9d79-aa62bd827847.png" width="800" title="Snipped: September 28, 2022" />

* Confirm that all actions succeed and click on those you would like to understand better

### Step 10: Query Data

* Navigate to Data Explorer Database, then Query in the Data grouping of the left-hand navigation

  <img src="https://user-images.githubusercontent.com/44923999/192793220-ea2cc1b9-d35a-4255-b591-a2c2892d55ae.png" width="800" title="Snipped: September 28, 2022" />

* Run the following KQL:

  ```
  CostManagement
  | extend IngestionTime = ingestion_time()
  | sort by IngestionTime desc
  ```







-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference

* Alation
  * [Refresh & Access Token Overview](https://developer.alation.com/dev/reference/refresh-access-token-overview)
  * [Create a datasource](https://developer.alation.com/dev/reference/postdatasource)
  * [Create new schemas under a particular data source](https://developer.alation.com/dev/reference/postschemas)
* Purview
  * [Discovery - Query](https://learn.microsoft.com/en-us/rest/api/purview/catalogdataplane/discovery/query)
  * [Entity - Get By Guid](https://learn.microsoft.com/en-us/rest/api/purview/catalogdataplane/entity/get-by-guid)
