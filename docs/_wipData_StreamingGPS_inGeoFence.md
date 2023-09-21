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
WITH X AS (
    SELECT s.dealer_cd,
        CreatePoint(e.latitude,e.longitude) as geography,
        s.geojson polygon
    FROM rchaplereh e CROSS JOIN rchaplersd s
),
Y AS (
SELECT dealer_cd,
    geography,
    ST_WITHIN(geography, udf.parseJson(polygon)) as isWithin
FROM X
),
Z AS ( SELECT LAG(*,1) OVER (PARTITION BY dealer_cd LIMIT DURATION(minute, 5)) AS previous, * FROM Y )
SELECT dealer_cd,
    geography gps_current,
    previous.geography gps_previous,
    CASE WHEN isWithin = 1 AND previous.isWithin = 0 THEN 'ENTER'
        WHEN isWithin = 0 AND previous.isWithin = 1 THEN 'EXIT'
        ELSE ''
        END Status
INTO rchaplerdlsfs
FROM Z
```

## Miscellaneous
Sample GeoJSON file (for using Azure Blob Storage, reference)
https://www.kaggle.com/datasets/pompelmo/usa-states-geojson

## Reference

* [Collect (Azure Stream Analytics)](https://learn.microsoft.com/en-us/stream-analytics-query/collect-azure-stream-analytics)
* [Introduction to Stream Analytics windowing functions](https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-window-functions)
