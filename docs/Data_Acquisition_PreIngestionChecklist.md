## Data Acquisition... Pre-Ingestion Checklist

![image](https://user-images.githubusercontent.com/44923999/185980410-353cda9e-d0a8-405c-ab1c-409df61e46c4.png)

This use case considers requirement statements like:
* "We are planning to kick-off ingestion (both continuous and historical)... is there anything we should consider before we begin?"
* "We plan to use [H3: Uber's Hexagonal Hierarchical Spatial Index](https://www.uber.com/blog/h3/) for advanced analysis"

### Core Transformations via **Update Policy**
In this step, we will consider transformations to raw data that are permanent and necessary. We will capture those transformation in a function and surface the resulting data to a target table.

Navigate to https://dataexplorer.azure.com/clusters/help/databases/Samples to run KQL queries against the sample **StormEvents** data.

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

This is also a reasonable time (though not the only time) to incorporate data quality considerations {i.e., "drop any rows where the value of Column X is abnormal"}

#### Timestamp
Most customers care about timestamp data because of its value in partitioning.

The StormEvents data has two columns, `StartTime` and `EndTime` and both are type `datetime`. Either is sufficient for our needs; the choice of one, the other, or both should be based on solution context.

If your Timestamp column is of data type `long` (as in the case of a Unix timestamp), you should extend a new `UnixTime_datetime` column with one of various KQL functions like: [unixtime_milliseconds_todatetime()](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/unixtime-milliseconds-todatetimefunction)

#### [Geospatial](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/geospatial-grid-systems)
Most customers care about geospatial data because of the depth that it adds to analytics.

The StormEvents data has two pairs of columns, `BeginLon` :: `BeginLat` and `EndLon` :: `EndLat`.
Using these, we can extend new column(s) using [geo_point_to_h3cell()](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/geo-point-to-h3cell-function) **to address our use case requirements**.

  <img src="https://user-images.githubusercontent.com/44923999/186732552-646affc0-5ebc-43cc-89ca-83af6f32ef97.png" width="800" title="Snipped: August 25, 2022" />

### Reference
https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/updatepolicy
