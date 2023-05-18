# Data Governance: Purview >> Alation Bridge (WiP)

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/40433723-dfa4-44b0-bbf3-7632d0278389" width="1000" />

## Use Case
This documentation considers the following requirements and goals:
* "Alation is our enterprise data catalog"
* "We use Azure resources (like Data Explorer) to which Alation cannot connect"
* "We want to get Azure resource metadata into Alation without manual effort"

## Proposed Solution
* Use Purview to gather metadata from Data Explorer
* Use Logic App to bridge metadata into Alation

## Prerequisites
The proposed solution requires:
* [**Alation**](https://www.alation.com/)
* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/)
* [**Logic App**](https://learn.microsoft.com/en-us/azure/logic-apps/)
* [**Purview**](Infrastructure_Purview.md)

-----

## Exercise 1: Gather Metadata
In this exercise, we will gather metadata from Data Explorer.

### Step 1: Lorem Ipsum

Open Visual Studio.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/c19a84db-d898-4ab2-9456-b168c1cd9220" width="600" title="Snipped: May 10, 2023" />

-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference

* Azure Data Explorer
  * [geo_point_to_h3cell()](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/geo-point-to-h3cell-function)
