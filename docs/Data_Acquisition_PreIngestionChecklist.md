## Data Acquisition... Pre-Ingestion Checklist

![image](https://user-images.githubusercontent.com/44923999/185980410-353cda9e-d0a8-405c-ab1c-409df61e46c4.png)

This use case considers requirement statements like:
* "We are planning to kick-off ingestion (both continuous and historical)... is there anything we should consider before we begin?"

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* **Data Explorer** [Cluster](Infrastructure_DataExplorer_Cluster.md) and [Database](Infrastructure_DataExplorer_Database.md)

### Step 2: Core Transformations via **Update Policy**
In this step, we will consider transformations to raw data that are permanent and necessary. We will capture those transformation in a function and surface the resulting data to a target table.

Complete the following steps:

* Navigate to https://dataexplorer.azure.com/clusters/help/databases/Samples
* Replace the presented KQL with:
  ```  
  StormEvents
  | take 5
  ```
  
  <img src="https://user-images.githubusercontent.com/44923999/186710088-4b80f89b-36da-437e-8686-48581d5ff07e.png" width="800" title="Snipped: August 25, 2022" />

#### Timestamp Column
Data Explorer is a **time-series** database, so having at least one meaningful timestamp column is expected. Besides its obvious value, this column will be very useful for partitioning.

The StormEvents data has two columns, `StartTime` and `EndTime` and both are type `datetime`. This is sufficient for our needs.

If, however, they were of time `long` as in the case of a Unix timestamp, it would advanageous to extend a new `UnixTime_long` column
  
### Reference
https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/updatepolicy
