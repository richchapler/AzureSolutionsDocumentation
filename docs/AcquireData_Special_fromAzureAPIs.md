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
* [Synapse](PrepareResources_Synapse.md) (with linked service and dataset for your Data Lake)
