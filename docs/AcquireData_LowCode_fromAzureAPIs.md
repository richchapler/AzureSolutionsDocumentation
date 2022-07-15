## Source from Azure APIs

Requirement statements might include:

  API | Statement
  :----- | :-----
  **Cost Management** | _"We tried using the Power BI, Cost Management connector but lack sufficient permissions at the billing account (and that is not going to change because of corporate policy)"_
  **Log Analytics** | _"We capture logs from various devices and apps (on-prem) to Log Analytics"_<br>_"We want to capture Log Analytics data and maintain an extended history"_
  **Purview** | _"We want to use Purview to collect source metadata from Azure resources"_<br>_"Can we port Purview-collected metadata into our corporate-approved solution?"_

### Step 1: Prepare Resources

This solution requires the following resources:

* [Application Registration](PrepareResources_ApplicationRegistration.md)
* [Data Lake](PrepareResources_DataLake.md) (with [container](PrepareResources_DataLake_Container.md))
* [Synapse](PrepareResources_Synapse.md) (with [linked services](PrepareResources_Synapse_LinkedService.md) and [datasets](PrepareResources_Synapse_Dataset.md) for your source Azure API and target Data Lake, delimited output)

Depending on the Azure API you choose, this solution might also require:
*	Application Registration + **Cost Management Reader** role assignment (granted at subscription-level)
*	Log Analytics (with on-prem agent installed and custom logs configured)
*	Purview (with collection role assignments **Collection admins**, **Data source admins**, and **Data curators** for your Application Registration)

### Step 2: Create Pipeline

* Navigate to **Synapse Studio**
* Click the **Integrate** navigation icon
* Click **+** and select **Pipeline** from the resulting dropdown menu

#### Activity 1: Get Token

This activity will make a REST API call to http://login.microsoftonline.com and get a bearer token.

  <img src="https://user-images.githubusercontent.com/44923999/179229885-810ac78b-b59c-4ce6-a2c5-6e12047011b7.png" width="800" title="Snipped: July 15, 2022" />
