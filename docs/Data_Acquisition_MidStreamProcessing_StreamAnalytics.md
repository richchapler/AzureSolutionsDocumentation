# Data Acquisition: Mid-Stream Processing via Stream Analytics

![image](https://user-images.githubusercontent.com/44923999/214475405-ae4cd7af-d318-49e7-854d-3060609c2124.png)

## Use Case
This solution considers the following requirements:

* "We used Data Explorer to capture a significant amount of rich, historical data from a soon-to-be-deprecated, on-prem database"
* "Most of the historical data is static, but a small percentage will change over time"
* "We want to help business users search this data asset"

## Required Infrastructure
This solution requires the following resources:

* [**Cognitive Search**](https://azure.microsoft.com/en-us/products/search)
* [**Cognitive Services**](https://learn.microsoft.com/en-us/azure/cognitive-services/)
* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [Cluster and Database](https://learn.microsoft.com/en-us/azure/data-explorer/create-cluster-database-portal)
* [**Storage Account**](Infrastructure_StorageAccount.md) with container "stormevents" (and related SAS token)

## Proposed Solution
This solution will address requirements in three exercises:

* Exercise 1: Data Explorer, Export Data
* Exercise 2: Cognitive Search, Import Data
* Exercise 3: Cognitive Search, Create App

-----

## Exercise 1: Data Explorer, Export Data
In this exercise, we will discuss two ways of preparing Data Explorer-based data for use by Cognitive Search:

* Option #1: One-Time Export (to Blob Storage)... for data that will not change
* Option #2: Continuous Export (to Blob Storage)... for data that will change over time

_Note: Continuous Export does not provide for data headers and that complicates the import of data into Cognitive Search_

### Step 1: Perform One-Time Export (Option #1)
In this step, we will: 1) load sample data and 2) run a KQL query to perform an export to Blob Storage.

Load sample data as specified in [Quickstart: Ingest sample data into Azure Data Explorer](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data?tabs=ingestion-wizard).

<img src="https://user-images.githubusercontent.com/44923999/214618239-312d447b-230b-4ed2-acbb-2dbf2b520ce7.png" width="800" title="Snipped: January 25, 2023" />

Confirm sample data ingestion.

<img src="https://user-images.githubusercontent.com/44923999/214860857-bad99774-9e00-4875-80a9-8f87e35cc0f2.png" width="800" title="Snipped: January 26, 2023" />

Update {i.e., replace STORAGEACCOUNT_ACCESSKEY with a real value} and then **Run** the following KQL:

```
.export to csv ( "https://rchaplers.blob.core.windows.net/stormevents;STORAGEACCOUNT_ACCESSKEY" )
with ( includeHeaders = "all" ) <| StormEvents
```

-----

## Reference

* Data Explorer
  * [Export data to storage](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/management/data-export/export-data-to-storage)
  * [Create and alter Azure Storage external tables](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/management/external-tables-azurestorage-azuredatalake)
  * [Create or alter continuous export](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/management/data-export/create-alter-continuous)
* Cognitive Search
  * [What's Azure Cognitive Search?](https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search)
  * [Import data wizard in Azure Cognitive Search](https://learn.microsoft.com/en-us/azure/search/search-import-data-portal)
  * [Quickstart: Create an Azure Cognitive Search index in the Azure portal](https://learn.microsoft.com/en-us/azure/search/search-get-started-portal)
  * [Quickstart: Use Search explorer to run queries in the portal](https://learn.microsoft.com/en-us/azure/search/search-explorer)
  * [Quickstart: Create a demo app in the portal...](https://learn.microsoft.com/en-us/azure/search/search-create-app-portal)
