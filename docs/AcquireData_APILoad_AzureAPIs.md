## Source from Azure APIs

Requirement statements might include:

Cost Management
•	“We tried using the Power BI, Cost Management connector but lack sufficient permissions at the billing account (and that is not going to change because of corporate policy)”
Log Analytics
•	“We capture logs from various device and apps to Log Analytics”
•	“We want to capture Log Analytics data and maintain an extended history”
Purview
•	“We want to use Purview to collect source metadata from Azure resources”
•	“Can we port Purview-collected metadata into our corporate-approved solution?”


### Step 1: Prepare Resources

This solution requires the following resources:

* [Data Explorer Cluster](PrepareResources_DataExplorer_Cluster.md) (with [database](PrepareResources_DataExplorer_Database.md))
* [Data Lake](PrepareResources_DataLake.md) (with [container](PrepareResources_DataLake_Container.md) and downloaded [sample data](https://github.com/richchapler/AzureDataSolutions/wiki/Sample-Data))

### Step 2: One-Time Ingestion

Complete the following steps:

* Navigate to https://dataexplorer.azure.com/ and then click **Data** in the navigation pane
* Click **Ingest Data** on the resulting **Data Management** page

  <img src="https://user-images.githubusercontent.com/44923999/178302280-6dad3275-e252-4368-8f69-fa5b098dd1bb.png" width="800" title="Snipped: July 11, 2022" />
