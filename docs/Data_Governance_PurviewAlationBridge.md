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
* Use Logic App to automate a metadata "bridge" from Purview to Alation

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
**Headers** | `token` :: `{Alation_APIAccessToken}`
**Body** |  `[ { "key": "{Alation_DataSourceId}.{Purview_DatabaseName}", "title": "{Purview_DatabaseName}" } ]`

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
**Headers** | `token` :: `{Alation_APIAccessToken}`
**Body** |  `[ { "key": "{Alation_DataSourceId}.{Purview_DatabaseName}.{Purview_TableName}", "title": "{Purview_TableName}" } ]`

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
**Headers** | `token` :: `{Alation_APIAccessToken}`
**Body** |  `[ { "key": "{Alation_DataSourceId}.{Purview_DatabaseName}.{Purview_TableName}.{Purview_ColumnName}", "title": "{Purview_ColumnName}", "column_type": "{Purview_ColumnType}" } ]`

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

Navigate to Logic App and create a new "**Stateful**" workflow.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/3175f084-c54c-4b57-a59f-e304c19a53ca" width="800" title="Snipped: May 25, 2023" />

Open the new workflow and navigate to "**Designer**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/119cd5b1-6d1b-4dd3-bcb2-3a104b152632" width="800" title="Snipped: May 25, 2023" />

Click "**Add a trigger**" and on the resulting "**Add an action**" pop-out, search for and select "**Recurrence**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b8247bbb-fc36-48a7-a351-e9d4dc1b5c7d" width="800" title="Snipped: May 25, 2023" />

Complete the "**Recurrence**" pop-out form, and then click "**Save**".

-----

### Step 2: Authentication, Purview

#### HTTP POST, Purview Bearer Token

Click "+" to insert a step below "**Recurrence**", and then "**Add an action**" on the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/5a2dc9aa-fea0-4c4f-8d7e-8ebbb19813c2" width="800" title="Snipped: May 25, 2023" />

On the resulting "**Add an action**" pop-out, search for and then select "**HTTP**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/98df3a2b-0a06-451c-8789-aa06f25882c9" width="800" title="Snipped: May 25, 2023" />

Prompt | Entry
:----- | :-----
**URI** | `https://login.microsoftonline.com/{TenantId}/oauth2/token`
**Method** | `POST`
**Headers** | `content-type` :: `application/x-www-form-urlencoded`
**Body** | `grant_type=client_credentials&client_id={ApplicationRegistration_ClientId}&client_secret={ApplicationRegistration_ClientSecret}& resource=https://purview.azure.net`

Click "**Save**"

-----

#### Initialize Variable, `Purview_BearerToken`

Click "+" to insert a step below "**HTTP POST, Purview Bearer Token**", and then "**Add an action**" on the resulting menu.
<br>On the resulting "**Add an action**" pop-out, search for and then select "**Initialize Variable**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a812d391-6424-45ae-9830-f85e31c3ea35" width="800" title="Snipped: May 25, 2023" />

Prompt | Entry
:----- | :-----
**Name** | `Purview_BearerToken`
**Type** | `String`
**Value** | `concat('Bearer ',body('HTTP_POST,_Purview_Bearer_Token').access_token)`

Click "**Save**"

-----

### Step 3: Authentication, Alation

#### Initialize Variable, `Alation_InstanceName`

Click "+" to insert a step below "**Recurrence**", and then "**Add a parallel branch**" on the resulting menu.
<br>On the resulting "**Add an action**" pop-out, search for and then select "**Initialize Variable**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/9821d96c-eef2-41e9-8bae-f57545b2b940" width="800" title="Snipped: May 25, 2023" />

Prompt | Entry
:----- | :-----
**Name** | `Alation_InstanceName`
**Type** | `String`
**Value** | `{Alation_InstanceName}`

Click "**Save**"

-----

#### HTTP POST, Alation Refresh Token

Click "+" to insert a step below "**Initialize Variable, Alation_InstanceName**", and then "**Add an action**" on the resulting menu.
<br>On the resulting "**Add an action**" pop-out, search for and then select "**HTTP**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e1b11586-7b9d-4c80-a19e-c24c1ede0687" width="800" title="Snipped: May 25, 2023" />

Prompt | Entry
:----- | :-----
**URI** | `https://@{variables('Alation_InstanceName')}.alationcatalog.com/integration/v1/createRefreshToken/`
**Method** | `POST`
**Headers** | `content-type` :: `application/x-www-form-urlencoded`
**Body** | `username={username}&password={password}&name=rt`

