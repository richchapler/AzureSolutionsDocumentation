# Data Governance: Purview >> Alation Bridge (WiP)

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/40433723-dfa4-44b0-bbf3-7632d0278389" width="1000" />

## Use Case
This documentation considers the following requirements and goals:
* "Alation is our enterprise data catalog"
* "We use Azure resources (like Data Explorer) to which Alation cannot connect"
* "We want to get Azure resource metadata into Alation without manual effort"

## Proposed Solution
* Use Purview to gather metadata from Data Explorer
* Use Postman to prepare API requests for Purview and Alation
* Use Logic App to bridge metadata into Alation

## Prerequisites
The proposed solution requires:
* [**Alation**](https://www.alation.com/)
* [**Application Registration**](Infrastructure_ApplicationRegistration.md) with Purview [collection role assignments](Infrastructure_Purview_CollectionRoleAssignment.md) for `Collection admins`, `Data source admins`, and `Data curators`
* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [Cluster and Database](https://learn.microsoft.com/en-us/azure/data-explorer/create-cluster-database-portal)
  * ...with [StormEvents](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data) sample data
  * ...with `Database User` permissions for the Purview System-Assigned Managed Identity
* [**Logic App**](https://learn.microsoft.com/en-us/azure/logic-apps/)
* [**Postman**](https://www.postman.com/product/workspaces/)
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

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/ce6a4eee-ca73-4f95-86ca-e4415a455a8a" width="800" title="Snipped: May 18, 2023" />

Navigate to "**Data catalog**" >> "**Browse**" and confirm scan results on the "**Browse assets**" page.

-----

**Congratulations... you have successfully completed this exercise**

-----

## Exercise 2: Prepare Requests
In this exercise, we will prepare API Requests manually approximating the flow of data from Purview to Alation.

### Step 1: Authentication

#### Purview

Navigate to your Postman Workspace and Collection.<br>
_{e.g., https://web.postman.co/workspace/My-Workspace~00000000-0000-0000-0000-000000000000}_

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/7fb36d57-ae03-404c-a89a-af5fa3ed8881" width="800" title="Snipped: May 18, 2023" />

Click "+" to create a new request.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/13c84344-2c86-454d-bb69-1fcda5075426" width="800" title="Snipped: May 18, 2023" />

Prompt | Entry
:----- | :-----
**HTTP Method** | Select "**POST**"
**Enter URL or paste text** | Modify and paste:<br>`https://login.microsoftonline.com/{TenantId}/oauth2/token`
**Authorization** >> Type | Select "**No Auth**"
**Body** >> raw >> Text | Modify and paste:<br>`grant_type=client_credentials&client_id={Client Identifier}&client_secret={Client Secret}& resource=https://purview.azure.net`


-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference

* Alation
  * [Create new schemas under a particular data source](https://developer.alation.com/dev/reference/postschemas)
