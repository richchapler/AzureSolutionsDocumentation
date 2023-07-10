# Data Acquisition: Python >> ADX via Service Principal / Key Vault

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d7ad1df6-3f08-4397-88b6-a4b7b67ea90d" width="1000" />

## Use Case
This documentation considers the following requirements and goals:
* "We want to connect to Data Explorer with Python"
* "We do not want to use device or user login"
* "We do not want to bake any keys or secrets into our Python code"

## Proposed Solution
* Demonstrate secure Python >> Data Explorer connectivity

## Solution Requirements
The proposed solution requires:
* [**Application Registration**](Infrastructure_ApplicationRegistration.md)
* [**Databricks**](https://learn.microsoft.com/en-us/azure/databricks/)
* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) with:
  * [StormEvents](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data) sample data
* [**Key Vault**](https://learn.microsoft.com/en-us/azure/key-vault) with [Secret](https://learn.microsoft.com/en-us/azure/key-vault/secrets)

-----

## Exercise 1: Create API
In this exercise, we will create and publish a Function App-based API that will receive a parameter, query SQL, and package a response.

### Step 1: Create Visual Studio Project

Open Visual Studio.

<img src="https://user-images.githubusercontent.com/44923999/212137484-599c9cd8-5e0e-46b1-818d-a3a008fecd5b.png" width="600" title="Snipped: January 12, 2023" />

-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference

* [Add a custom skill to an Azure Cognitive Search enrichment pipeline](https://learn.microsoft.com/en-us/azure/search/cognitive-search-custom-skill-interface)
* [Tips for AI enrichment in Azure Cognitive Search](https://learn.microsoft.com/en-us/azure/search/cognitive-search-concept-troubleshooting)
* [Debug an Azure Cognitive Search skillset in Azure portal](https://learn.microsoft.com/en-us/azure/search/cognitive-search-how-to-debug-skillset#debug-a-custom-skill-locally)
