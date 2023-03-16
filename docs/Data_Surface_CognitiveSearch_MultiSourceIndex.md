# Data Surface: Cognitive Search, Multi-Source Index (WiP)

<img src="https://user-images.githubusercontent.com/44923999/225638762-782f272b-be53-4fd0-a798-96dd8fa801aa.png" width="1000" />

## Use Case
This solution considers the following requirements:

* "We want to have a single search experience, not one for each source"

## Required Infrastructure
This solution requires the following resources:

* [**Cognitive Search**](https://azure.microsoft.com/en-us/products/search), with System-Assigned Managed Identity enabled
  * Use a Region that supports Cognitive Services {e.g., West}
* SQL Server and Database
  * "Alow Azure services and resources to access this server"
  * Grant IAC "Reader" role in place for Cognitive Search, SAMI
  * Grant access to Cognitive Search:

    ```
    CREATE USER [rchaplerss] FROM EXTERNAL PROVIDER;
    EXEC sp_addrolemember 'db_datareader', [rchaplerss];
    ```
 
## Proposed Solution
This solution will address requirements in three exercises:

* Exercise 1: Lorem Ipsum

-----

## Exercise 1: Create Indexer
In this exercise, we will Lorem Ipsum.

### Step 1: Lorem Ipsum
Navigate to "**blah**" in the "**blah**" grouping of the Blah navigation pane.

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
