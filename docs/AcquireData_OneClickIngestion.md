[Acquire Data](AcquireData.md) > No Code > "One-Click" Ingestion

## Use Case

Requirement statements:

* "Devices running Application X, running around the world, continuously drop files into the Data Lake"
* “We want to automatically ingest data files as they appear”

## Prerequisites

This solution requires the following resources:

* [Data Explorer Cluster](https://github.com/richchapler/AzureDataSolutions/wiki/Data-Explorer-Cluster) (with [database](https://github.com/richchapler/AzureDataSolutions/wiki/Data-Explorer-Database))
* [Data Lake](https://github.com/richchapler/AzureDataSolutions/wiki/Data-Lake) (with [container](https://github.com/richchapler/AzureDataSolutions/wiki/Data-Lake-Container) and downloaded [sample data](https://github.com/richchapler/AzureDataSolutions/wiki/Sample-Data))

## One-Time Ingestion

Complete the following steps:

* Navigate to https://dataexplorer.azure.com/ and then click **Data** in the navigation pane
* Click **Ingest Data** on the resulting **Data Management** page

  <img src="https://user-images.githubusercontent.com/44923999/178302280-6dad3275-e252-4368-8f69-fa5b098dd1bb.png" width="750" title="Snipped: July 11, 2022" />

* On the resulting **Ingest data** page, **1. Destination** tab, select your Data Explorer cluster and database, then select the **New table** radio button, enter the name **sample1** (to match our downloaded sample file) and finally, click **Next: Source**

  <img src="https://user-images.githubusercontent.com/44923999/178302589-c0843482-0d4d-4626-a224-28cac8771f85.png" width="750" title="Snipped: July 11, 2022" />

* On the resulting **Ingest data** page, **2. Source** tab, complete the form, including:

  Prompt| Entry
  ------ | ------
  **Source type** | Select **From ADLS Gen2 container**
  **Ingestion type** | Select **One-time + Continuous**<br /><br />_Note: The “…+ Continuous” part of this UI element is misleading. Simply running through the four-step wizard will not produce a solution that continuously loads data. There is an additional set of steps that will be described after this “One-Time Ingestion” section._
  **Select source** | Click **Select container**
  **Storage...** | Select your subscription and data lake
  **Container** | Select the container where you have stored the sample file

* Review the resulting **Schema defining file** list, confirm selection of **sample1.csv**, and then click **Next: Schema**

  <img src="https://user-images.githubusercontent.com/44923999/178303001-e318752e-5869-40d3-9298-f936fd8d0cd9.png" width="750" title="Snipped: July 11, 2022" />

* Confirm the generated schema definition on the resulting **Ingest data** page, **3. Schema** tab, modify as needed and then click **Next: Start Ingestion**

  <img src="https://user-images.githubusercontent.com/44923999/178304273-bb4f553f-eaab-411f-b0b6-037775cf972a.png" width="800" title="Snipped: July 11, 2022" />

* Allow time for processing, then on the resulting **Data ingestion completed** page, confirm successful Ingestion Preparation, Ingestion and Data Preview

You have completed the One-Time Ingestion.

**BUT WAIT, you are not yet configured for continuous ingestion... do not click Close yet!**

## Continuous Ingestion

Complete the following steps:
* Scroll down on the **Data ingestion completed** page, and click on the **Event grid** link under the **Continuous ingestion** header

  <img src="https://user-images.githubusercontent.com/44923999/178315981-baca4ff4-65cb-46c1-8a7f-29ba9a7bad6e.png" width="500" title="Snipped: July 11, 2022" />

* Complete the **Data connection** pop-out form
* Click **Next: Ingest properties >** and then review default values on the **Ingest properties** tab
* Click **Next: Review + create >**, confirm configuration and then click **Create**

## Confirm Success

### One-Time Ingestion

Complete the following steps:
* Navigate to https://dataexplorer.azure.com/ and then click **Query** in the navigation pane
* Select your cluster and database, then execute the following KQL:
`sample1`

  <img src="https://user-images.githubusercontent.com/44923999/178317352-b82d1183-178e-4aa2-a4a6-7e447b561b73.png" width="800" title="Snipped: July 11, 2022" />

You should see the data from the One-Time Ingestion.

### Continuous Ingestion

Complete the following steps:
* Navigate to your Data Lake, then **Storage Browser…** in the navigation pane
* Navigate to your Data Lake Container, then **Copy** and **Paste** the sample1.csv file
  * Edit the copied sample1.csv data if you want to make things interesting
  * **Keep Both** when prompted
* Navigate to https://dataexplorer.azure.com/ and then click **Query** in the navigation pane
* Select your cluster and database, then execute the following KQL:
`sample1`

  <img src="https://user-images.githubusercontent.com/44923999/178322843-980e871a-5d93-4992-9dd5-ae8a1f9c567e.png" width="800" title="Snipped: July 11, 2022" />

You should see the old data plus new data from the copy of the sample1.csv file.

## Pro Tips

* Ingestion can be delayed by system processing... be patient (but not too patient!)
* Consider increasing the speed of ingestion (for testing) by executing the following KQL:<br />
`.alter database rchaplerded policy ingestionbatching @'{"maximumBatchingTimeSpan":"00:00:30"}'`
* You might also detect ingestion failures by executing the following KQL:<br />
`.show ingestion failures | take 10`
* Last, but not least… if you have **Auto-Stop** activated, it can negatively impact continuous ingestion

## Reference
> https://docs.microsoft.com/en-us/azure/data-explorer/ingest-data-one-click
