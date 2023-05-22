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

Navigate to Postman and click "+" to create a new request.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/15e5dd21-850c-42f5-8d93-c2fcfdacc7c3" width="800" title="Snipped: May 18, 2023" />

<br>Complete the form:

Prompt | Entry
:----- | :-----
**HTTP Method** | Select `POST`
**Enter URL or paste text** | Modify and paste: `https://login.microsoftonline.com/{TENANTID}/oauth2/token`
**Authorization** >> Type | Select `No Auth`
**Body** | Select `form-data` and modify/add the following key-value pairs:<br>* `grant_type` :: `client_credentials`<br>* `client_id` :: `{APPLICATIONREGISTRATION_CLIENTID}`<br>* `client_secret` :: `{APPLICATIONREGISTRATION_CLIENTSECRET}`<br>* `resource` :: `https://purview.azure.net`

Click "**Send**".

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
    "access_token": "{access_token}"
}
```

The resulting `access_token` value will be used in all subsequent Purview API requests.

#### Alation, Refresh Token

Navigate to Postman and click "+" to create a new request.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/3770cf4e-2b3d-49b0-a6ee-f6ccb544a30e" width="800" title="Snipped: May 19, 2023" />

<br>Complete the form:

Prompt | Entry
:----- | :-----
**HTTP Method** | Select `POST`
**Enter URL or paste text** | Modify and paste: `{AlationInstanceURL}/integration/v1/createRefreshToken/`
**Authorization** >> Type | Select `No Auth`
**Body** | Select `form-data` and modify/add the following key-value pairs:<br>* `username` :: `{ALATION_USERNAME}`<br>* `password` :: `{ALATION_PASSWORD}`<br>* `name` :: `rt` (abbreviation for Refresh Token, but can be anything)

Click "**Send**".

##### Sample Response
Status: `201 Created`<br>
```
{
    "user_id": {user_id},
    "created_at": "2023-05-19T15:06:35.830643Z",
    "token_expires_at": "2023-07-18T15:06:35.830178Z",
    "token_status": "ACTIVE",
    "last_used_at": null,
    "name": "rt",
    "refresh_token": "{refresh_token}"
}
```

The resulting `{user_id}` and `{refresh_token}` values will be used in the Access Token request.

#### Alation, API Access Token

Navigate to Postman and click "+" to create a new request.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/5c01dee8-4331-464e-b755-42886b34e789" width="800" title="Snipped: May 19, 2023" />

<br>Complete the form:

Prompt | Entry
:----- | :-----
**HTTP Method** | Select `POST`
**Enter URL or paste text** | Modify and paste: `{AlationInstanceURL}//integration/v1/createAPIAccessToken/`
**Authorization** >> Type | Select `No Auth`
**Body** | Select `form-data` and modify/add the following key-value pairs:<br>* `refresh_token` :: `{REFRESH_TOKEN}`<br>* `user_id` :: `{USER_ID}`

Click "**Send**".

##### Expected Response
Status: `201 Created`<br>
```
{
    "user_id": {user_id},
    "created_at": "2023-05-19T15:16:10.225053Z",
    "token_expires_at": "2023-05-20T15:16:10.222161Z",
    "token_status": "ACTIVE",
    "api_access_token": "{api_access_token}"
}
```

The resulting `{user_id}` and `{api_access_token}` values will be used in all subsequent Alation API requests.

-----

### Request Type 2: Purview `azure_data_explorer_cluster` >> Alation "Virtual Data Source"

#### Purview Query `azure_data_explorer_cluster`

Navigate to Postman and click "+" to create a new request.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/957f807e-93f6-4132-8b81-7daa28c66f9a" width="800" title="Snipped: May 19, 2023" />

<br>Complete the form:

