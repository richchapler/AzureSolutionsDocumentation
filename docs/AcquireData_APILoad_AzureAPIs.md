## Source from Azure APIs

Requirement statements might include:

* "Devices running Application X, running around the world, continuously drop files into the Data Lake"
* “We want to automatically ingest data files as they appear”

### Step 1: Prepare Resources

This solution requires the following resources:

* [Data Explorer Cluster](PrepareResources_DataExplorer_Cluster.md) (with [database](PrepareResources_DataExplorer_Database.md))
* [Data Lake](PrepareResources_DataLake.md) (with [container](PrepareResources_DataLake_Container.md) and downloaded [sample data](https://github.com/richchapler/AzureDataSolutions/wiki/Sample-Data))

### Step 2: One-Time Ingestion

Complete the following steps:

* Navigate to https://dataexplorer.azure.com/ and then click **Data** in the navigation pane
* Click **Ingest Data** on the resulting **Data Management** page

  <img src="https://user-images.githubusercontent.com/44923999/178302280-6dad3275-e252-4368-8f69-fa5b098dd1bb.png" width="800" title="Snipped: July 11, 2022" />
