# Data Enrichment: Open AI (WiP)

<img src="https://user-images.githubusercontent.com/44923999/227210296-1540091a-e156-41d9-9cfd-278246c311f1.png" width="1000" />

## Use Case
This solution considers the following requirements:

* "I want to learn about Azure OpenAI with a simple use case"
* "I want to enrich data with compelling content ready for consumption by end-users"

## Required Infrastructure
This solution requires the following resources:

* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [Cluster](Infrastructure_DataExplorer_Cluster.md), [Database](Infrastructure_DataExplorer_Database.md), and [sample data](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data?tabs=ingestion-wizard)
* [**OpenAI**](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/overview)
* [**Postman**](https://www.postman.com/product/workspaces/) with a workspace and collection

## Proposed Solution
This solution will address requirements in three exercises:

* Exercise 1: Test API
* Exercise 2: Data Flow

-----

## Exercise 1: Test API
In this exercise, we will use Postman to test the OpenAI API request and generate a prompt that can be used in relation to the Data Explorer, StormEvents data.

Navigate to your Postman workspace and collection {e.g., https://web.postman.co/workspace/My-Workspace~00000000-0000-0000-0000-000000000000}

<img src="https://user-images.githubusercontent.com/44923999/227523628-acba95f1-1cf4-416f-a7ba-f2787a3301d6.png" width="800" title="Snipped: March 23, 2023" />

Click "+" to open a new tab and paste the following JSON:

```
{
    "prompt":"Redmond weather",
    "max_tokens":1000,
    "temperature":0.90
}
```

Lorem Ispum

-----

## Reference

* [Azure Data Explorer output from Azure Stream Analytics](https://learn.microsoft.com/en-us/azure/stream-analytics/azure-database-explorer-output)