Click "**Save**"

-----

#### HTTP POST, Alation API Access Token

Click "+" to insert a step below "**HTTP POST, Alation Refresh Token**", and then "**Add an action**" on the resulting menu.
<br>On the resulting "**Add an action**" pop-out, search for and then select "**HTTP**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/eeb371ee-de5b-45d8-9658-cc7e49fff8eb" width="800" title="Snipped: May 25, 2023" />

Prompt | Entry
:----- | :-----
**URI** | `https://@{variables('Alation_InstanceName')}.alationcatalog.com/integration/v1/createAPIAccessToken/`
**Method** | `POST`
**Headers** | `content-type` :: `application/x-www-form-urlencoded`
**Body** | `refresh_token=@{body('HTTP_POST,_Alation_Refresh_Token').refresh_token}&user_id=@{body('HTTP_POST,_Alation_Refresh_Token').user_id}`

Click "**Save**"

-----

#### Initialize Variable, `Alation_APIAccessToken`

Click "+" to insert a step below "**HTTP POST, Alation API Access Token**", and then "**Add an action**" on the resulting menu.
<br>On the resulting "**Add an action**" pop-out, search for and then select "**Initialize Variable**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6fd1c8d1-ef09-4125-9754-3caa8469ed6a" width="800" title="Snipped: May 25, 2023" />

Prompt | Entry
:----- | :-----
**Name** | `Alation_APIAccessToken`
**Type** | `String`
**Value** | `@{body('HTTP_POST,_Alation_API_Access_Token').api_access_token}`

Click "**Save**"

-----

### Step 4: Variables

#### Initialize Variable, `Database`

Click "+" to insert a step below "**Recurrence**", and then "**Add a parallel branch**" on the resulting menu.
<br>On the resulting "**Add an action**" pop-out, search for and then select "**Initialize Variable**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/394a974c-cd75-4411-b89f-43bb9ed70ca9" width="800" title="Snipped: May 26, 2023" />

Prompt | Entry
:----- | :-----
**Name** | `Database`
**Type** | `String`
**Value** | {null}

Click "**Save**"

Repeat this process for: 1) `Table` and 2 `Column`

-----

### Step 5: Iteration, Purview adxCluster >> Alation "Virtual Data Source"
_Note: Iteration steps will follow the same three-step pattern: 1) pull data from Purview, 2) setup iteration, and 3) iteratively write data to Alation_

#### Initialize Variable, `Purview_AccountName`

Click "+" to insert a step below "**Initial Variable, Purview_BearerToken**", and then "**Add an action**" on the resulting menu.
<br>On the resulting "**Add an action**" pop-out, search for and then select "**Initialize Variable**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/be90c0dd-a6e4-4d5e-a625-5eb295363238" width="800" title="Snipped: May 26, 2023" />

Prompt | Entry
:----- | :-----
**Name** | `Purview_AccountName`
**Type** | `String`
**Value** | `{Purview_AccountName}`

Click "**Save**"

-----

#### HTTP POST, Purview Query adxCluster

Click "+" to insert a step below "**HTTP POST, Purview_AccountName**", and then "**Add an action**" on the resulting menu.
<br>On the resulting "**Add an action**" pop-out, search for and then select "**HTTP**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/c122c4ad-4afa-4548-aba9-117cf75cb60e" width="800" title="Snipped: May 26, 2023" />

Prompt | Entry
:----- | :-----
**URI** | `https://.purview.azure.com/catalog/api/search/query?api-version=2022-08-01-preview`
**Method** | `POST`
**Headers** | `content-type` :: `application/json` and `authorization` :: `@{variables('Purview_BearerToken')}`
**Body** | `{ "filter": { "and": [ { "entityType": "azure_data_explorer_cluster" } ] } }`

Click "**Save**"

-----

#### For Each Cluster

Click "+" to insert a step below "**HTTP POST, Purview Query adxCluster**", and then "**Add an action**" on the resulting menu.
<br>On the resulting "**Add an action**" pop-out, search for and then select "**For each**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e9bf32c2-4a13-40a7-9ef1-907b4614ed8e" width="800" title="Snipped: May 26, 2023" />

Prompt | Entry
:----- | :-----
**Add new parameters** | Check "**Select an output**..." and then enter expression `@body('HTTP_POST,_Purview_Query_adxCluster').value`

Click "**Save**"

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/aed8e6d6-48c3-4a38-9fd9-b42536b7c6ba" width="800" title="Snipped: May 26, 2023" />

