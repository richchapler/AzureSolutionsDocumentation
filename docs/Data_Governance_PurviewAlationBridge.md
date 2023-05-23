# Data Governance: Purview >> Alation Bridge (WiP)

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/40433723-dfa4-44b0-bbf3-7632d0278389" width="1000" />

## Use Case
This documentation considers the following requirements and goals:
* "Alation is our enterprise data catalog"
* "We use Azure resources (like Data Explorer) to which Alation cannot connect"
* "We want to get Azure resource metadata into Alation without manual effort"

## Proposed Solution
* Use Purview to gather metadata from Data Explorer
* Use Postman to prepare API requests for Purview and Alation
* Use Logic App to bridge metadata into Alation

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

### Request Type 2: Purview `azure_data_explorer_cluster` >> Alation "Virtual Data Source"

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

### Request Type 4: Purview `azure_data_explorer_table` >> Alation "Table"

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
            "id": "55b852c7-59d5-495c-88bb-33f6f6f60000",
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

### Request Type 5: Purview `azure_data_explorer_column` >> Alation "Column" ???

LOREM IPSUM

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
