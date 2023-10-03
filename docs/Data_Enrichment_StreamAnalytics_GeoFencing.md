# Data Enrichment: Stream Analytics, Geo-Fencing

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/92b29e97-31a6-4d39-b494-9d1d4dd5a6c7" width="1000" />

## Use Case
* "Our devices stream hundreds of millions of events daily (including GSP coordinate data)"
* "We need to compare streaming coordinates to known geofence locations"
* "We need to characterize streaming coordinates as 'outside of', 'entering', 'inside of', or 'exiting' known geofence locations"

## Proposed Solution
* Create Stream Analytics Job
* Generate Sample Data
* Automate Comparison

## Solution Requirements
* [**Event Hub**](https://learn.microsoft.com/en-us/azure/event-hubs/) >> Namespace :: Hub :: Consumer Group
* [**Storage Account**](Infrastructure_StorageAccount.md)
* [**Stream Analytics**](https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-introduction) [**Job**](https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-quick-create-portal)

-----

### Exercise 1: Create Stream Analytics Job
In this exercise, we will create and configure a Stream Analytics Job

### Step 1: Lorem Ipsum
Navigate to your Stream Analytics Job, then select "**Inputs**" from the "**Job topology**" group of the navigation pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/589d5957-5d3a-42c7-935e-f9d643c7f47d" width="800" title="Snipped: Oct 3, 2023" />

Click "**Add input**" and select "**Event Hub**" from the resulting dropdown.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b4eb2dca-9034-4d29-9441-059c3bb624cb" width="800" title="Snipped: Oct 3, 2023" />




-----

### Exercise 2: Generate Sample Data
In this exercise, we will fabricate stream data in Event Hub and reference data in Storage Account.

### Step 1: Event Hub Data Generator
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

* [Introducing Event Hubs Data Generator](https://www.microsoft.com/en-gb/industry/blog/technetuk/2020/02/20/introducing-event-hubs-data-generator/)
