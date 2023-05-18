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
* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [Cluster and Database](https://learn.microsoft.com/en-us/azure/data-explorer/create-cluster-database-portal)
  * ...with [StormEvents](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data) sample data
  * ...with `Database User` permissions for the Purview System-Assigned Managed Identity
* [**Logic App**](https://learn.microsoft.com/en-us/azure/logic-apps/)
* [**Purview**](Infrastructure_Purview.md)

-----

## Exercise 1: Gather Metadata
In this exercise, we will register and scan Data Explorer.

### Step 1: Purview, Register Data Explorer

Open Microsoft Purview Governance Portal and navigate to "**Data map**" >> "**Data sources**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/31d804e0-dc6b-4752-b1d8-6d5329bf9a57" width="800" title="Snipped: May 18, 2023" />

Click "**Register**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2e263fd8-e4d2-45a7-a27c-5ef0fac600c2" width="800" title="Snipped: May 18, 2023" />

On the "**Register data source**" pop-out, search for and select "**Azure Data Explorer (Kusto)**", then click "**Continue**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0f6e0602-761a-4c1d-b40b-cfe1492c359b" width="800" title="Snipped: May 18, 2023" />

Complete the "**Register data source (Azure Data Explorer (Kusto))**" form, then click "**Register**".

### Step 2: Purview, Scan Data Explorer

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d4f5a783-471c-4efc-862b-d80dec885a80" width="800" title="Snipped: May 18, 2023" />

On the "**Data sources**" page, click the "**New scan**" icon on your registered data source.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/f1142f3a-bc18-44a4-9392-870d64344fe1" width="800" title="Snipped: May 18, 2023" />

Complete the "**Scan**..." pop-out form, then click "**Continue**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/f7542d38-a911-4804-a513-f3f78ebd97d5" width="800" title="Snipped: May 18, 2023" />

Confirm selections on the "**Scope your scan**" pop-out form, then click "**Continue**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e8a6ea30-3ecc-4e0b-b955-f6ce6905ec38" width="800" title="Snipped: May 18, 2023" />

Confirm selection on the "**Select a scan rule set**" pop-out form, then click "**Continue**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/5626d2a3-df02-48bf-b224-c27df9e35219" width="800" title="Snipped: May 18, 2023" />

Complete the "**Set a scan trigger**" pop-out form, then click "**Continue**".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/61d0a41a-ca51-4ea5-a6e0-4f1a6f2976d9" width="800" title="Snipped: May 18, 2023" />

Review selections on the "**Review your scan**" pop-out form, then click "**Scan and run**".

### Step 3: Purview, Confirm Success

Navigate to the registered data source.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/64c8f4c8-f51d-4335-8784-d09e3a50ae18" width="800" title="Snipped: May 18, 2023" />

Monitor scan progress until "**Last run status**" equals "**Completed**".

-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference

* Azure Data Explorer
  * [geo_point_to_h3cell()](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/geo-point-to-h3cell-function)
