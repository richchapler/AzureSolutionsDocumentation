# Data Enrichment: Stream Analytics, Geo-Fencing (new section, work-in-progress)

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/92b29e97-31a6-4d39-b494-9d1d4dd5a6c7" width="1000" />

## Use Case
* "Our devices stream billions of events daily"
* "We need to compare the streaming data to known geofence locations in near real time"
* "We need to characterize streaming coordinates as 'outside of', 'entering', 'inside of', or 'exiting' known geofence locations"
* "Output should include H3 encoding for the GPS coordinates, based on a set resolution"

## Proposed Solution
* Add Inputs / Outputs: Add inputs and outputs in the Stream Analytics Job
* Generate Sample Data: Fabricate stream data in the Event Hub and reference data in the Storage Account
* Prepare Logic: Prepare functions and query logic in the Stream Analytics Job
* Confirm Success: Test inputs and query logic in the Stream Analytics Job

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

**Congratulations... you have successfully completed this exercise**

-----
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

Click "**Send**".

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

**Congratulations... you have successfully completed this exercise**

-----
-----

## Exercise 3: Prepare Logic
In this exercise, we will prepare functions and query logic in the Stream Analytics Job.

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

### Step 2: Add Function, encodeH3

Lorem Ipsum!!!

```
/* Must install node to run javascript on local machine: https://nodejs.org/en/download */
/* Must install h3-js module from terminal: npm install h3-js */
/* Check version of h3-js module with: npm list h3-js */
/* Call this function with: node encodeh3.js 47.673988 122.121513 12 (Redmond WA coordinates) */

const h3 = require('h3-js');

/* Parameter */
let latitude = process.argv[2];
let longitude = process.argv[3];
let resolution = process.argv[4];

/* Body */
console.log(`Latitude: ${latitude} | Longitude: ${longitude} | Resolution: ${resolution}`);
console.log(`H3 Index: ${h3.latLngToCell(latitude, longitude, resolution)}`);

/* Keep window open until keypress */
const readline = require('readline');
readline.emitKeypressEvents(process.stdin);
process.stdin.setRawMode(true);
process.stdin.once('keypress', () => process.exit());

/* KQL Equivalent
datatable( id: string, latitude: real, longitude: real, precision: int ) [ '1', 47.673988, 122.121513, 12 ] 
| extend h3_encoded = geo_point_to_h3cell(longitude, latitude, precision)
*/
```

-----

### Step 3: Write Query
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
        e.geography,
        udf.parseJSON(s.polygon) polygon,
        ST_WITHIN(e.geography, udf.parseJSON(s.polygon)) as isWithin
    FROM events e CROSS JOIN rchaplersa s
    ),
lookback AS (
    SELECT LAG(*,1) OVER (PARTITION BY id LIMIT DURATION(minute, 5)) AS previous, *
    FROM comparison
    ),
normalized AS (
    SELECT dealer_cd,
        processedOn,
        geography gps_current,
        previous.geography gps_previous,
        polygon geofence_current,
        previous.polygon geofence_previous,
        CASE WHEN isWithin IS NULL THEN 0 ELSE isWithin END iswithin_current,
        CASE WHEN previous.isWithin IS NULL THEN 0 ELSE previous.isWithin END iswithin_previous
    FROM lookback
    )
SELECT *,
    CASE WHEN iswithin_current = 1 AND iswithin_previous = 0 THEN 'ENTER'
        WHEN iswithin_current = 0 AND iswithin_previous = 1 THEN 'EXIT'
        ELSE ''
        END Status
INTO rchaplerdlsfs
FROM normalized
```

#### Logic Explained...

* `WITH events AS (...` is a Common Table Expression (CTE) intended to pull data, first thing, from the stream input
  * `CreatePoint(...` creates a geographical point (usable by later geospatial functions) from latitude and longitude data
* `comparison AS (...` marries stream data with data from the reference input
  * `udf.parseJSON(...` uses the previously-created Function to convert polygon data to JSON
  * `ST_WITHIN(...` determines whether the coordinates from the stream are inside one or more polygon from the reference data
* `lookback AS (...` gets previous records for a given `id` in a five-minute window
  * `LAG(*,1) OVER...` used to get all columns (`*`) from the previous `1` row for each `id`
* `normalized AS (...` standardizes the dataset for troubleshooting
* `CASE WHEN...` determines whether the event represents an entrance to or exit from a geofence polygon

Click "**Save query**".

-----

**Congratulations... you have successfully completed this exercise**

-----
-----

## Exercise 4: Confirm Success
In this exercise, we will test inputs and query logic in the Stream Analytics Job.

### Step 1: Test Query

Navigate to your Stream Analytics Job, then select "**Query**" from the "**Job topology**" group of the navigation pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a3138c70-1901-4e0e-8a76-97312e96d3b4" width="800" title="Snipped: October 4, 2023" />

Click "**Test query**" and confirm result.

```
[{"id":"ABC123","processedOn":"2023-10-03T20:01:40.0896607Z","gps_current":{"type":"Point","coordinates":[-122.123736172,47.6370891183]},"gps_previous":null,"geofence":{"coordinates":[[[10,10],[20,10],[20,20],[10,20],[10,10]]],"type":"Polygon"},"geofence_previous":null,"Status":""}]
```

_Note: Because the GPS coordinates from the event are not in the polygon, status is null_

### Step 2: Test "Enter" / "Exit"

#### Event Hub
Navigate to your Event Hub, then select "**Generate Data**..." from the "**Features**" group of the navigation pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a65ad150-63f4-47d5-9ad0-b2ada0ae1a14" width="800" title="Snipped: October 3, 2023" />

Paste the following JSON in the "**Enter payload**" textbox:

```
[
    {
        "latitude": "11.0",
        "longitude": "11.0"
    }
]
```

Click "**Send**".

#### Stream Analytics
Navigate to your Stream Analytics Job, then select "**Query**" from the "**Job topology**" group of the navigation pane.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/7dc8b125-f3fd-45e6-8858-0501486e97d9" width="800" title="Snipped: October 4, 2023" />

Click on the Event Hub Input, then "**Refresh**" in the "**Input preview**" tab.
<br>You should see the newly sent event.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/bf3be956-a5e4-47fa-8bae-51b942eade8e" width="800" title="Snipped: October 4, 2023" />

Click "**Test query**" and confirm result.

```
[...{"id":"ABC123","processedOn":"2023-10-04T17:39:49.3349609Z","gps_current":{"type":"Point","coordinates":[11,11]},"gps_previous":{"type":"Point","coordinates":[-122.123736172,47.6370891183]},"geofence":{"coordinates":[[[10,10],[20,10],[20,20],[10,20],[10,10]]],"type":"Polygon"},"geofence_previous":{"coordinates":[[[10,10],[20,10],[20,20],[10,20],[10,10]]],"type":"Polygon"},"Status":"**ENTER**"}]
```

_Note: Because the GPS coordinates from the second event are in the polygon, status is ENTER_

-----

**Congratulations... you have successfully completed all exercises**

-----

## Reference

* [Introducing Event Hubs Data Generator](https://www.microsoft.com/en-gb/industry/blog/technetuk/2020/02/20/introducing-event-hubs-data-generator/)
