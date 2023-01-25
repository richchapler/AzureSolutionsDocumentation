# Surface Data with Cognitive Search
_Note: Cognitive Search is also known as "Search Services" and "Azure Search"_

![image](https://user-images.githubusercontent.com/44923999/214475405-ae4cd7af-d318-49e7-854d-3060609c2124.png)

## Use Case
This solution considers the following requirements:

* "We used Data Explorer to capture a significant amount of rich, historical data from a soon-to-be-deprecated, on-prem database"
* "Most of the historical data is static, but a small percentage will change over time"
* "We want to help business users search this data asset"

## Required Infrastructure
This solution requires the following resources:

* [**Cognitive Search**](https://azure.microsoft.com/en-us/products/search)
* [**Cognitive Services**](https://learn.microsoft.com/en-us/azure/cognitive-services/)
* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [**Cluster**](Infrastructure_DataExplorer_Cluster.md) and [**Database**](Infrastructure_DataExplorer_Database.md)
* [**Storage Account**](Infrastructure_StorageAccount.md) with container "stormevents"

## Proposed Solution
This solution will address requirements in three exercises:

* Exercise 1: Prepare Demonstration Data
* Exercise 2: Cognitive Search Import
* Exercise 3: Create Search Index (pending)
* Exercise 4: Prepare Interface (pending)

-----

## Exercise 1: Prepare Demonstration Data
In this exercise, we will discuss two ways of preparing Data Explorer-based data for use by Cognitive Search:

* Option #1: One-Time Export (to Blob Storage)... for data that will not change
* Option #2: Continuous Export (to Blob Storage)... for data that will change over time

### Step 1: Create External Table
Both One-Time and Continuous Export use an External Table to interact with data stored in an external data source (such as Blob Storage).

In this step, we will run a KQL query to create an external table that we can use as an export destination.

<img src="https://user-images.githubusercontent.com/44923999/214453289-002dc1bf-24aa-4883-af20-34e3fb300f62.png" width="800" title="Snipped: January 24, 2023" />

Navigate to Data Explorer, and then "**Query**" in the "**Data**" grouping of the left-hand navigation pane.

Update {i.e., replace STORAGEACCOUNT_ACCESSKEY with a real value} and then **Run** the following KQL:

```
.create external table eStormEvents (
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
    EpisodeNarrative: string,
    EventNarrative: string,
    StormSummary: string
    )
kind = storage
dataformat = csv ( 'https://rchaplers.blob.core.windows.net/stormevents;STORAGEACCOUNT_ACCESSKEY' )
```

_Note: The external table name is prefixed with the letter "e" {i.e., "eStormEvents"} because the sample data uses table name "StormEvents"_

### Step 2: Perform One-Time Export (Option #1)
In this step, we will: 1) load sample data and 2) run a KQL query to perform an export to Blob Storage.

Load sample data as specified in [Quickstart: Ingest sample data into Azure Data Explorer](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data?tabs=ingestion-wizard)

<img src="https://user-images.githubusercontent.com/44923999/214618239-312d447b-230b-4ed2-acbb-2dbf2b520ce7.png" width="800" title="Snipped: January 25, 2023" />

Confirm sample data ingestion and then **Run** the following KQL:

```
.export to table eStormEvents <| StormEvents
```

<img src="https://user-images.githubusercontent.com/44923999/214618664-ff0186ba-772b-4a52-abfb-f9d71d8ea7b0.png" width="800" title="Snipped: January 25, 2023" />

Navigate to your Storage Account, then "**Storage browser**" in the left-hand navigation

<img src="https://user-images.githubusercontent.com/44923999/214619240-351930f8-31e2-4433-8477-f366ec53519d.png" width="800" title="Snipped: January 25, 2023" />

Navigate to the "stormevents" Container and confirm your export.

### Step 3: Create Continuous Export (Option #2)
In this step, we will: 1) clear previously loaded sample data, 2) run a KQL query to create a Continuous Export job, and 3) re-import sample data (to trigger Continuous Export)

Navigate to Data Explorer, and then "**Query**" in the "**Data**" grouping of the left-hand navigation pane.

<img src="https://user-images.githubusercontent.com/44923999/214656930-8ae8c556-40f2-4764-a54d-e7af6382845a.png" width="800" title="Snipped: January 25, 2023" />

