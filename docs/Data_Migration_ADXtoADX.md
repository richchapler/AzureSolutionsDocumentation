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

### Step 2: Ingest Sample Data
In this step, we will ingest sample data into our Data Explorer instance.

* Navigate to https://dataexplorer.azure.com/oneclick

  <img src="https://user-images.githubusercontent.com/44923999/195886569-1c6a3b3b-1e13-4bba-89b8-0f97fb445b52.png" width="800" title="Snipped: October 14, 2022" />

* Click "**Ingest data**"

  <img src="https://user-images.githubusercontent.com/44923999/195891239-91587c4f-5d19-49a8-b2d7-6e47b8b172b5.png" width="800" title="Snipped: October 14, 2022" />

* Complete the resulting "**Ingest data**" > "**1. Destination**" form:

  Prompt | Entry
  ------ | ------
  **Cluster** and **Database** | Select your instance
  **Table** | Select the "**New table**" radio button, and then enter the name "**StormEvents_old**" 

* Click "**Next: Source**"

  <img src="https://user-images.githubusercontent.com/44923999/195893255-ffe3d239-73cf-435e-aca0-997ba0f2779f.png" width="800" title="Snipped: October 14, 2022" />

* Complete the resulting "**Ingest data**" > "**2. Source**" form:

  Prompt | Entry
  ------ | ------
  **Source type** | Select **Blob**
  **Link to source** | Enter https://kustosamples.blob.core.windows.net/samplefiles/StormEvents.csv

* Click "**Next: Schema**"

  <img src="https://user-images.githubusercontent.com/44923999/195892772-d0759270-a6ac-4520-a053-604eb7505286.png" width="800" title="Snipped: October 14, 2022" />

* Review automatically-generated values and then click "**Next: Start ingestion**"

  <img src="https://user-images.githubusercontent.com/44923999/195893525-cbb9a491-74f6-4620-8c3c-112e0272617c.png" width="800" title="Snipped: October 14, 2022" />

* Confirm success and then click **Close**

  <img src="https://user-images.githubusercontent.com/44923999/187472753-de7b0a75-cea5-4ae0-af73-4117b65fa92d.png" width="200" title="Congratulations... you have successfuly completed this exercise!" />

### Reference
> https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data?tabs=ingestion-wizard
