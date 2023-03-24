# Data Enrichment: Open AI (WiP)

<img src="https://user-images.githubusercontent.com/44923999/227210296-1540091a-e156-41d9-9cfd-278246c311f1.png" width="1000" />

## Use Case
This solution considers the following requirements:

* "I want to learn about Azure OpenAI with a simple use case"
* "I want to enrich data with compelling content ready for consumption by end-users"

## Required Infrastructure
This solution requires the following resources:

* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [cluster](Infrastructure_DataExplorer_Cluster.md), [database](Infrastructure_DataExplorer_Database.md), and [sample data](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data?tabs=ingestion-wizard)
* [**OpenAI**](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/overview) with [deployment model](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/how-to/create-resource?pivots=web-portal)
* [**Postman**](https://www.postman.com/product/workspaces/) with a workspace and collection

## Proposed Solution

The Concept... enrich Data Explorer's sample dataset, "StormEvents" with cost data for similar, recent events {i.e., a similar storm event had damages that cost $X to repair}.

This solution will address requirements in three exercises:

* Exercise 1: Prepare Prompt
* Exercise 2: Create Data Flow

-----

## Exercise 1: Prepare Prompt
In this exercise, we will use Postman to test the OpenAI API and generate a prompt that can be used in relation to the Data Explorer, StormEvents data.

Navigate to your Postman workspace and collection {e.g., https://web.postman.co/workspace/My-Workspace~00000000-0000-0000-0000-000000000000}.

Click "+" to open a new tab and rename to "OpenAI_DaVinci".
Enter the following values:

Prompt | Entry
:----- | :-----
**HTTP Method** | Select "**POST**"
**Enter URL or paste text** | Modify and paste: `https://rchaplerai.openai.azure.com/openai/deployments/rchapleraimd/completions?api-version=2022-12-01`

Navigate to the "Headers" tab.

<img src="https://user-images.githubusercontent.com/44923999/227531164-07b77089-5142-4db1-8d3e-05a5b2a79381.png" width="800" title="Snipped: March 24, 2023" />

Enter the following values:

Key | Value
:----- | :-----
**api-key** | your OpenAI Key
**Content-Type** | application/json`

<img src="https://user-images.githubusercontent.com/44923999/227530522-a2adbc66-42f6-4102-ad21-acb8d5fb39fb.png" width="800" title="Snipped: March 24, 2023" />

Click "+" to open a new tab 
paste the following JSON:

```
{
    "prompt":"storm events",
    "max_tokens":1000,
    "temperature":0.10
}
```

Logic Explained:
* `prompt`... the query for which to generate a completion; our strategy will be to start with something very simple and hone in later steps
* `max_tokens`... establishes context length
* `temperature`... closer to 0 for greater accuracy and closer to 1 for more creativity


-----

## Reference

* [OpenAI Service REST API Reference](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/reference)
