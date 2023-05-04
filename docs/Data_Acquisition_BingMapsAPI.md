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


.alter table Elevations_fromAPI (batch: int, points: string, elevations: string)

.clear table Elevations_fromAPI data

Elevations_fromAPI
| project batch, elevations = split(replace_string(replace_string(replace_string(replace_string(tostring(elevations),"[",""),"]",""),"\"","")," ",""), ",")
| mv-expand
    bagexpansion=array
    with_itemindex=row
    elevation = todynamic(elevations) to typeof(string)
| project batch, row, elevation
| join kind=inner (
    Elevations_fromAPI
    | project batch, points = split(replace_string(replace_string(replace_string(replace_string(tostring(points),"[",""),"]",""),"\"","")," ",""), ",")
    | mv-expand
        bagexpansion=array
        with_itemindex=i
        latitude = todynamic(points) to typeof(string)
    | where i/2.0==i/2
    | project batch, row=1/2, latitude
    ) on batch, row
| join kind=inner (
    Elevations_fromAPI
    | project batch, points = split(replace_string(replace_string(replace_string(replace_string(tostring(points),"[",""),"]",""),"\"","")," ",""), ",")
    | mv-expand
        bagexpansion=array
        with_itemindex=i
        longitude = todynamic(points) to typeof(string)
    | where i/2.0!=i/2
    | project batch, row=1/2, longitude
    ) on batch, row
| project batch, row, elevation, latitude, longitude

-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference

* Bing Maps
  * [Get Elevations](https://learn.microsoft.com/en-us/bingmaps/rest-services/elevations/get-elevations)
* Data Explorer
  * [make_list() (aggregation function)](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/makelist-aggfunction)

