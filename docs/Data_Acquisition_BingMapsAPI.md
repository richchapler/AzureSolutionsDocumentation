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

<img src="https://user-images.githubusercontent.com/44923999/236506885-3a206fa4-50f9-4b5f-8c58-2f5de48c2a5b.png" width="800" title="Snipped: May 5, 2023" />

Navigate to Data Explorer >> "**Query**".

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
`347` |  `22.2083,-159.4784,28.9,-97.2...` | `[6,50,183,360,621,228...]"`

### `Elevations` Table
This table will receive data from an Update Policy (as new data streams to Elevations_fromAPI).

Run the following KQL:
```
.create table Elevations (latitude: real, longitude: real, elevation: int) 
```

### Function, Lorem Ipsum
This table will receive data from an Update Policy (as new data streams to Elevations_fromAPI).

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




```
StormEvents
| where not(isnull(BeginLat)) and not(isnull(BeginLon))
| distinct BeginLat = round(BeginLat,5), BeginLon = round(BeginLon,5)
| take 128
| summarize points = make_list(strcat(BeginLat,",",BeginLon))
| project points = replace_string(replace_string(replace_string(tostring(points),"[",""),"]",""),"\"","")
```
```
@concat("http://dev.virtualearth.net/REST/v1/Elevation/List?points=",item().BeginLat,",",item().BeginLon,"&key=AovJ4RaLmics_D-oHTKlr35bg5S_W4T5m6ualG7i8Lsb09-6K1YvW939JjbPkbto")
```

```
http://dev.virtualearth.net/REST/v1/Elevation/List?points=35.52,-81.63,38.8,-75.58&key=AovJ4RaLmics_D-oHTKlr35bg5S_W4T5m6ualG7i8Lsb09-6K1YvW939JjbPkbto
```

```
let blah = StormEvents | summarize c=count() | project toint(c/128);
let groupCount = materialize(blah);
StormEvents
| where not(isnull(BeginLat)) and not(isnull(BeginLon))
| distinct coordinates = strcat(round(BeginLat,5),",",round(BeginLon,5))
| extend groupNumber = hash_xxhash64(coordinates, groupCount)
| summarize points = make_list(coordinates) by groupNumber
| project points = replace_string(replace_string(replace_string(tostring(points),"[",""),"]",""),"\"","")
```

```
let rowCount = 59066;
let groupSize = 128;
let groupCount = tolong( rowCount / groupSize );
StormEvents
| where not(isnull(BeginLat)) and not(isnull(BeginLon))
| distinct coordinates = strcat(round(BeginLat,5),",",round(BeginLon,5)) // count 27,321
| extend groupNumber = hash_xxhash64(coordinates, groupCount)
| summarize points = make_list(coordinates) by groupNumber
| project points = replace_string(replace_string(replace_string(tostring(points),"[",""),"]",""),"\"","")
```

```
.create table Elevations (latitude: decimal, longitude: decimal, elevation: int)
```

```
.create table Elevations (latitude: decimal, longitude: decimal, elevation: int)
```



```
.alter table Elevations policy update '[ { "IsEnabled": true, "Source": "Elevations_fromAPI", "Query": "transformElevations()" } ]'
```

-----

**Congratulations... you have successfully completed this exercise**

-----

## Exercise 2: Create Pipeline
In this exercise, we will package and orchestrate a Bing Maps API request with a Synapse Pipeline

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

