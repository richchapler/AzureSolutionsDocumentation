Event Hub Data Generator, user-defined payload examples:

```
[
    {
        "latitude": "40.82018471253422",
        "longitude": "-102.97433999137931"
    }
]
```

```
[
    {
        "x": "{\"type\": \"Polygon\",\"coordinates\": [[[0, 0],[3, 6],[6, 1],[0, 0]]]}"
    }
]
```

https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-geospatial-functions

```
SELECT latitude
    , longitude
    , ST_WITHIN(
            CreatePoint(latitude, longitude),
            CreatePolygon(
                CreatePoint(36.992426, -109.060253)
                , CreatePoint(36.992426, -102.041524)
                , CreatePoint(41.003444, -102.041524)
                , CreatePoint(41.003444, -109.060253)
                , CreatePoint(36.992426, -109.060253)
                )
        ) 
INTO rchaplerdls 
FROM rchaplereh
WHERE latitude IS NOT NULL and longitude IS NOT NULL
```

```
WITH X AS (
    SELECT s.latitude lat_s
        ,s.longitude lon_s
        ,g.dealer_cd
        ,g.ordinal
        ,g.latitude lat_g
        ,g.longitude lon_g
    FROM rchaplereh s CROSS JOIN geofence g
),
Y AS (
    SELECT dealer_cd, lon_s longitude, lat_s latitude, Collect(CreatePoint(lon_g,lat_g)) as points
    FROM X
    GROUP BY dealer_cd, lon_s, lat_s, SlidingWindow(second,10)
)
SELECT dealer_cd, longitude, latitude, points, CreatePolygon(points) as geofence
INTO rchaplerdls
FROM Y
```

![image](https://github.com/richchapler/AzureSolutions/assets/44923999/705a6ed9-e672-4298-bb02-8fe7a56317a0)

Sample GeoJSON file (for using Azure Blob Storage, reference)
https://www.kaggle.com/datasets/pompelmo/usa-states-geojson

```
WITH X AS ( SELECT DEALER_CD, REPLACE(SUBSTRING(feature_geometry, 2, LEN(feature_geometry) - 2),'""','"') AS feature_geometry FROM GeoFence )
SELECT JSON_QUERY(feature_geometry, '$.coordinates') FROM X
```

```
SELECT 
    GetRecordPropertyValue(GetArrayElement(feature_geometry.coordinates, 0), '0') AS Longitude,
    GetRecordPropertyValue(GetArrayElement(feature_geometry.coordinates, 0), '1') AS Latitude
FROM [rchaplersds-rchaplersd]
WHERE feature_geometry.type = 'Polygon'
```

## Reference

* [Collect (Azure Stream Analytics)](https://learn.microsoft.com/en-us/stream-analytics-query/collect-azure-stream-analytics)
* [Introduction to Stream Analytics windowing functions](https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-window-functions)
