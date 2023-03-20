# Data Surface: Cognitive Search, Multi-Source Index (WiP)

<img src="https://user-images.githubusercontent.com/44923999/225638762-782f272b-be53-4fd0-a798-96dd8fa801aa.png" width="1000" />

## Use Case
This solution considers the following requirements:

* "We want to have a single search experience, not one for each source"

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

<img src="https://user-images.githubusercontent.com/44923999/226381683-263cf8ea-8974-4181-b5c0-4ff8f3163183.png" width="800" title="Snipped: March 20, 2023" />

On the "**Add cognitive skills**..." tab,  select:
* "Extract people names"
* "Extract organization names"
* "Extract location names"
* "Extract key phrases"

_Note: We are choosing all possible options to support our learning goals {e.g., understand Cognitive Search index functionality for SQL Server and create useful reference JSON}_

Click "**Next: Customize target index**".

<img src="https://user-images.githubusercontent.com/44923999/226395418-2d3a9fd1-9f9d-4522-b0f7-ea9e4c8b6597.png" width="800" title="Snipped: March 20, 2023" />

On the "**Customize target index**" tab, enter **Suggester** name "**azuresql-suggester**" and then select all available options.

Click "**Next: Create an indexer**".

<img src="https://user-images.githubusercontent.com/44923999/226385246-bf6f57bf-c315-4513-9920-fd4254f7c4ec.png" width="800" title="Snipped: March 20, 2023" />

On the "**Create an indexer**" tab, confirm default values and then click "**Submit**".

### Step 2: Lorem Ipsum
Navigate to "**Overview**" in the navigation pane and then click into the newly-created index.

<img src="https://user-images.githubusercontent.com/44923999/221903858-42e8284b-3178-4f1f-a7c7-49f07e12568d.png" width="800" title="Snipped: February 28, 2023" />

```
{
  "@odata.context": "https://rchaplerss.search.windows.net/$metadata#indexers/$entity",
  "@odata.etag": "\"0x8DB2403A0F5F720\"",
  "name": "azureblob-indexer",
  "description": "",
  "dataSourceName": "rchaplers",
  "skillsetName": null,
  "targetIndexName": "multisource",
  "disabled": null,
  "schedule": null,
  "parameters": {
    "batchSize": null,
    "maxFailedItems": 0,
    "maxFailedItemsPerBatch": 0,
    "base64EncodeKeys": null,
    "configuration": {
      "dataToExtract": "contentAndMetadata",
      "parsingMode": "default"
    }
  },
  "fieldMappings": [
    {
      "sourceFieldName": "metadata_storage_path",
      "targetFieldName": "id",
      "mappingFunction": {
        "name": "base64Encode",
        "parameters": null
      }
    }
  ],
  "outputFieldMappings": [],
  "cache": null,
  "encryptionKey": null
}
```


-----

## Reference

* [Set up an indexer connection to Azure SQL using a managed identity](https://learn.microsoft.com/en-us/azure/search/search-howto-managed-identities-sql)