Prompt | Entry
:----- | :-----
**HTTP Method** | Select `POST`
**Enter URL or paste text** | Modify and paste: `https://{PURVIEWACCOUNT_NAME}.purview.azure.com/catalog/api/search/query?api-version=2022-08-01-preview`
**Authorization** >> Type | Select `No Auth`
**Headers** >> Type | Modify/add `Authorization` :: `Bearer {access_token}`
**Body** | Enter `{ "filter": { "and": [ { "entityType": "azure_data_explorer_cluster" } ] } }`

Click "**Send**".

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
            "name": "rchaplerdec.westus3",
            "@search.score": 3.2775774
        }
    ],
    "@search.facets": null
}
```

The resulting `{name}` value will be used in the **Alation, Create Data Source** section.

-----

#### Alation, Create Data Source

Navigate to Postman and click "+" to create a new request.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6a334eb7-3e4f-45a8-a569-d22e4eb2af87" width="800" title="Snipped: May 22, 2023" />

<br>Complete the form:

Prompt | Entry
:----- | :-----
**HTTP Method** | `POST`
**Enter URL or paste text** | `https://{ALATIONINSTANCE_NAME}.alationcatalog.com/integration/v1/datasource/`
**Authorization** >> Type | `No Auth`
**Headers** | `token` :: "Alation, API Access Token" section >> resulting `{api_access_token}` value
**Body** | Select `form-data` and modify/add the following key-value pairs:<br>* `dbtype` :: `customdb`<br>* `is_virtual` :: `true`<br>* `title` :: "Purview Query `azure_data_explorer_cluster`" section >> resultling `{name}` value<br>* `deployment_setup_complete` :: `true`

Click "**Send**".

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
    "id": 31,
    ...
}
```

The resulting `id` value will be used in all subsequent Alation API requests.

-----

### Request Type 3: Purview Query `azure_data_explorer_database` >> Alation "Schema" ???

#### Purview `azure_data_explorer_database`

Navigate to Postman and click "+" to create a new request.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/cf61ec70-3380-47af-8e5f-6c521decbec1" width="800" title="Snipped: May 22, 2023" />

<br>Complete the form:

Prompt | Entry
:----- | :-----
**HTTP Method** | Select `POST`
**Enter URL or paste text** | Modify and paste: `https://{PURVIEWACCOUNT_NAME}.purview.azure.com/catalog/api/search/query?api-version=2022-08-01-preview`
**Authorization** >> Type | Select `No Auth`
**Headers** >> Type | Modify/add `Authorization` :: `Bearer {access_token}`
**Body** | Enter `{ "filter": { "and": [ { "entityType": "azure_data_explorer_database" } ] } }`

Click "**Send**".

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
            "name": "rchaplerded",
            "@search.score": 3.496634
        }
    ],
    "@search.facets": null
}
```

The resulting `{name}` value will be used in the **Alation, Create Schema** section.

-----

#### Alation, Create Schema

Navigate to Postman and click "+" to create a new request.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6a334eb7-3e4f-45a8-a569-d22e4eb2af87" width="800" title="Snipped: May 22, 2023" />

<br>Complete the form:

Prompt | Entry
:----- | :-----
**HTTP Method** | Select `POST`
**Enter URL or paste text** | Modify and paste: `{ALATIONINSTANCEURL}/integration/v2/schema/`
**Params** >> Type | Add the following key-value pair:<br>`ds_id` :: `LOREM`
**Authorization** >> Select `No Auth`
and enter the previously-generated `{api_access_token}` value in the "**Token**" input
**Body** | Select `form-data` and modify/add the following key-value pairs:<br>* `dbtype` :: `customdb`<br>* `is_virtual` :: `true`<br>* `title` :: `{name}` value created in the **Purview, Query** section<br>* `deployment_setup_complete` :: `true`

Click "**Send**".

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
    "id": 31,
    ...
}
```

The resulting `id` value will be used in all subsequent Alation API requests.









-----

### Request Type 4: Purview `azure_data_explorer_table` >> Alation "Table" ???

LOREM IPSUM

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