Clear previously imported StormEvents data by running the following KQL:

```
.clear table StormEvents data
```

<img src="https://user-images.githubusercontent.com/44923999/214656813-f859e3a8-fb82-49f7-9a6b-376419f0eba9.png" width="800" title="Snipped: January 25, 2023" />

Create a Continuous Export by running the following KQL:

```
.create-or-alter continuous-export ceStormEvents to table eStormEvents with ( intervalBetweenRuns = 1m )
<| StormEvents
```

_Note: Once you have activated Continuous Export, any newly imported data will be exported per schedule {e.g., every one minute for `intervalBetweenRuns = 1m`}_

<img src="https://user-images.githubusercontent.com/44923999/214658454-38befdd7-ae1e-4702-b96a-29b1d598ac0e.png" width="800" title="Snipped: January 25, 2023" />

Re-load sample data as specified in [Quickstart: Ingest sample data into Azure Data Explorer](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data?tabs=ingestion-wizard)

<img src="https://user-images.githubusercontent.com/44923999/214658739-5f91e5e3-7758-4664-89a5-6ca3e13f4a42.png" width="800" title="Snipped: January 25, 2023" />

Navigate back to the "stormevents" Container and confirm the second export.

-----

## Exercise 2: Cognitive Search Import
In this exercise, we will import built-in sample data.

<img src="https://user-images.githubusercontent.com/44923999/214132098-21179866-e85c-43f8-8737-7bf5efd0ebef.png" width="800" title="Snipped: January 23, 2023" />

Navigate to your instance and then click "**Import Data**"

### Step 1: Connect Data
On the "**Connect to your data**" tab, select **Samples** from the "**Existing data source**" drop-down menu

  <img src="https://user-images.githubusercontent.com/44923999/214140498-3b1a486c-57bd-407a-b249-60a93e54602e.png" width="800" title="Snipped: January 23, 2023" />

Select the "**realestate-us-sample**" row and then click "**Next: Add cognitive skills (Optional)**"

### Step 2: Add Cognitive Skills
<img src="https://user-images.githubusercontent.com/44923999/214144851-f5be1f08-374d-4b8d-8fed-b56e6a7980a7.png" width="800" title="Snipped: January 23, 2023" />

Expand "**Attach Cognitive Services**" and then select your instance of Cognitive Services

<img src="https://user-images.githubusercontent.com/44923999/214149017-338ba6c7-3281-40ee-92ca-e8b099b5430f.png" width="800" title="Snipped: January 23, 2023" />

