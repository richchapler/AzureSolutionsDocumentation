# Data Acquisition: Elevation Data from Bing Maps

<img src="https://user-images.githubusercontent.com/44923999/236240083-61fb1241-39e5-4d4c-ad0e-3f83480c8edf.png" width="1000" />

## Use Case
This solution considers the following requirements:

* "Our IoT devices capture GPS coordinates, but not elevation data"
* "We want to enrich our data to enhance our machine learning models"

## Prerequisites
This solution requires the following resources:

* [Bing Maps](https://learn.microsoft.com/en-us/bingmaps/) [**Key**](https://learn.microsoft.com/en-us/bingmaps/getting-started/bing-maps-dev-center-help/getting-a-bing-maps-key)
* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [Cluster and Database](https://learn.microsoft.com/en-us/azure/data-explorer/create-cluster-database-portal) with [StormEvents](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data) sample data
* [Synapse Workspace](Infrastructure_Synapse.md) with...
  * Data Explorer [Linked Service and Integration Dataset](https://learn.microsoft.com/en-us/azure/data-factory/concepts-linked-services)

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

-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference

* Bing Maps
  * [Get Elevations](https://learn.microsoft.com/en-us/bingmaps/rest-services/elevations/get-elevations)
