# Data Enrichment: Cognitive Search, Custom "Get Data" Skillset

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/ee4be331-4ba9-47ce-a891-d146710430b7" width="1000" />

## Use Case
This documentation considers the following requirements and goals:
* "We want to enrich our Cognitive Search index with metadata from SQL"

## Proposed Solution
* Create a "Get Data" API using Function Apps
* Add a custom skillset to the Cognitive Search index

## Solution Requirements
The proposed solution requires:
* [**Cognitive Search**](https://azure.microsoft.com/en-us/products/search)
* [**Function App**](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview), including:
  * [Application Insights]
  * [Application Service Plan]
  * [Storage Account]

-----

## Exercise 1: "Get Data API"
In this exercise, we will create and publish a Function App-based API that will receive a parameter, query SQL, and package a response.

### Step 1: LOREM

Open Microsoft Purview Governance Portal and navigate to "**Data map**" >> "**Data sources**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/31d804e0-dc6b-4752-b1d8-6d5329bf9a57" width="800" title="Snipped: May 18, 2023" />

-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference

* Function App
  * LOREM
