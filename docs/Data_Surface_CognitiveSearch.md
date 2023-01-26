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
* [**Storage Account**](Infrastructure_StorageAccount.md) with container "stormevents" (and related SAS token)

## Proposed Solution
This solution will address requirements in three exercises:

* Exercise 1: Prepare Demonstration Data
* Exercise 2: Cognitive Search Import
* Exercise 3: Prepare Interface

-----

## Exercise 1: Data Exploer, Export Data
In this exercise, we will discuss two ways of preparing Data Explorer-based data for use by Cognitive Search:

* Option #1: One-Time Export (to Blob Storage)... for data that will not change
* Option #2: Continuous Export (to Blob Storage)... for data that will change over time

_Note: Continuous Export does not provide for data headers and that complicates the import of data into Cognitive Search_

### Step 1: Perform One-Time Export (Option #1)
In this step, we will: 1) load sample data and 2) run a KQL query to perform an export to Blob Storage.

Load sample data as specified in [Quickstart: Ingest sample data into Azure Data Explorer](https://learn.microsoft.com/en-us/azure/data-explorer/ingest-sample-data?tabs=ingestion-wizard).

<img src="https://user-images.githubusercontent.com/44923999/214618239-312d447b-230b-4ed2-acbb-2dbf2b520ce7.png" width="800" title="Snipped: January 25, 2023" />

Confirm sample data ingestion.

<img src="https://user-images.githubusercontent.com/44923999/214860857-bad99774-9e00-4875-80a9-8f87e35cc0f2.png" width="800" title="Snipped: January 26, 2023" />

Update {i.e., replace STORAGEACCOUNT_ACCESSKEY with a real value} and then **Run** the following KQL:

```
.export to csv ( "https://rchaplers.blob.core.windows.net/stormevents;STORAGEACCOUNT_ACCESSKEY" )
with ( includeHeaders = "all" ) <| StormEvents
```

<img src="https://user-images.githubusercontent.com/44923999/214619240-351930f8-31e2-4433-8477-f366ec53519d.png" width="800" title="Snipped: January 25, 2023" />

Navigate to the "stormevents" Container and confirm your export.

<img src="https://user-images.githubusercontent.com/44923999/214861628-06359051-426b-4061-80ff-58f2143798d0.png" width="800" title="Snipped: January 26, 2023" />

Open the file with Excel to confirm data quality.

### Step 3: Create Continuous Export (Option #2)
In this step, we will: 1) create an External Table and 2) create a Continuous Export.

#### Step 3a: Create External Table
In this step, we will run a KQL query to create an external table that we can use as a Continuous Export destination.

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

#### Step 3b: Create Continuous Export
In this step, we will: 1) clear sample data previously loaded to Data Explorer, 2) run a KQL query to create a Continuous Export job, and 3) re-import sample data (to trigger Continuous Export)

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

## Exercise 2: Cognitive Search, Import Data
In this exercise, we will import the exported StormEvents data into Cognitive Search.

<img src="https://user-images.githubusercontent.com/44923999/214132098-21179866-e85c-43f8-8737-7bf5efd0ebef.png" width="800" title="Snipped: January 23, 2023" />

Navigate to your instance of Cognitive Search and then click "**Import Data**".

### Step 1: Connect Data

<img src="https://user-images.githubusercontent.com/44923999/214865867-e82b4a08-00ee-488d-8a89-dd87e7fb65a3.png" width="800" title="Snipped: January 26, 2023" />

Complete the resulting "**Connect to your data**" form, including:

| **Prompt** | **Entry** |
| :----- | :----- |
| **Data Source** | Select "**Azure Blob Storage**" |
| **Data Source Name** | Enter a meaningful name aligned with naming standards {e.g., STORAGEACCOUNTNAME-CONTAINERNAME} |
| **Data to extract** | Select "**Content and metadata**" |
| **Parsing mode** | Select "**Delimited text**" |
| **First Line Contains Header** | Checked |
| **Delimiter Character** | Select "**,**" |
| **Connection string** | Use the "**Choose an existing connection**" link |
| **Managed identity authentication** | Select "**None**" |
| **Container name** | Select "**stormevents**" |

Click "**Next: Add cognitive skills (Optional)**".

### Step 2: Add Cognitive Skills

#### Attach Cognitive Services

<img src="https://user-images.githubusercontent.com/44923999/214895862-67c4bc0d-831c-4255-a1ff-1dc8f17697e5.png" width="800" title="Snipped: January 26, 2023" />

Expand "**Attach Cognitive Services**" and then select your instance of Cognitive Services.

#### Add Enrichments

<img src="https://user-images.githubusercontent.com/44923999/214915613-19b182d9-a32d-4e4d-bde2-5b5161521ed1.png" width="800" title="Snipped: January 26, 2023" />

Collapse "**Attach Cognitive Services**" and then expand "**Add enrichments**".

