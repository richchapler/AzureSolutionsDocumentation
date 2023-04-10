# Surface Data: Cognitive Search (via Power Apps)
_Note: Cognitive Search is also known as "Search Services" and "Azure Search"_

<img src="https://user-images.githubusercontent.com/44923999/230908250-3724404b-8aff-4ce5-9e2d-315db10c27eb.png" width="1000" />

## Use Case
This solution considers the following requirements:

* "We want to share our secured Cognitive Search index with our communit of business users"
* "The Demo App only supports QueryKey-based security"

## Required Infrastructure
This solution requires the following resources:

* [**Cognitive Search**](https://azure.microsoft.com/en-us/products/search)
* [**Power Apps**](xxx)

## Proposed Solution
This solution will address requirements in three exercises:

* Exercise 1: Lorem
* Exercise 2: Lorem

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

-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference

* [Tutorial: Query a Cognitive Search index from Power Apps](https://learn.microsoft.com/en-us/azure/search/search-howto-powerapps)
