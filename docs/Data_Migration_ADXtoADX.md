## Data Migration... Data Explorer to Data Explorer

![image](https://user-images.githubusercontent.com/44923999/195881302-076c87c3-3bd0-4495-ac0a-f111730bf670.png)

This use case considers requirement statements like:
* "We have instantiated a new instance of Data Explorer and need to migrate ~1.5 billion records from the old instance"
* "We have tried `set-or-append async`, but we are hitting Data Explorer's one-hour timeout maximum"
* "To overcome the timeout, we want to chunk and iteratively copy our data by date"

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* Data Explorer, [**Cluster**](Infrastructure_DataExplorer_Cluster.md) and [**Database**](Infrastructure_DataExplorer_Database.md)
* [**Synapse**](Infrastructure_Synapse.md)

_Note: The use case requirements reference a new and an old instance of Data Explorer; for the sake of simplicy, we will use the same instance, but two separate tables_

#### Sample Data

* Navigate to https://dataexplorer.azure.com/oneclick

  <img src="https://user-images.githubusercontent.com/44923999/195886569-1c6a3b3b-1e13-4bba-89b8-0f97fb445b52.png" width="800" title="Snipped: October 14, 2022" />

* Click "**Ingest data**"

  <img src="https://user-images.githubusercontent.com/44923999/195889671-1d1975bb-cc82-487f-9440-ba526b7f02ed.png" width="800" title="Snipped: October 14, 2022" />

* On the resulting "**Ingest data**" page, "**1. Destination**" tab, select your Data Explorer Cluster and Database
* Select the "**New table**" radio button, enter the name "**StormEvents_old**" 
* Click "**Next: Source**"



https://kustosamples.blob.core.windows.net/samplefiles/StormEvents.csv

### Step 2: Prepare Workflow
In this step, we will create a workflow, initialize variables, and add parameters.

### Reference
> https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data?tabs=ingestion-wizard

  <img src="https://user-images.githubusercontent.com/44923999/187472753-de7b0a75-cea5-4ae0-af73-4117b65fa92d.png" width="200" title="Congratulations... you have successfuly completed this exercise!" />
