## Data Acquisition... Pre-Ingestion Checklist

![image](https://user-images.githubusercontent.com/44923999/185980410-353cda9e-d0a8-405c-ab1c-409df61e46c4.png)

This use case considers requirement statements like:
* "We are planning to kick-off ingestion (both continuous and historical)... is there anything we should consider before we begin?"
* "We plan to use [H3: Uber's Hexagonal Hierarchical Spatial Index](https://www.uber.com/blog/h3/) for advanced analysis"
* "We need to extract X from dynamic column Y"

### Core Transformations via **Update Policy**
In this step, we will consider transformations to raw data that are permanent and necessary. We will capture those transformation in a function and surface the resulting data to a target table.

Navigate to https://dataexplorer.azure.com/clusters/help/databases/Samples to run KQL queries against the sample **StormEvents** data.

  ```  
  StormEvents
  | take 25
  ```
  
  <img src="https://user-images.githubusercontent.com/44923999/186710088-4b80f89b-36da-437e-8686-48581d5ff07e.png" width="800" title="Snipped: August 25, 2022" />

#### Minimum Viable...
Consider beginning every KQL exercise with thinning {i.e., "what columns and rows can I drop from downstream processing?"}. Because update policies process data only during ingestion {i.e., no updates after initial processing}, we want to be sure to consider every possible downstream need. If there is a chance you might need column X, better to materialize it in the more-efficient, resulting table than not.

In the following example, we limit the original set of columns with `project` and rows with `where`.

  ```  
  StormEvents
  | where not(isempty(BeginLon))
  | project EventId, EventType, StartTime, BeginLon, BeginLat, StormSummary
  ```  

  <img src="https://user-images.githubusercontent.com/44923999/186737820-ae23621b-1421-41de-a1e3-bf64b2ece6c4.png" width="800" title="Snipped: August 25, 2022" />

This is also a reasonable time (though not the only time) to incorporate data quality considerations {i.e., "drop any rows where the value of Column X is abnormal"}

#### Timestamp
Most customers care about timestamp data because of its value in partitioning.

The StormEvents data has two columns, `StartTime` and `EndTime` and both are type `datetime`. Either is sufficient for our needs; the choice of one, the other, or both should be based on solution context.

If your Timestamp column is of data type `long` (as in the case of a Unix timestamp), you should extend a new `UnixTime_datetime` column with one of various KQL functions like: [unixtime_milliseconds_todatetime()](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/unixtime-milliseconds-todatetimefunction)

#### Geospatial
Most customers care about [geospatial](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/geospatial-grid-systems) data because of the depth that it adds to analytics.

The StormEvents data has two pairs of columns, `BeginLon` :: `BeginLat` and `EndLon` :: `EndLat`.

  <img src="https://user-images.githubusercontent.com/44923999/186732552-646affc0-5ebc-43cc-89ca-83af6f32ef97.png" width="800" title="Snipped: August 25, 2022" />

Using these columns and [geo_point_to_h3cell()](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/geo-point-to-h3cell-function) **we can address our use case requirements** with KQL like:

```| extend H3_Resolution6 = geo_point_to_h3cell(BeginLon, BeginLat, 6)```

#### Dynamic Arrays
Most customers care about pre-parsing dynamic arrays because inline processing is memory- and compute-intensive.

The StormEvents data has one dynamic column, `StormSummary` which contains JSON data:

```
{"TotalDamages":0,"StartTime":"2007-01-01T00:00:00.0000000Z","EndTime":"2007-01-27T14:00:00.0000000Z","Details":{"Description":"At the Petersburg river gage, the White River crested at 25.09 feet on the 21st. This is almost ten feet above the flood stage of 16 feet. Although the flooding was moderately severe, this crest was over two and a half feet lower than the major flood in January, 2005. Extensive bottomland flooding affected agricultural and rural residential areas. Several rural county roads flooded. A few residents moved out of an area locally known as Dodge City. Several small oil fields were inaccessible. Since this occurred outside of agricultural season, little or no crop damage occurred. High water isolated some river cabin residents. State Road 257 was completely flooded.","Location":"INDIANA"}}
```

<br>We can parse desired values from this dynamic column with KQL like:

```| extend TotalDamages = StormSummary.TotalDamages```

### Reference
https://docs.microsoft.com/en-us/azure/data-explorer/kusto/management/updatepolicy
