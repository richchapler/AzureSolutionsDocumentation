# Data Acquisition: Elevation Data from Bing Maps (WiP)

<img src="https://user-images.githubusercontent.com/44923999/236240083-61fb1241-39e5-4d4c-ad0e-3f83480c8edf.png" width="1000" />

## Use Case
This solution considers the following requirements:

* "Our vehicular IoT devices capture GPS coordinates, but not elevation data"
* "We want to enrich data to enhance machine learning models"
* "As new data streams in, we want to enrich the stream with elevation data (based on the most granular possible GPS coordinates)"
* "We want to query Elevation GPS coordinates to five decimal places {i.e., round coordinates with more than five decimal places}"
* Bing Maps API documentation: "The maximum number of elevations returned in a request is 1024"

## Prerequisites
This solution requires the following resources:

* [Bing Maps](https://learn.microsoft.com/en-us/bingmaps/) [**Key**](https://learn.microsoft.com/en-us/bingmaps/getting-started/bing-maps-dev-center-help/getting-a-bing-maps-key)
* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [Cluster and Database](https://learn.microsoft.com/en-us/azure/data-explorer/create-cluster-database-portal) with [StormEvents](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data) sample data
* [Synapse Workspace](Infrastructure_Synapse.md) with...
  * Data Explorer [Linked Service and Integration Dataset](https://learn.microsoft.com/en-us/azure/data-factory/concepts-linked-services)
  * Data Explorer Database, "User" and "Ingestor" permissions for the Synapse Managed Identity

-----

## Exercise 1: Enrich Data using Bing Maps
In this exercise, we will add an "**Elevation**" column to the **StormEvents** table.

## Step 1: Modify Schema

<img src="https://user-images.githubusercontent.com/44923999/236253087-0bf8388c-618d-4046-9d6a-4d05198346af.png" width="800" title="Snipped: May 4, 2023" />

Navigate to Data Explorer >> "**Query**" and then run the following KQL:
```
.alter table StormEvents (
    StartTime: datetime,
    EndTime: datetime,
    EpisodeId: long,
    EventId: long,
    State: string,
    EventType: string,
    InjuriesDirect: long,
    InjuriesIndirect: long,
    DeathsDirect: long,
    DeathsIndirect: long,
    DamageProperty: long,
    DamageCrops: long,
    Source: string,
    BeginLocation: string,
    EndLocation: string,
    BeginLat: real,
    BeginLon: real,
    EndLat: real,
    EndLon: real,
    EpisodeNarrative:string,
    EventNarrative: string,
    StormSummary: string,
    Elevation: int
    ) 
```

## Step 2: Create Pipeline

LOREM IPSUM

StormEvents
| where not(isnull(BeginLat)) and not(isnull(BeginLon))
| distinct BeginLat = round(BeginLat,5), BeginLon = round(BeginLon,5)
| take 128
| summarize points = make_list(strcat(BeginLat,",",BeginLon))
| project points = replace_string(replace_string(replace_string(tostring(points),"[",""),"]",""),"\"","")

@concat("http://dev.virtualearth.net/REST/v1/Elevation/List?points=",item().BeginLat,",",item().BeginLon,"&key=AovJ4RaLmics_D-oHTKlr35bg5S_W4T5m6ualG7i8Lsb09-6K1YvW939JjbPkbto")

http://dev.virtualearth.net/REST/v1/Elevation/List?points=35.52,-81.63,38.8,-75.58&key=AovJ4RaLmics_D-oHTKlr35bg5S_W4T5m6ualG7i8Lsb09-6K1YvW939JjbPkbto

let blah = StormEvents | summarize c=count() | project toint(c/128);
let groupCount = materialize(blah);
StormEvents
| where not(isnull(BeginLat)) and not(isnull(BeginLon))
| distinct coordinates = strcat(round(BeginLat,5),",",round(BeginLon,5))
| extend groupNumber = hash_xxhash64(coordinates, groupCount)
| summarize points = make_list(coordinates) by groupNumber
| project points = replace_string(replace_string(replace_string(tostring(points),"[",""),"]",""),"\"","")

let rowCount = 59066;
let groupSize = 128;
let groupCount = tolong( rowCount / groupSize );
StormEvents
| where not(isnull(BeginLat)) and not(isnull(BeginLon))
| distinct coordinates = strcat(round(BeginLat,5),",",round(BeginLon,5)) // count 27,321
| extend groupNumber = hash_xxhash64(coordinates, groupCount)
| summarize points = make_list(coordinates) by groupNumber
| project points = replace_string(replace_string(replace_string(tostring(points),"[",""),"]",""),"\"","")

.create table Elevations (latitude: decimal, longitude: decimal, elevation: int)

let data = datatable(batch:int, points:string, elevations:string) [
    440,
    " 34.7608,-88.0188,31.8313,-84.8722,32.55,-92.9372,43.4868,-102.2181,32.45,-99.73,39.59,-89.41,33.65,-81.4047,30.5913,-96.6088,38.0655,-102.62,42.0307,-83.8287,39.02,-99.97,38.039,-97.9086,45.7277,-102.67,32.48,-97.82,37.39,-97.7,30.42,-99.3691,47.32,-114.1,46.72,-92.45,34.9284,-105.4927,29.7,-91.2,29.01,-82.04,40.05,-80.68,41.53,-87.43,34.3099,-82.9099,31.43,-90.45,35.77,-89.4,45.54,-100.5,35.95,-86.68,34.75,-95.05,48.7887,-106.3068,29.1746,-99.28,31.5023,-102.6083,41.3349,-74.9528,41.589,-100.2,34.0,-85.87,33.67,-111.13,31.3852,-100.2771,39.96,-76.77,44.92,-69.25,40.78,-73.7227,39.85,-79.91,42.17,-98.18,44.3411,-105.4286,39.5579,-99.7,43.46,-85.95,38.26,-104.61,47.9403,-96.5211,43.7,-71.63,43.4743,-98.6772,35.4651,-94.3945,43.6445,-96.0615,36.62,-89.82,27.9411,-82.3102,33.56,-89.73,36.12,-89.27,38.2465,-85.6883,34.4855,-89.0",
    " [149,96,80,929,533,184,108,88,1148,228,735,473,830,269,413,628,943,358,1983,0,23,362,188,227,133,101,559,224,240,945,238,800,372,890,175,806,567,164,95,20,330,558,1362,670,228,1451,290,353,486,141,471,85,14,121,100,157,142]"
];
let cleanDynamic = (arg0:string) { todynamic(split(replace_string(replace_string(replace_string(replace_string(arg0,"[",""),"]",""),"\"","")," ",""), ",")) };
let commonBase = (
    data
    | project batch, points = cleanDynamic(points)
    | mv-expand with_itemindex=i value = points to typeof(string)
    | extend type = iif( i/2.0 == i/2, "latitude", "longitude" ), row = i/2
);
let latitudes = (
    commonBase
    | where type == "latitude"
    | project batch, row, latitude = value
    );
let longitudes = (
    commonBase
    | where type == "longitude"
    | project batch, row, longitude = value
    );
let elevations = (
    data
    | project batch, elevations = cleanDynamic(elevations)
    | mv-expand with_itemindex=row elevation = elevations to typeof(string)
    | project batch, row, elevation
);
latitudes
| join kind = inner longitudes on batch, row
| join kind = inner elevations on batch, row
| project batch, row, latitude, longitude, elevation

-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference

* Bing Maps
  * [Get Elevations](https://learn.microsoft.com/en-us/bingmaps/rest-services/elevations/get-elevations)
* Data Explorer
  * [make_list() (aggregation function)](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/makelist-aggfunction)

