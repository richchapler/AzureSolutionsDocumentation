# Data Surface: Cognitive Search, Multi-Source Index (WiP)

<img src="https://user-images.githubusercontent.com/44923999/226446292-5131f0ab-9d2c-45aa-80f2-20ddd0224458.png" width="1000" />

## Use Case
This solution considers the following requirements:

* "We have structured data {e.g., SQL Database} and unstructured data {e.g., CAD drawings} that want to search"
* "We want to have a single search experience, not one for each source"
* "We are eager to embrace Azure OpenAI when it becomes available"

## Required Infrastructure
This solution requires the following resources:

* [**Cognitive Search**](https://azure.microsoft.com/en-us/products/search) (aka "Search Service")
  * Enable **System-Assigned Managed Identity**
  * Use a Region that supports Cognitive Search {e.g., West}

* [**Cognitive Services**](https://learn.microsoft.com/en-us/azure/cognitive-services/)

* [**SQL**](https://learn.microsoft.com/en-us/azure/azure-sql) Server and Database
  * Include sample data
  * Enable "**Allow Azure services and resources to access this server**"
  * Grant Role Assignment "Reader" role for Search Service, System-Assigned Managed Identity
  * Grant access to Cognitive Search using the following T-SQL:
    ```
    CREATE USER [rchaplerss] FROM EXTERNAL PROVIDER;
    EXEC sp_addrolemember 'db_datareader', [rchaplerss];
    ```
    
* [**Storage Account**](Infrastructure_StorageAccount.md)
  * Create container named "drawings" (with sample files)
  * Grant Role Assignment "Storage Blob Data Reader" role for Search Service, System-Assigned Managed Identity

## Proposed Solution
This solution will address requirements in three exercises:

* Exercise 1: Cognitive Search + SQL Database
* Exercise 2: Cognitive Search + Blob Storage
* Exercise 3: Manually Create Multi-Source Index

-----

## Exercise 1: Cognitive Search + SQL Database
In this exercise, we will create and learn about Cognitive Search index functionality for SQL Server.

### Step 1: Import Data
Navigate to Cognitive Search, "**Overview**" and then click "**Import data**".

<img src="https://user-images.githubusercontent.com/44923999/226375829-57106809-9582-46b5-ba64-638d3348e36b.png" width="800" title="Snipped: March 20, 2023" />

Complete the "**Import Data**" >> "**Connect to your data**" form, including:

Prompt | Entry
:----- | :-----
**Data Source** | Select "**Azure SQL Database**"
**Data source name** | Enter a meaningful name aligned with your standard {e.g., SERVER-DATABASE}
**Connection string** | Click "**Choose an existing connection**", then select your SQL Database from the resulting pop-out menu
**Managed identity authentication** | Select "**System-assigned**"
**Table/View** | Enter "**SalesLT.Product**"

Click "**Next: Add cognitive skills**...".<br>
On the resulting "**Add cognitive skills**..." page, expand "**Attach Cognitive Services**".

<img src="https://user-images.githubusercontent.com/44923999/226380779-1feebb45-d656-4288-ae6b-f6e67c48a5e8.png" width="800" title="Snipped: March 20, 2023" />

Select your instance of Cognitive Services.<br>
Collapse "**Attach Cognitive Services**" and expand "**Add enrichments**".

<img src="https://user-images.githubusercontent.com/44923999/226403680-d650824d-0b63-4334-b3a9-1179890dfc44.png" width="800" title="Snipped: March 20, 2023" />

On the "**Add cognitive skills**..." tab,  select:
* "Extract people names"
* "Extract organization names"
* "Extract location names"
* "Extract key phrases"

Click "**Next: Customize target index**".

<img src="https://user-images.githubusercontent.com/44923999/226404041-aa514da5-1a5c-4edd-90c9-a034488f15be.png" width="800" title="Snipped: March 20, 2023" />

On the "**Customize target index**" tab, enter **Suggester** name "**azuresql-suggester**" and select all available options.<br>
Click "**Next: Create an indexer**".

<img src="https://user-images.githubusercontent.com/44923999/226385246-bf6f57bf-c315-4513-9920-fd4254f7c4ec.png" width="800" title="Snipped: March 20, 2023" />

On the "**Create an indexer**" tab, confirm default values and then click "**Submit**".

### Step 2: Confirm Success
Navigate to Cognitive Search, "**Overview**" and then the "**Indexers**" tab.<br>

<img src="https://user-images.githubusercontent.com/44923999/226418120-1ab6b78c-8b5d-4abf-a75f-e50a19ab6061.png" width="800" title="Snipped: March 20, 2023" />

Click on the newly-created Indexer.

<img src="https://user-images.githubusercontent.com/44923999/226404925-b1cf0637-b9db-4093-b5e7-62bdaa7d2140.png" width="800" title="Snipped: March 20, 2023" />

Confirm successful execution.<br>
Navigate to the "**Indexer Definition (JSON)**" tab.

<img src="https://user-images.githubusercontent.com/44923999/226405366-d5a3beaf-4c32-40ad-bd8c-43e3c1059a84.png" width="800" title="Snipped: March 20, 2023" />

Review the produced JSON content... we will use the example below in Exercise Three.

```
{
  "@odata.context": "https://rchaplerss.search.windows.net/$metadata#indexers/$entity",
  "@odata.etag": "\"0x8DB295E944A0232\"",
  "name": "azuresql-indexer",
  "description": "",
  "dataSourceName": "rchaplersds-rchaplersd",
  "skillsetName": "azuresql-skillset",
  "targetIndexName": "azuresql-index",
  "disabled": null,
  "schedule": null,
  "parameters": {
    "batchSize": null,
    "maxFailedItems": 0,
    "maxFailedItemsPerBatch": 0,
    "base64EncodeKeys": false,
    "configuration": {}
  },
  "fieldMappings": [],
  "outputFieldMappings": [
    {
      "sourceFieldName": "/document/ProductID/people",
      "targetFieldName": "people"
    },
    {
      "sourceFieldName": "/document/ProductID/organizations",
      "targetFieldName": "organizations"
    },
    {
      "sourceFieldName": "/document/ProductID/locations",
      "targetFieldName": "locations"
    },
    {
      "sourceFieldName": "/document/ProductID/keyphrases",
      "targetFieldName": "keyphrases"
    }
  ],
  "cache": null,
  "encryptionKey": null
}
```

Navigate to Cognitive Search, "**Overview**" and then the "**Indexes**" tab.<br>
Click on the newly-created index.

<img src="https://user-images.githubusercontent.com/44923999/226405949-7cc8aafa-4b24-44b0-90a6-2773ba0275ed.png" width="800" title="Snipped: March 20, 2023" />

Paste the following "**Query string**" value: `$top=1` and then click **Search**.<br>
Review "**Results**" content; example below:

```
{
  "@odata.context": "https://rchaplerss.search.windows.net/indexes('azuresql-index')/$metadata#docs(*)",
  "value": [
    {
      "@search.score": 1,
      "ProductID": "710",
      "Name": "Mountain Bike Socks, L",
      "ProductNumber": "SO-B909-L",
      "Color": "White",
      "StandardCost": "3.3963",
      "ListPrice": "9.5000",
      "Size": "L",
      "Weight": null,
      "ProductCategoryID": 27,
      "ProductModelID": 18,
      "SellStartDate": "2005-07-01T00:00:00Z",
      "SellEndDate": "2006-06-30T00:00:00Z",
      "DiscontinuedDate": null,
      "ThumbnailPhotoFileName": "no_image_available_small.gif",
      "rowguid": "161c035e-21b3-4e14-8e44-af508f35d80a",
      "ModifiedDate": "2008-03-11T10:01:36.827Z",
      "people": [],
      "organizations": [],
      "locations": [],
      "keyphrases": []
    }
  ]
}
```

**Congratulations... you have successfully completed this exercise**

-----

## Exercise 2: Cognitive Search + Blob Storage
In this exercise, we will create and learn about Cognitive Search index functionality for Blob Storage.

### Step 1: Import Data
Navigate to Cognitive Search, "**Overview**" and then click "**Import data**".

<img src="https://user-images.githubusercontent.com/44923999/226424973-8d3802c5-6459-47be-9651-ed78ac6c6378.png" width="800" title="Snipped: March 20, 2023" />

Complete the "**Import Data**" >> "**Connect to your data**" form, including:

Prompt | Entry
:----- | :-----
**Data Source** | Select "**Azure Blob Storage**"
**Data source name** | Enter a meaningful name aligned with your standard {e.g., ACCOUNT-CONTAINER}
**Data to extract** | Select "**Content and metadata**"
**Parsing mode** | Select "**Default**"
**Connection string** | Click "**Choose an existing connection**", then select your Storage Account and Container from the resulting menus
**Managed identity authentication** | Select "**System-assigned**"
**Container name** | Enter "**drawings**"

Click "**Next: Add cognitive skills**...".<br>
On the resulting "**Add cognitive skills**..." page, expand "**Attach Cognitive Services**".

<img src="https://user-images.githubusercontent.com/44923999/226438820-bf994ed7-09b1-4fc7-8aab-387b279bfa31.png" width="800" title="Snipped: March 20, 2023" />

Select your instance of Cognitive Services.<br>
Collapse "**Attach Cognitive Services**" and expand "**Add enrichments**".

<img src="https://user-images.githubusercontent.com/44923999/226439985-d3345185-8c71-49ab-bf2d-6b0bb81fb8d9.png" width="800" title="Snipped: March 20, 2023" />

On the "**Add cognitive skills**..." tab,  select:
* "Enable OCR and merge all text into merged_content field"
* "Extract people names"
* "Extract organization names"
* "Extract location names"
* "Extract key phrases"
* "Generate tags from images"
* "Generate captions from images"

Click "**Next: Customize target index**".

<img src="https://user-images.githubusercontent.com/44923999/226440751-f1218c0f-3f86-489b-8ff5-a2b3ef898978.png" width="800" title="Snipped: March 20, 2023" />

On the "**Customize target index**" tab, enter **Suggester** name "**azureblob-suggester**" and select all available options.<br>
Click "**Next: Create an indexer**".

<img src="https://user-images.githubusercontent.com/44923999/226441062-5c3304bb-05a0-401d-8ff4-be1ec077a028.png" width="800" title="Snipped: March 20, 2023" />

On the "**Create an indexer**" tab, confirm default values and then click "**Submit**".

### Step 2: Confirm Success
Navigate to Cognitive Search, "**Overview**" and then the "**Indexers**" tab.<br>

<img src="https://user-images.githubusercontent.com/44923999/226442015-6dd2e1db-37d8-4a62-9f69-1d27349eaa1a.png" width="800" title="Snipped: March 20, 2023" />

Click on the newly-created Indexer.

<img src="https://user-images.githubusercontent.com/44923999/226442058-459b4099-b530-43b7-8d4c-773d9a3ef8c1.png" width="800" title="Snipped: March 20, 2023" />

Confirm successful execution.<br>
Navigate to the "**Indexer Definition (JSON)**" tab.

<img src="https://user-images.githubusercontent.com/44923999/226442137-0061ef1e-8815-47a2-a495-81676d112bfb.png" width="800" title="Snipped: March 20, 2023" />

Review the produced JSON content... we will use the example below in Exercise Three.

```
{
  "@odata.context": "https://rchaplerss.search.windows.net/$metadata#indexers/$entity",
  "@odata.etag": "\"0x8DB297649341A54\"",
  "name": "azureblob-indexer",
  "description": "",
  "dataSourceName": "rchaplers-drawings",
  "skillsetName": "azureblob-skillset",
  "targetIndexName": "azureblob-index",
  "disabled": null,
  "schedule": null,
  "parameters": {
    "batchSize": null,
    "maxFailedItems": 0,
    "maxFailedItemsPerBatch": 0,
    "base64EncodeKeys": null,
    "configuration": {
      "dataToExtract": "contentAndMetadata",
      "parsingMode": "default",
      "imageAction": "generateNormalizedImages"
    }
  },
  "fieldMappings": [
    {
      "sourceFieldName": "metadata_storage_path",
      "targetFieldName": "metadata_storage_path",
      "mappingFunction": {
        "name": "base64Encode",
        "parameters": null
      }
    }
  ],
  "outputFieldMappings": [
    {
      "sourceFieldName": "/document/merged_content/people",
      "targetFieldName": "people"
    },
    {
      "sourceFieldName": "/document/merged_content/organizations",
      "targetFieldName": "organizations"
    },
    {
      "sourceFieldName": "/document/merged_content/locations",
      "targetFieldName": "locations"
    },
    {
      "sourceFieldName": "/document/merged_content/keyphrases",
      "targetFieldName": "keyphrases"
    },
    {
      "sourceFieldName": "/document/merged_content",
      "targetFieldName": "merged_content"
    },
    {
      "sourceFieldName": "/document/normalized_images/*/text",
      "targetFieldName": "text"
    },
    {
      "sourceFieldName": "/document/normalized_images/*/layoutText",
      "targetFieldName": "layoutText"
    },
    {
      "sourceFieldName": "/document/normalized_images/*/imageTags/*/name",
      "targetFieldName": "imageTags"
    },
    {
      "sourceFieldName": "/document/normalized_images/*/imageCaption",
      "targetFieldName": "imageCaption"
    }
  ],
  "cache": null,
  "encryptionKey": null
}
```

Navigate to Cognitive Search, "**Overview**" and then the "**Indexes**" tab.<br>
Click on the newly-created index.

<img src="https://user-images.githubusercontent.com/44923999/226443772-0ca61cb5-3848-4201-8a29-8cf0004c4535.png" width="800" title="Snipped: March 20, 2023" />

Paste the following "**Query string**" value: `$top=1` and then click **Search**.<br>
Review "**Results**" content; example below:

```
{
  "@odata.context": "https://rchaplerss.search.windows.net/indexes('azureblob-index')/$metadata#docs(*)",
  "value": [
    {
      "@search.score": 1,
      "content": "\n6'\n\n4'\n\n1' 2' 1'\n\n2'\n-6\n\n\"\n2'\n\n-6\n\"\n\n1'\n\n4'\n\n12\n\n12\n\n5'\n\n1' 2' 1'\n\n2'\n\n4'\n\n3'\n6\"6\"\n\n5'\n\n6'\n\nFLOOR PLAN FRONT ELEVATION SIDE ELEVATION\n\nDOG HOUSE PLANS\n\n6\"6\"\n\n\n\tSheets and Views\n\tModel\n\n\n",
      "metadata_storage_content_type": "application/pdf",
      "metadata_storage_size": 89182,
      "metadata_storage_last_modified": "2023-02-14T14:03:20Z",
      "metadata_storage_content_md5": "FzOLnd9v1JDJwH9AQwFCmA==",
      "metadata_storage_name": "Dog House Plan Sample - Model.pdf",
      "metadata_storage_path": "aHR0cHM6Ly9yY2hhcGxlcnMuYmxvYi5jb3JlLndpbmRvd3MubmV0L2RyYXdpbmdzL0RvZyUyMEhvdXNlJTIwUGxhbiUyMFNhbXBsZSUyMC0lMjBNb2RlbC5wZGY1",
      "metadata_storage_file_extension": ".pdf",
      "metadata_content_type": "application/pdf",
      "metadata_language": "de",
      "metadata_title": "Model",
      "metadata_creation_date": "2023-02-14T14:02:17Z",
      "people": [],
      "organizations": [],
      "locations": [
        "FLOOR"
      ],
      "keyphrases": [
        "FLOOR PLAN FRONT ELEVATION SIDE ELEVATION",
        "DOG HOUSE PLANS",
        "Views Model",
        "Sheets",
        "6"
      ],
      "merged_content": "\n6'\n\n4'\n\n1' 2' 1'\n\n2'\n-6\n\n\"\n2'\n\n-6\n\"\n\n1'\n\n4'\n\n12\n\n12\n\n5'\n\n1' 2' 1'\n\n2'\n\n4'\n\n3'\n6\"6\"\n\n5'\n\n6'\n\nFLOOR PLAN FRONT ELEVATION SIDE ELEVATION\n\nDOG HOUSE PLANS\n\n6\"6\"\n\n\n\tSheets and Views\n\tModel\n\n\n",
      "text": [],
      "layoutText": [],
      "imageTags": [],
      "imageCaption": []
    }
  ]
}
```

**Congratulations... you have successfully completed this exercise**

-----

## Exercise 3: Manually Create Multi-Source Index
In this exercise, we will manually create a multi-source index populated by two indexers.

### Step 1: Create Index
Navigate to Cognitive Search, "**Overview**" and then click "**+ Add index**".

<img src="https://user-images.githubusercontent.com/44923999/226464184-15effd4e-5df3-4f50-b7ca-360957356078.png" width="800" title="Snipped: March 20, 2023" />

Enter an "**Index name**" value {e.g., "**multisource-index**"} on the "**Create index**" page.<br>

Click "**Create**".

_Note: We will add fields (besides the default "id") later in this exercise._

### Step 2: Add Indexer for SQL Database
Navigate to Cognitive Search, "**Overview**" and then the "**Indexers**" tab.<br>
Click "**+ Add indexer**".

<img src="https://user-images.githubusercontent.com/44923999/226461680-5e63e584-3d8b-4219-9721-f95c394ce737.png" width="800" title="Snipped: March 20, 2023" />

Complete the "**Add indexer**" >> "**Settings**" form, including:

Prompt | Entry
:----- | :-----
**Name** | Enter a meaningful name aligned with your standard {e.g., SERVER-DATABASE-indexer}
**Index** | Select "**multisource-index**"
**Datasource** | Select "**rchaplersds-rchaplersd**"
**Skillset** | Select "**None**"

_Note: You have likely noticed that we are reusing previously created items like data source "rchaplersds-rchaplersd"... new or modified items can be used, of course, if different configuration is required._

Navigate to the "**Indexer Definition (JSON)**" tab.

<img src="https://user-images.githubusercontent.com/44923999/226469403-8a5549ca-a569-4c9a-9f38-8dc74c6e54ac.png" width="800" title="Snipped: March 20, 2023" />

Append the following JSON to the end of the default JSON content (included in the image above):
```
,
  "fieldMappings": [
    {
      "sourceFieldName": "ProductID",
      "targetFieldName": "id",
      "mappingFunction": null
    }
  ]
```

_Note: I determined the correct field in the SQL Database by looking for the key column in the SalesLT.Product table._

Click "**Save**".

<img src="https://user-images.githubusercontent.com/44923999/226469577-d1cb84cc-720c-41e9-a0b0-148be7c49e44.png" width="800" title="Snipped: March 20, 2023" />

_Note: You will see that additional JSON is added on save._

##### Confirm Success

Navigate to the newly created Indexer.

<img src="https://user-images.githubusercontent.com/44923999/226470050-40b41f4a-ed34-470a-801e-88962cb4f6de.png" width="800" title="Snipped: March 20, 2023" />

Confirm successful execution.

Navigate to the `multisource-index` Index.

<img src="https://user-images.githubusercontent.com/44923999/226470609-bfed84db-b4b3-4d44-9a8b-3499018bb99c.png" width="800" title="Snipped: March 20, 2023" />

Paste the following "**Query string**" value: `$top=1` and then click **Search**.<br>
Review "**Results**" content; example below:

```
{
  "@odata.context": "https://rchaplerss.search.windows.net/indexes('multisource-index')/$metadata#docs(*)",
  "value": [
    {
      "@search.score": 1,
      "id": "710"
    }
  ]
}
```

_Note: You will see that the results are not that interesting (yet!)._

### Step 3: Add Indexer for Blob Storage

Navigate to Cognitive Search, "**Overview**" and then the "**Indexers**" tab.<br>
Click "**+ Add indexer**".

<img src="https://user-images.githubusercontent.com/44923999/226608271-bac1d556-dc99-49d7-ba7d-0f0ddcf4e2fe.png" width="800" title="Snipped: March 21, 2023" />

Complete the "**Add indexer**" >> "**Settings**" form, including:

Prompt | Entry
:----- | :-----
**Name** | Enter a meaningful name aligned with your standard {e.g., ACCOUNT-CONTAINER-indexer}
**Index** | Select "**multisource-index**"
**Datasource** | Select "**rchaplers-drawings**"
**Skillset** | Select "**None**"

_Note: We do not have to map a field to id because this is handled automatically._

Click "**Save**".

##### Confirm Success

Navigate to the newly created Indexer.

<img src="https://user-images.githubusercontent.com/44923999/226609003-c4f9957a-fa24-416f-80a6-4d91b269b301.png" width="800" title="Snipped: March 21, 2023" />

Confirm successful execution.

Navigate to the `multisource-index` Index.

<img src="https://user-images.githubusercontent.com/44923999/226609444-47e6488d-62da-4adb-ada2-f841a795f048.png" width="800" title="Snipped: March 21, 2023" />

You will notice that the "**Documents**" count has increased by the number of documents included in your "**drawings**" container.

### Step 4: Add "name" Field

#### Modify Multi-Source Index
Navigate to Cognitive Search, "**Overview**" and then the "**Indexes**" tab.<br>
Click to open "**multisource-index**".
On the resulting page, click "**Edit JSON**".

<img src="https://user-images.githubusercontent.com/44923999/226611500-d1c0ead6-9b0c-4b2a-9d45-57bf4b3df7d9.png" width="800" title="Snipped: March 21, 2023" />

Paste the following JSON into the "fields" definition:

```
,
  {
    "name": "name",
    "type": "Edm.String",
    "searchable": true,
    "filterable": true,
    "retrievable": true,
    "sortable": true,
    "facetable": true,
    "key": false,
    "indexAnalyzer": null,
    "searchAnalyzer": null,
    "analyzer": null,
    "normalizer": null,
    "synonymMaps": []
  }
```

_Note: This addition will add a new non-key "name" field to "multisource-index" that can be searched, filtered, sorted, etc._

Click "**Save**".

<img src="https://user-images.githubusercontent.com/44923999/226612904-255222ba-61a6-41b8-81bf-2dac62eafb01.png" width="800" title="Snipped: March 21, 2023" />

Navigate to the "**Fields**" tab on the "**multisource-index**" page to confirm addition of the new field.

#### Modify SQL Database Indexer

Navigate to Cognitive Search, "**Overview**" and then the "**Indexers**" tab.<br>
Click to open the newly-created "**SERVER-DATABASE-indexer**".
Navigate to the "Indexer Definition (JSON)" tab.

<img src="https://user-images.githubusercontent.com/44923999/226615980-49de37bd-3394-4165-af16-38db025cdd9b.png" width="800" title="Snipped: March 21, 2023" />

Paste the following JSON into the "fieldMappings" definition:

```
,
  {
    "sourceFieldName": "Name",
    "targetFieldName": "name",
    "mappingFunction": null
  }
```

_Note: I determined the correct field in the SQL Database by looking for the SalesLT.Product column that best matched the newly-added "name" field in the index._

Click "**Save**" and then click "**Run**" to update the index.

<img src="https://user-images.githubusercontent.com/44923999/226617073-407e57c7-c63e-42e3-8b90-89af27853317.png" width="800" title="Snipped: March 21, 2023" />

Navigate to the "**Search explorer**" tab on the "**multisource-index**" page and confirm addition of the new field with a simple Search.

#### Modify Blob Storage Indexer

Navigate to Cognitive Search, "**Overview**" and then the "**Indexers**" tab.<br>
Click to open the newly-created "**ACCOUNT-CONTAINER-indexer**".
Navigate to the "Indexer Definition (JSON)" tab.














<img src="https://user-images.githubusercontent.com/44923999/226615980-49de37bd-3394-4165-af16-38db025cdd9b.png" width="800" title="Snipped: March 21, 2023" />

Paste the following JSON into the "fieldMappings" definition:

```
,
  {
    "sourceFieldName": "Name",
    "targetFieldName": "name",
    "mappingFunction": null
  }
```

_Note: I determined the correct field in the SQL Database by looking for the SalesLT.Product column that best matched the newly-added "name" field in the index._

Click "**Save**" and then click "**Run**" to update the index.

<img src="https://user-images.githubusercontent.com/44923999/226617073-407e57c7-c63e-42e3-8b90-89af27853317.png" width="800" title="Snipped: March 21, 2023" />

Navigate to the "**Search explorer**" tab on the "**multisource-index**" page and confirm addition of the new field with a simple Search.

-----

**Congratulations... you have successfully completed this exercise**