On the "**Settings**" tab, expand "Run After" and add dependencies: 1) `Initialize Variable, Column` and 2) `Initialize Variable, Alation_APIAccessToken`

-----

#### HTTP POST, Alation Create Data Source

Click "+" inside the "**For Each Cluster**" action and then "**Add an action**" on the resulting menu.
<br>On the resulting "**Add an action**" pop-out, search for and then select "**HTTP**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/fd42d466-171c-4c60-8e42-43347357d171" width="800" title="Snipped: May 26, 2023" />

Prompt | Entry
:----- | :-----
**URI** | `https://@{variables('Alation_InstanceName')}.alationcatalog.com/integration/v1/datasource/`
**Method** | `POST`
**Headers** | `token` :: `@{variables('Alation_APIAccessToken')}` and `content-type` :: `application/json`
**Body** | `{ "dbtype": "customdb", "is_virtual": "true", "title": "@{item().name}", "deployment_setup_complete": "true" }`

Click "**Save**"

-----

### Step 6: Iteration, Purview adxDatabase >> Alation "Schema"

#### HTTP POST, Purview Query adxDatabase

Click "+" to insert a step below "**HTTP POST, Alation Create Data Source**", and then "**Add an action**" on the resulting menu.
<br>On the resulting "**Add an action**" pop-out, search for and then select "**HTTP**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e6c37896-c5d3-406f-962d-d1ed2bb4c384" width="800" title="Snipped: May 26, 2023" />

Prompt | Entry
:----- | :-----
**URI** | `https://.purview.azure.com/catalog/api/search/query?api-version=2022-08-01-preview`
**Method** | `POST`
**Headers** | `content-type` :: `application/json` and `authorization` :: `@{variables('Purview_BearerToken')}`
**Body** | `{ "filter": { "and": [ { "entityType": "azure_data_explorer_database" } ] } }`

Click "**Save**"

-----

#### For Each Database

Click "+" to insert a step below "**HTTP POST, Purview Query adxDatabase**", and then "**Add an action**" on the resulting menu.
<br>On the resulting "**Add an action**" pop-out, search for and then select "**For each**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/4093b9c2-4988-4b3d-8bed-0c329eb198f9" width="800" title="Snipped: May 26, 2023" />

Prompt | Entry
:----- | :-----
**Add new parameters** | Check "**Select an output**..." and then enter expression `@body('HTTP_POST,_Purview_Query_adxDatabase').value`

Click "**Save**"

-----

#### Set Variable, Database

Click "+" inside the "**For Each Database**" action and then "**Add an action**" on the resulting menu.
<br>On the resulting "**Add an action**" pop-out, search for and then select "**Set variable**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/35af8bd0-7228-4e97-a2bf-c95efd873ca9" width="800" title="Snipped: May 26, 2023" />

Prompt | Entry
:----- | :-----
**Name** | `Database`
**Value** | `@item().name`

Click "**Save**"

-----

#### HTTP POST, Alation Create Schema

Click "+" to insert a step below "**Set Variable, Database**", and then "**Add an action**" on the resulting menu.
<br>On the resulting "**Add an action**" pop-out, search for and then select "**HTTP**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/1eb5c52c-5563-4085-8f7b-9a235bef3758" width="800" title="Snipped: May 26, 2023" />

Prompt | Entry
:----- | :-----
**URI** | `https://@{variables('Alation_InstanceName')}.alationcatalog.com/integration/v2/schema/?ds_id=@{body('HTTP_POST,_Alation_Create_Data_Source').id}`
**Method** | `POST`
**Headers** | `token` :: `@{variables('Alation_APIAccessToken')}` and `content-type` :: `application/json`
**Body** | `[ { "key": "@{body('HTTP_POST,_Alation_Create_Data_Source').id}.@{variables('DatabaseName')}", "title": "@{variables('DatabaseName')}" } ]`

Click "**Save**"

-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference

* Alation
  * [Refresh & Access Token Overview](https://developer.alation.com/dev/reference/refresh-access-token-overview)
  * [Create a datasource](https://developer.alation.com/dev/reference/postdatasource)
  * [Create new schemas under a particular data source](https://developer.alation.com/dev/reference/postschemas)
  * [Create new columns under a particular data source](https://developer.alation.com/dev/reference/postcolumns)
* Purview
  * [Discovery - Query](https://learn.microsoft.com/en-us/rest/api/purview/catalogdataplane/discovery/query)
  * [Entity - Get By Guid](https://learn.microsoft.com/en-us/rest/api/purview/catalogdataplane/entity/get-by-guid)
