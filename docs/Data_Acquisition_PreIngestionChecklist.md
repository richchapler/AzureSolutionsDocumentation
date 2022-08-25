## Data Acquisition... Pre-Ingestion Checklist

![image](https://user-images.githubusercontent.com/44923999/185980410-353cda9e-d0a8-405c-ab1c-409df61e46c4.png)

This use case considers requirement statements like:
* "We are planning to kick-off ingestion (both continuous and historical)... is there anything we should consider before we begin?"
* "We plan to use [H3: Uber's Hexagonal Hierarchical Spatial Index](https://www.uber.com/blog/h3/) for advanced analysis"

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* **Data Explorer** [Cluster](Infrastructure_DataExplorer_Cluster.md) and [Database](Infrastructure_DataExplorer_Database.md)

### Step 2: Core Transformations via **Update Policy**
In this step, we will consider transformations to raw data that are permanent and necessary. We will capture those transformation in a function and surface the resulting data to a target table.

Complete the following steps:

* Navigate to https://dataexplorer.azure.com/clusters/help/databases/Samples
* Replace the default KQL with:
  ```  
  StormEvents
  | take 25
  ```
  
  <img src="https://user-images.githubusercontent.com/44923999/186710088-4b80f89b-36da-437e-8686-48581d5ff07e.png" width="800" title="Snipped: August 25, 2022" />

#### Minimum Viable Product
Consider beginning every KQL exercise with thinning {i.e., "what columns and rows can I drop from downstream processing?"}. Because update policies process data only during ingestion {i.e., no updates after initial processing}, we want to be sure to consider every possible downstream need. If there is a chance you might need column X, better to materialize it in the more-efficient, resulting table than not.

In the following example, we limit the original set of columns with `project` and rows with `where`.

  ```  
  StormEvents
  | where not(isempty(BeginLon))
  | project EventId, EventType, StartTime, BeginLon, BeginLat
  ```  

#### Timestamp
Data Explorer is a **time-series** database, so having at least one meaningful timestamp column is expected. Besides its obvious value, this column will be very useful for partitioning.

The StormEvents data has two columns, `StartTime` and `EndTime` and both are type `datetime`. Either is sufficient for our needs; the choice of one, the other, or both should be based on solution context.

If your Timestamp column is of data type `long` (as in the case of a Unix timestamp), you should extend a new `UnixTime_datetime` column with one of various KQL functions like: [unixtime_milliseconds_todatetime()](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/unixtime-milliseconds-todatetimefunction)

#### Geospatial
Data Explorer handling of [geospatial](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/geospatial-grid-systems) data is simply awesome.

The StormEvents data has two pairs of columns, `BeginLon` :: `BeginLat` and `EndLon` :: `EndLat`.
Using these, we can extend new columns using [geo_point_to_h3cell()](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/geo-point-to-h3cell-function) **to address our use case requirements**.

### Reference
https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/updatepolicy
