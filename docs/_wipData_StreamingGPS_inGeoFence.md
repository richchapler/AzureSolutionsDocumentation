## Event Hub Data Generator
...user-defined payload example

```
[
    {
        "latitude": "40.82018471253422",
        "longitude": "-102.97433999137931"
    }
]
```

## T-SQL Table
```
CREATE TABLE [dbo].[geojson](
	[dealer_cd] [nvarchar](10) NULL,
	[geojson] [nvarchar](max) NULL
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
```

### Sample data for geojson column
```
{"type":"Polygon", "coordinates": [[ [10.0, 10.0], [20.0, 10.0], [20.0, 20.0], [10.0, 20.0], [10.0, 10.0] ]]}
```

https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-geospatial-functions

## Stream Analytics 'parseJson' Logic
```
function parseJson (strjson) { return JSON.parse(strjson); }
```

## Stream Analytics Query Logic
```
WITH events AS (
    SELECT e.EventProcessedUtcTime as processedOn,
        CreatePoint(e.latitude,e.longitude) as geography
    FROM rchaplereh e
    ),
comparison AS (
    SELECT s.dealer_cd,
        e.processedOn,
        e.geography, -- streamed coordinates
        udf.parseJson(s.feature_geometry) polygon, -- geofence
        ST_WITHIN(e.geography, udf.parseJson(s.feature_geometry)) as isWithin
    FROM events e CROSS JOIN rchaplers s
    ),
lookback AS (
    SELECT LAG(*,1) OVER (PARTITION BY dealer_cd LIMIT DURATION(minute, 5)) AS previous, *
    FROM comparison
    )
SELECT dealer_cd,
    processedOn,
    geography gps_current,
    previous.geography gps_previous,
    polygon geofence,
    previous.polygon geofence_previous,
    CASE WHEN isWithin = 1 AND previous.isWithin = 0 THEN 'ENTER'
        WHEN isWithin = 0 AND previous.isWithin = 1 THEN 'EXIT'
        ELSE ''
        END Status
INTO rchaplerdlsfs
FROM lookback
```

## Miscellaneous
Sample GeoJSON file (for using Azure Blob Storage, reference)
https://www.kaggle.com/datasets/pompelmo/usa-states-geojson

## Reference

* [Collect (Azure Stream Analytics)](https://learn.microsoft.com/en-us/stream-analytics-query/collect-azure-stream-analytics)
* [Introduction to Stream Analytics windowing functions](https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-window-functions)
