# Data Enrichment: Stream Analytics, Geo-Fencing

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/92b29e97-31a6-4d39-b494-9d1d4dd5a6c7" width="1000" />

## Use Case
* "Our devices stream billions of events daily"
* "We need to compare GPS coordinates from the streaming data to known geofence locations in near real time"
* "We need to characterize streaming coordinates as 'outside of', 'entering', 'inside of', or 'exiting' known geofence locations"

## Proposed Solution
* Add Inputs / Outputs
* Generate Sample Data
* Prepare Function / Query

## Solution Requirements
* [**Event Hub**](https://learn.microsoft.com/en-us/azure/event-hubs/) >> Namespace :: Hub :: Consumer Group
* [**Storage Account**](Infrastructure_StorageAccount.md)
* [**Stream Analytics**](https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-introduction) [**Job**](https://learn.microsoft.com/en-us/azure/stream-analytics/stream-analytics-quick-create-portal)

-----

## Exercise 1: Add Inputs / Outputs
In this exercise, we will add inputs and outputs in the Stream Analytics Job.

### Step 1: Add Stream Input, Event Hub
Navigate to your Stream Analytics Job, then select "**Inputs**" from the "**Job topology**" group of the navigation pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/589d5957-5d3a-42c7-935e-f9d643c7f47d" width="800" title="Snipped: October 3, 2023" />

Click "**Add input**" and select "**Event Hub**" from the "**Stream input**" group in the resulting dropdown.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/1af03d71-adc8-44d6-ba4c-e07d3f01b37e" width="800" title="Snipped: October 3, 2023" />

Complete the resulting "**Event Hub**" >> "**New Input**" popout and click "**Save**".

### Step 2: Add Reference Input, Blob Storage
Navigate to your Stream Analytics Job, then select "**Inputs**" from the "**Job topology**" group of the navigation pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e5e52708-0966-42e0-adfc-d13591c1e84a" width="800" title="Snipped: October 3, 2023" />

Click "**Add input**" and select "**Blob storage/ADLS Gen2**" from the "**Reference input**" group in the resulting dropdown.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2dcc9363-0a28-40b2-af92-f1b4f71df795" width="800" title="Snipped: October 3, 2023" />

Complete the resulting "**Blob storage/ADLS Gen2**" >> "**New Input**" popout and click "**Save**".

### Step 3: Add Output, ADLS Gen2

Navigate to your Stream Analytics Job, then select "**Outputs**" from the "**Job topology**" group of the navigation pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/3bc3ee8d-e63c-4a0a-9ba0-2062ffcfe1cb" width="800" title="Snipped: October 3, 2023" />

Click "**Add output**" and select "**Blob storage/ADLS Gen2**" from the the resulting dropdown.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/98d91db0-9b1f-4001-a716-53bbc0f12dc4" width="800" title="Snipped: October 3, 2023" />

Complete the resulting "**Blob storage/ADLS Gen2**" >> "**New Output**" popout and click "**Save**".

-----

## Exercise 2: Generate Sample Data
In this exercise, we will fabricate stream data in the Event Hub and reference data in the Storage Account.

### Step 1: Event Hub
Navigate to your Event Hub, then select "**Generate Data**..." from the "**Features**" group of the navigation pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6638fc3f-ce28-4c8c-878b-c14f5bd8c0aa" width="800" title="Snipped: October 3, 2023" />

Paste the following JSON in the "**Enter payload**" textbox:

```
[
    {
        "latitude": "47.6370891183",
        "longitude": "-122.123736172"
    }
]
```

Click "Send".

_Note: This will create a single event that we will pickup in Stream Analytics. You will repeat this in later steps._

#### Confirm Success
Navigate to your Stream Analytics Job, then select "**Query**" from the "**Job topology**" group of the navigation pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/ba1e7fae-f7ee-4bb5-b3a7-517dada20291" width="800" title="Snipped: October 3, 2023" />

Click on your Event Hub Input, then "**Refresh**" on the "**Input Preview**" tab.
<br>You can expect to see the data sent from the Event Hub Data Generator.

### Step 2: Storage Account
Use a text editor to create a CSV file with the following content:

```
id, polygon
ABC123, "{ ""coordinates"": [ [ [ 10.0, 10.0 ], [ 20.0, 10.0 ], [ 20.0, 20.0 ], [ 10.0, 20.0 ], [ 10.0, 10.0 ] ] ], ""type"": ""Polygon""}"
```

_Note: The GPS coordinates for this polygon were not chosen because they are meaningful._

Upload the CSV file to your Storage Account Container.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/768cd4e7-a276-4bd6-97e9-4c267d1d9070" width="800" title="Snipped: October 3, 2023" />

#### Confirm Success
Navigate to your Stream Analytics Job, then select "**Query**" from the "**Job topology**" group of the navigation pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2b41e7ba-d7c1-41ab-88ac-317dde42dc31" width="800" title="Snipped: October 3, 2023" />

Click on your Storage Account Input, then "**Upload sample input**" on the "**Input Preview**" tab.
<br>Upload the CSV file and confirm that the correct data surfaces.

-----

## Exercise 3: Prepare Function / Query
In this exercise, we will prepare function and query logic in the Stream Analytics Job.

### Step 1: Add Function, parseJSON
Navigate to your Stream Analytics Job, then select "**Functions**" from the "**Job topology**" group of the navigation pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/dfd195d3-6fce-48b2-b9ba-ea8b844121c2" width="800" title="Snipped: October 4, 2023" />

Click "**Add function**" and select "**Javascript UDF**" from the the resulting dropdown.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a663e0b0-122c-405a-8996-0a60a1d48697" width="800" title="Snipped: October 4, 2023" />

Complete the resulting "**Javascript function**" page, including the following code:

```
function parseJson (strjson) { return JSON.parse(strjson); }
```

Click "**Save**".

-----

### Step 2: Write Query
Navigate to your Stream Analytics Job, then select "**Query**" from the "**Job topology**" group of the navigation pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b5c65f6d-5baf-49be-877c-dc86bec8e01d" width="800" title="Snipped: October 4, 2023" />

Paste the following query logic:

```
WITH events AS (
    SELECT e.EventProcessedUtcTime as processedOn,
        CreatePoint(e.latitude,e.longitude) as geography
    FROM rchaplereh e
    ),
comparison AS (
    SELECT s.id,
        e.processedOn,
        e.geography, -- streamed coordinates
        udf.parseJSON(s.polygon) polygon, -- geofence
        ST_WITHIN(e.geography, udf.parseJSON(s.feature_geometry)) as isWithin
    FROM events e CROSS JOIN rchaplers s
    ),
lookback AS (
    SELECT LAG(*,1) OVER (PARTITION BY id LIMIT DURATION(minute, 5)) AS previous, *
    FROM comparison
    )
SELECT id,
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

#### Logic Explained...

* `WITH events AS (...` is a Common Table Expression (CTE) intended to pull data, first thing, from the stream input
  * `CreatePoint(...` creates a geographical point (usable by later geospatial functions) from latitude and longitude data
* `comparison AS (...` marries stream data with data from the reference input
  * `udf.parseJSON(...` uses the previously-created Function to convert polygon data to JSON
  * `ST_WITHIN(...` determines whether the coordinates from the stream are inside one or more polygon from the reference data
* `lookback AS (...` gets previous records for a given `id` in a five-minute window
  * `LAG(*,1) OVER...` used to get all columns (`*`) from the previous `1` row for each `id`
* `CASE WHEN...` determines whether the event represents an entrance to or exit from a geofence polygon

-----

### Step 3: Confirm Success
Lorem Ipsum

-----

**Congratulations... you have successfully completed all exercises**

-----

## Reference

* [Introducing Event Hubs Data Generator](https://www.microsoft.com/en-gb/industry/blog/technetuk/2020/02/20/introducing-event-hubs-data-generator/)