Collapse "**Attach Cognitive Services**", expand "**Add enrichments**", and then add appropriate enrichments<br>
_Note: Even familiar data sources can be hard to configure the first time through {e.g., "will the Description field have "people names"?}. I lean towards experimentation in this situation... we can always re-configure once the initial Import Data exercise is complete._

<img src="https://user-images.githubusercontent.com/44923999/214153287-beed5d62-200a-48ec-b852-578f38621af6.png" width="800" title="Snipped: January 23, 2023" />

Collapse "**Add enrichments**", expand "**Save enrichments to a knowledge store**", configure knowledge store options and then click "**Next: Customize target index**"

### Step 3: Customize Index
<img src="https://user-images.githubusercontent.com/44923999/214154309-fea824d2-d3bf-435b-bf85-500dd28be462.png" width="800" title="Snipped: January 23, 2023" />

Configure index options and then click "**Next: Create an indexer**"

### Step 4: Create Indexer
<img src="https://user-images.githubusercontent.com/44923999/214154948-102c8185-cbdc-44f9-b21f-12bb9a22a43b.png" width="800" title="Snipped: January 23, 2023" />

Configure and then click **Submit**

### Step 5: Confirm Success
<img src="https://user-images.githubusercontent.com/44923999/214312163-6f2c0749-ac0b-4a61-9bcd-f2e5b25e50ea.png" width="800" title="Snipped: January 24, 2023" />

On the **Overview** page, click "**Search explorer**"

<img src="https://user-images.githubusercontent.com/44923999/214313821-47edb62f-b48b-4a25-8893-7668224abb9b.png" width="800" title="Snipped: January 24, 2023" />

"**Search Explorer**" supports your efforts to:
* Interact with and test a search index
* Test and debug search queries
* Craft a Request URL (for API calls)

Execute the following (very simple) query string: `$top=1`

Results
```
{
  "@odata.context": "https://rchaplercs.search.windows.net/indexes/realestate-us-sample-index/$metadata#docs(*)",
  "value": [
    {
      "@search.score": 1,
      "listingId": "OTM4MjI2NQ2",
      "beds": 5,
      "baths": 4,
      "description": "This is a apartment residence and is perfect for entertaining.  This home provides lakefront property located close to parks and features a detached garage, beautiful bedroom floors and lots of storage.",
      "description_de": "Dies ist eine Wohnanlage und ist perfekt für Unterhaltung.  Dieses Haus bietet Seeliegenschaft Parks in der Nähe und verfügt über eine freistehende Garage schöne Zimmer-Etagen and viel Stauraum.",
      "description_fr": "Il s’agit d’un appartement de la résidence et est parfait pour se divertir.  Cette maison offre propriété au bord du lac Situé à proximité de Parcs et dispose d’un garage détaché, planchers de belle chambre and beaucoup de rangement.",
      "description_it": "Si tratta di un appartamento residence ed è perfetto per intrattenere.  Questa casa fornisce proprietà lungolago Situato vicino ai parchi e dispone di un garage indipendente, piani di bella camera da letto and sacco di stoccaggio.",
      "description_es": "Se trata de una residencia Apartamento y es perfecto para el entretenimiento.  Esta casa ofrece propiedad de lago situado cerca de parques y cuenta con un garaje independiente, pisos de dormitorio hermoso and montón de almacenamiento.",
      "description_pl": "Jest to apartament residence i jest idealny do zabawy.  Ten dom zapewnia lakefront Wlasciwosc usytuowany w poblizu parków i oferuje garaz wolnostojacy, piekna sypialnia podlogi and mnóstwo miejsca do przechowywania.",
      "description_nl": "Dit is een appartement Residentie en is perfect voor entertaining.  Dit huis biedt lakefront eigenschap vlakbij parken en beschikt over een vrijstaande garage, mooie slaapkamer vloeren and veel opslag.",
      "sqft": 12960,
      "daysOnMarket": 9,
      "status": "sold",
      "source": "Pérez Realty",
      "number": "19339",
      "street": "Linden Avenue North",
      "unit": "658",
      "type": "Apartment",
      "city": "Shoreline",
      "region": "wa",
      "countryCode": "us",
      "postCode": "98133",
      "location": {
        "type": "Point",
        "coordinates": [
          -122.35,
          47.7699
        ],
        "crs": {
          "type": "name",
          "properties": {
            "name": "EPSG:4326"
          }
        }
      },
      "price": 3693600,
      "thumbnail": "https://searchdatasets.blob.core.windows.net/images/bd5bt4apt.jpg",
      "tags": [
        "apartment residence",
        "entertaining",
        "lakefront property",
        "parks",
        "detached garage",
        "beautiful bedroom floors",
        "lots of storage"
      ],
      "organizations": [],
      "locations": [
        "apartment residence",
        "home",
        "parks",
        "garage"
      ],
      "people": [],
      "keyphrases": [
        "beautiful bedroom floors",
        "apartment residence",
        "lakefront property",
        "detached garage",
        "home",
        "parks",
        "lots",
        "storage"
      ],
      "masked_text": "This is a apartment residence and is perfect for entertaining.  This home provides lakefront property located close to parks and features a detached garage, beautiful bedroom floors and lots of storage.",
      "pii_entities": []
    }
  ]
}
```

Confirm successful data import.

-----------------------

## Reference

* Data Explorer
  * [Create and alter Azure Storage external tables](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/management/external-tables-azurestorage-azuredatalake)
  * [Export data to an external table](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/management/data-export/export-data-to-an-external-table)
  * [Continuous data export overview](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/management/data-export/continuous-data-export)
* Cognitive Search
  * [What's Azure Cognitive Search?](https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search)
  * [Quickstart: Create an Azure Cognitive Search index in the Azure portal](https://learn.microsoft.com/en-us/azure/search/search-get-started-portal)
  * [Quickstart: Use Search explorer to run queries in the portal](https://learn.microsoft.com/en-us/azure/search/search-explorer)
