# Data Acquisition: Elevation Data from Bing Maps (WiP)

<img src="https://user-images.githubusercontent.com/44923999/236240083-61fb1241-39e5-4d4c-ad0e-3f83480c8edf.png" width="1000" />

## Use Case
This solution considers the following requirements:

* "Our vehicular IoT devices capture GPS coordinates, but not elevation data"
* "As new data streams in, we want to capture elevation data that we do not have from previous runs"
* "We want to query GPS coordinates to no more than five decimal places"

## Prerequisites
This solution requires the following resources:

* [Bing Maps](https://learn.microsoft.com/en-us/bingmaps/) [**Key**](https://learn.microsoft.com/en-us/bingmaps/getting-started/bing-maps-dev-center-help/getting-a-bing-maps-key)
* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [Cluster and Database](https://learn.microsoft.com/en-us/azure/data-explorer/create-cluster-database-portal) with [StormEvents](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data) sample data
* [Synapse Workspace](Infrastructure_Synapse.md) with...
  * Data Explorer [**Linked Service** and **Integration Dataset**](https://learn.microsoft.com/en-us/azure/data-factory/concepts-linked-services)
  * Data Explorer Database, "**User**" and "**Ingestor**" permissions for the Synapse Managed Identity

-----

## Exercise 1: Prepare Database
In this exercise, we will prepare the Data Explorer Database.

Navigate to Data Explorer >> "**Query**".

<img src="https://user-images.githubusercontent.com/44923999/236517174-bc833763-8f55-48f2-82a3-30b69a70fd80.png" width="800" title="Snipped: May 5, 2023" />

### `Elevations_fromAPI` Table
This table will receive data direct from the Bing Maps API (as shaped by the Synapse Pipeline).

Run the following KQL:
```
.create table Elevations_fromAPI (batch: int, points: string, elevations: string) 
```

Schema explained:
* `batch`... grouping mechanism used to optimize our interaction with the API {i.e., sending multiple coordinate pairs per request}
* `points`... set of latitude and longitude values in form `points=lat1,long1,lat2,long2,latn,longn` (the value passed to the API)
* `elevations`... array of values returned by API (ordinally aligned with `points`)

#### Sample Data

batch | points | elevations
:----- | :----- | :-----
`347` |  `22.2083,-159.4784,28.9,-97.2,...` | `[6,50,183,360,...]"`

_Note: Garbage characters {e.g., the double-quote at the end of the sample elevations column data} will be cleaned in downstream logic_

### `Elevations` Table
This table will receive data from an Update Policy (as new data streams to Elevations_fromAPI).

Run the following KQL:
```
.create table Elevations (latitude: real, longitude: real, elevation: int) 
```

#### Sample Result

latitude | longitude | elevation
:----- | :----- | :-----
`41.83` |  `-94.12` | `287`

### `transformElevations` Function
This logic transforms data from `Elevations_fromAPI` to `Elevations` and will be used by the Update Policy.

Run the following KQL:
```
.create-or-alter function transformElevations()  
{
let cleanDynamic = (arg0:string) { todynamic(split(replace_string(replace_string(replace_string(replace_string(arg0,"[",""),"]",""),"\"","")," ",""), ",")) };
let commonBase = (
    Elevations_fromAPI
    | project batch, points = cleanDynamic(points)
    | mv-expand with_itemindex=i value = points to typeof(string)
    | extend type = iif( i/2.0 == i/2, "latitude", "longitude" ), row = i/2 );
let latitudes = ( commonBase | where type == "latitude" | project batch, row, latitude = todecimal(value) );
let longitudes = ( commonBase | where type == "longitude" | project batch, row, longitude = todecimal(value) );
let elevations = (
    Elevations_fromAPI
    | project batch, value = cleanDynamic(elevations)
    | mv-expand with_itemindex=row elevation = value to typeof(string)
    | project batch, row, toint(elevation) );
latitudes
| join kind = inner longitudes on batch, row
| join kind = inner elevations on batch, row
| project latitude, longitude, elevation
}
```

Logic explained:
* `cleanDynamic` is a user-defined function for repetitive transformations {e.g., eliminate garbage characters, format array, change data type}
* `commonBase` expands `points` array into multiple records, adds a `row` index, and surfaces columns
* `let latitudes...` and `let longitudes...`... prepares two in-memory tables from `commonBase`
* `let elevations...` expands `elevations` array into multiple records, adds a `row` index, and surfaces columns
* Final query with `join` operators merges all in-memory tables into a final result

### `Elevations` Update Policy
This logic will trigger when data is added to `Elevations_fromAPI` and write data to `Elevations` using the `transformElevations` function.

`.alter table Elevations policy update '[ { "IsEnabled": true, "Source": "Elevations_fromAPI", "Query": "transformElevations()" } ]'`

_Note: Creation of the Update Policy is an important pre-emptive step to the data ingestion that will happen in the next Exercise_

-----

**Congratulations... you have successfully completed this exercise**

-----

## Exercise 2: Create Pipeline
In this exercise, we will package and orchestrate a Bing Maps API request with a Synapse Pipeline

Navigate to Synapse Studio >> "**Integrate**".

<img src="https://user-images.githubusercontent.com/44923999/236559050-f271eb9f-1858-4062-ba14-e4531bc97d1b.png" width="800" title="Snipped: May 5, 2023" />

Click the "**+**" icon and then click "**Pipeline**" in the resulting drop-down menu.

### Add Activity: `Rows` Lookup

<img src="https://user-images.githubusercontent.com/44923999/236561868-f3a7f1b0-62ae-491f-9245-4c8287af1384.png" width="800" title="Snipped: May 5, 2023" />

Drag-and-drop a "**Lookup**" component from the "**Activities**" tree, "**General**" grouping.<br>
Complete the form on the "**Settings**" tab, including:

Prompt | Entry
:----- | :-----
**Source dataset** | Select the integration dataset
**First row only** | **checked**

Finally, in the "**Query**" input, enter `StormEvents | summarize rowCount = count()`

Logic explained:
* `rowCount` will be used to define batches for iterative processing

### Add Activity: `Points` Lookup

<img src="https://user-images.githubusercontent.com/44923999/236684152-79871a72-b27c-435e-822c-b1f96617d000.png" width="800" title="Snipped: May 7, 2023" />

Drag-and-drop a "**Lookup**" component from the "**Activities**" tree, "**General**" grouping.<br>
Complete the form on the "**Settings**" tab, including:

Prompt | Entry
:----- | :-----
**Source dataset** | Select the integration dataset
**First row only** | **unchecked**

Finally, in the "**Query**" input, enter:

```
let rowCount = @{activity('Rows').output.firstRow.rowCount};
let batchSize = 128;
let batchCount = tolong( rowCount / batchSize );
let cleanString = (arg0:string) { replace_string(replace_string(replace_string(replace_string(arg0,"[",""),"]",""),"\"","")," ","") };
StormEvents
| where not(isnull(BeginLat)) and not(isnull(BeginLon))
| join kind = leftanti Elevations on $left.BeginLat == $right.latitude and $left.BeginLon == $right.longitude
| distinct coordinates = strcat(round(BeginLat,5),",",round(BeginLon,5))
| extend batch = hash_xxhash64(coordinates, batchCount)
| summarize points = make_list(coordinates) by batch
| project batch, points = cleanString(points)
```

Logic explained:
* `let rowCount...` adds the result of the `Rows` lookup as a variable
* `let batchSize...` specifies how many items should be included in each array (documentation specifies "The maximum number of elevations returned in a request is 1024", but I use 128 because I found it was hitting errors when I used 1024)
* `let batchCount...` divides the total number of records {i.e., `rowCount`} by `batchSize`
* `let cleanString...` is a user-defined function for elimination of garbage characters
* `where not(isnull(...` culls records with incomplete coordinate data
* `join kind = leftanti...` culls coordinates that already exist in `Elevations` 
* `distinct...` culls duplicate coordinates 
* `extend batch...` assigns a row number 
* `summarize points = make_list(...` prepares array from coordinates with the same `batch` 

Test by clicking "**Debug**".

#### Sample Result

batch | points
:----- | :-----
`113` |  `["38.9989","-85.9","44.02","-97.3893",...]`

### Add Activity: `ForEach`

<img src="https://user-images.githubusercontent.com/44923999/236684006-db9d279c-9281-4f81-9c49-ee6314f2a333.png" width="800" title="Snipped: May 7, 2023" />

Drag-and-drop a "**ForEach**" component from the "**Activities**" tree, "**Iteration & conditionals**" grouping.<br>
Complete the form on the "**Settings**" tab, including:

Prompt | Entry
:----- | :-----
**Sequential** | Checked
**Items** | Paste expression:<br>`@activity('Points').output.value`

#### Add Sub-Activity: `ForEach` >> `Web`

<img src="https://user-images.githubusercontent.com/44923999/236688310-eb40913b-02da-4e52-9405-38e140a10b13.png" width="800" title="Snipped: May 7, 2023" />

Click the "**+**" button in the "**Activities**" area of the `ForEach` component and then select "**Web**" from the "**General**" grouping of the resulting menu.<br>
Click the new "**Web**" sub-component and complete the form on the "**Settings**" tab.

Prompt | Entry
:----- | :-----
**URL** | Modify and enter:<br>`@concat('http://dev.virtualearth.net/REST/v1/Elevation/List?points=',item().points,'&heights=sealevel&key={BING_MAPS_KEY}')`
**Method** | Select "**GET**"
  
Click "**OK**".

#### Add Sub-Activity: `ForEach` >> `Azure Data Explorer Command`

LOREM IPSUM

-----

**Congratulations... you have successfully completed this exercise**

-----

## Exercise 3: Capture Data
In this exercise, we will capture and then transform the response

-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference

* Bing Maps
  * [Bing Maps Elevations API](https://learn.microsoft.com/en-us/bingmaps/rest-services/elevations/)
  * [Get Elevations](https://learn.microsoft.com/en-us/bingmaps/rest-services/elevations/get-elevations)
* Data Explorer
  * [make_list() (aggregation function)](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/makelist-aggfunction)