_Some thoughts..._
* _Even familiar data sources can be hard to configure the first time through {e.g., "will Column X have "people names"?}_
* _Looking at the StormEvents data, we see that the **EpisodeNarrative** column includes the most anecdotal data, including references to location_
* _Based on this observation, we can assert that the **EpisodeNarrative** column is a reasonable starting point for completing the "Import Data" wizard, confirm our results and refine iteratively based on what we learn_

Complete the "**Add cognitive skills**..." >> "**Add enrichments**" form, including:

| **Prompt** | **Entry** |
| :----- | :----- |
| **Skillset name** | Enter "stormevents-episodenarrative" |
| **Source data field** | Select **EpisodeNarrative** |
| **Check items below...** | Select "**Extract location names**" and "**Extract key phrases**" |

_Note: In the spirit of experimentation, we might include additional options like "**Extract people names**" or "**Extract organization names**", but given no evidence of that type of data in the **EpisodeNarrative** column, I am choosing to be more targeted_

#### Save enrichments to a knowledge store

<img src="https://user-images.githubusercontent.com/44923999/214929356-8c4ff44e-a21a-4b6f-a08f-f7d8c1e779a1.png" width="800" title="Snipped: January 26, 2023" />

Collapse "**Add enrichments**" and then expand "**Save enrichments to a knowledge store**".
Configure knowledge store options.
Click "**Next: Customize target index**".

### Step 3: Customize Target Index
<img src="https://user-images.githubusercontent.com/44923999/214931224-8b6fe474-0c02-4302-a174-3445881b6eb8.png" width="800" title="Snipped: January 26, 2023" />

_Some thoughts..._
* _There are a LOT of options!_
* _In a first, experimental pass, I am opting for greedy {i.e., selecting all possible choices}_
* _Once I see how those choices play out in the interface, I will come back and be more strategic_

Complete the "**Customize target index**" form, including:

| **Prompt** | **Entry** |
| :----- | :----- |
| **Index name** | Enter "stormevents-index" |
| **Key** | Select "EpisodeId" |
| **Suggester name** | Select "stormevents-suggester" |

...and, of course, select all available options.

Click "**Next: Create an indexer**".

### Step 4: Create Indexer
<img src="https://user-images.githubusercontent.com/44923999/214932334-e23022d2-0c9c-4ae2-8f18-375120e5cfe1.png" width="800" title="Snipped: January 26, 2023" />

Configure and then click **Submit**.

### Step 5: Confirm Success
<img src="https://user-images.githubusercontent.com/44923999/214312163-6f2c0749-ac0b-4a61-9bcd-f2e5b25e50ea.png" width="800" title="Snipped: January 26, 2023" />

On the **Overview** page, click "**Search explorer**"

<img src="https://user-images.githubusercontent.com/44923999/214932918-7896dc21-d2de-4b89-bc56-1c881d255b57.png" width="800" title="Snipped: January 26, 2023" />

Execute the following (very simple) query string: `$top=1`

Results
```
{
  "@odata.context": "https://rchaplerss.search.windows.net/indexes/stormevents-index/$metadata#docs(*)",
  "value": [
    {
      "@search.score": 1,
      "StartTime": "2007-01-03T04:30:00Z",
      "EndTime": "2007-01-03T09:00:00Z",
      "EpisodeId": "MTE3NQ2",
      "EventId": "5041",
      "State": "ALASKA",
      "EventType": "Blizzard",
      "InjuriesDirect": "0",
      "InjuriesIndirect": "0",
      "DeathsDirect": "0",
      "DeathsIndirect": "0",
      "DamageProperty": "0",
      "DamageCrops": "0",
      "Source": "Official NWS Observations",
      "BeginLocation": null,
      "EndLocation": null,
      "BeginLat": null,
      "BeginLon": null,
      "EndLat": null,
      "EndLon": null,
      "EpisodeNarrative": "A storm moved toward Prince william Sound generating strong wind and snow in the western Sound.",
      "EventNarrative": null,
      "AzureSearch_DocumentKey": "https://rchaplers.blob.core.windows.net/stormevents/af63802d-469c-4f37-b33a-f2dd47af2e95_1_882c013a24fc436db61b657f7cbb6802.csv;272",
      "metadata_storage_content_type": "application/octet-stream",
      "metadata_storage_size": 66547959,
      "metadata_storage_last_modified": "2023-01-26T14:24:51Z",
      "metadata_storage_content_md5": "imGCB90ICIv9EaDdsy7X/g==",
      "metadata_storage_name": "af63802d-469c-4f37-b33a-f2dd47af2e95_1_882c013a24fc436db61b657f7cbb6802.csv",
      "metadata_storage_path": "https://rchaplers.blob.core.windows.net/stormevents/af63802d-469c-4f37-b33a-f2dd47af2e95_1_882c013a24fc436db61b657f7cbb6802.csv",
      "metadata_storage_file_extension": ".csv",
      "locations": [
        "Prince william Sound",
        "Sound"
      ],
      "keyphrases": [
        "Prince william Sound",
        "western Sound",
        "strong wind",
        "storm",
        "snow"
      ],
      "StormSummary": {
        "TotalDamages": 0,
        "StartTime": "2007-01-03T04:30:00Z",
        "EndTime": "2007-01-03T09:00:00Z",
        "Details": {
          "Description": "",
          "Location": "ALASKA"
        }
      }
    }
  ]
}
```

Confirm successful data import.

-----

## Exercise 3: Cognitive Search, Prepare Interface
In this exercise, we will prepare a search interface for end-users.

**Lorem Ipsum**

-----

## Reference

* Data Explorer
  * [Export data to storage](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/management/data-export/export-data-to-storage)
  * [Create and alter Azure Storage external tables](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/management/external-tables-azurestorage-azuredatalake)
  * [Create or alter continuous export](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/management/data-export/create-alter-continuous)
* Cognitive Search
  * [What's Azure Cognitive Search?](https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search)
  * [Import data wizard in Azure Cognitive Search](https://learn.microsoft.com/en-us/azure/search/search-import-data-portal)
  * [Quickstart: Create an Azure Cognitive Search index in the Azure portal](https://learn.microsoft.com/en-us/azure/search/search-get-started-portal)
  * [Quickstart: Use Search explorer to run queries in the portal](https://learn.microsoft.com/en-us/azure/search/search-explorer)
