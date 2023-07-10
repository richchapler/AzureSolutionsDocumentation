# Data Acquisition: Python >> Data Explorer, Secure Connection

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/88915f08-7bfa-4e93-86e6-1f462e3b66a4" width="1000" />

## Use Case
This documentation considers the following requirements and goals:
* "We want to connect our Python Web App to Data Explorer"
* "We do not want to use device or user login"
* "We do not want to bake keys or secrets into Python code"

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

## Exercise 1: Create Demonstration
In this exercise, we will create a Databricks Notebook with Python code blocks.

### Step 1: Prepare Resources

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/8d067703-484c-44bc-a433-6ba04d3692ee" width="800" title="Snipped: July 10, 2023" />

Open Databricks and click "**Launch Workspace**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/c3a0ed84-f7e9-4fb5-b0a4-bc0aed92fc97" width="800" title="Snipped: July 10, 2023" />

Click "**Create a notebook**".

-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference

* [Add a custom skill to an Azure Cognitive Search enrichment pipeline](https://learn.microsoft.com/en-us/azure/search/cognitive-search-custom-skill-interface)
* [Tips for AI enrichment in Azure Cognitive Search](https://learn.microsoft.com/en-us/azure/search/cognitive-search-concept-troubleshooting)
* [Debug an Azure Cognitive Search skillset in Azure portal](https://learn.microsoft.com/en-us/azure/search/cognitive-search-how-to-debug-skillset#debug-a-custom-skill-locally)
