# Data Surface: Cognitive Search, Multi-Source Index (WiP)

<img src="https://user-images.githubusercontent.com/44923999/225638762-782f272b-be53-4fd0-a798-96dd8fa801aa.png" width="1000" />

## Use Case
This solution considers the following requirements:

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
  * Grant IAC "Reader" role in place for Cognitive Search, System-Assigned Managed Identity
  * Grant access to Cognitive Search using the following T-SQL:
    ```
    CREATE USER [rchaplerss] FROM EXTERNAL PROVIDER;
    EXEC sp_addrolemember 'db_datareader', [rchaplerss];
    ```
* [**Storage Account**](Infrastructure_StorageAccount.md)

## Proposed Solution
This solution will address requirements in three exercises:

* Exercise 1: Cognitive Search + SQL Database
* Exercise 2: Cognitive Search + Blob Storage
* Exercise 3: Manually Create Composite Index, Indexer, etc.

-----

## Exercise 1: SQL Database
In this exercise, we will create and learn about Cognitive Search index functionality for SQL Server.

### Step 1: Import Data
Navigate to Cognitive Search and click "**Import data**".

<img src="https://user-images.githubusercontent.com/44923999/226375829-57106809-9582-46b5-ba64-638d3348e36b.png" width="800" title="Snipped: March 20, 2023" />

Complete the "**Import Data**" >> "**Connect to your data**" form including:

Prompt | Entry
:----- | :-----
**Data Source** | Select "**Azure SQL Database**"
**Data source name** | Enter a meaningful name aligned with your standard {e.g., SERVER-DATABASE}
**Connection string** | Click "**Choose an existing connection**", then select your SQL Database from the resulting pop-out menu
**Managed identity authentication** | Select "**System-assigned**"
**Table/View** | Enter "**SalesLT.Product**"

Click "**Next: Add cognitive skills**..." and on the resulting "**Add cognitive skills**..." page, expand "**Attach Cognitive Services**".

<img src="https://user-images.githubusercontent.com/44923999/226380779-1feebb45-d656-4288-ae6b-f6e67c48a5e8.png" width="800" title="Snipped: March 20, 2023" />

Select your instance of Cognitive Services, collapse "**Attach Cognitive Services**" and expand "**Add enrichments**".

<img src="https://user-images.githubusercontent.com/44923999/226403680-d650824d-0b63-4334-b3a9-1179890dfc44.png" width="800" title="Snipped: March 20, 2023" />

On the "**Add cognitive skills**..." tab,  select:
* "Extract people names"
* "Extract organization names"
* "Extract location names"
* "Extract key phrases"

Click "**Next: Customize target index**".

<img src="https://user-images.githubusercontent.com/44923999/226404041-aa514da5-1a5c-4edd-90c9-a034488f15be.png" width="800" title="Snipped: March 20, 2023" />

On the "**Customize target index**" tab, enter **Suggester** name "**azuresql-suggester**", select all available options, and then click "**Next: Create an indexer**".

<img src="https://user-images.githubusercontent.com/44923999/226385246-bf6f57bf-c315-4513-9920-fd4254f7c4ec.png" width="800" title="Snipped: March 20, 2023" />

On the "**Create an indexer**" tab, confirm default values and then click "**Submit**".

### Step 2: Confirm Success
Navigate to "**Overview**" in the navigation pane, then click on the "Indexers" tab.

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

Navigate to "**Overview**" in the navigation pane, then click on the "Index" tab.

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

-----

## Reference

* [Set up an indexer connection to Azure SQL using a managed identity](https://learn.microsoft.com/en-us/azure/search/search-howto-managed-identities-sql)
