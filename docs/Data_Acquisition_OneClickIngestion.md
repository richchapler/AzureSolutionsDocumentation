## "One-Click" Ingestion

![image](https://user-images.githubusercontent.com/44923999/185980410-353cda9e-d0a8-405c-ab1c-409df61e46c4.png)

This use case considers requirement statements like:

* "We have devices running an app 24x7, around the world, continuously dropping files into the Data Lake"
* “We want to automatically ingest data files as they appear”

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* **Data Explorer** >> [Cluster](Infrastructure_DataExplorer_Cluster.md) :: [Database](Infrastructure_DataExplorer_Database.md)
* [**Data Lake**](Infrastructure_DataLake.md) (with [container](Infrastructure_DataLake_Container.md) and downloaded [sample data](https://filesamples.com/formats/csv))

### Step 2: One-Time Ingestion

Complete the following steps:

* Navigate to https://dataexplorer.azure.com/ and then click **Data** in the navigation pane
* Click **Ingest Data** on the resulting **Data Management** page

  <img src="https://user-images.githubusercontent.com/44923999/178302280-6dad3275-e252-4368-8f69-fa5b098dd1bb.png" width="800" title="Snipped: July 11, 2022" />

* On the resulting **Ingest data** page, **1. Destination** tab, select your Data Explorer cluster and database, then select the **New table** radio button, enter the name **sample1** (to match our downloaded sample file) and finally, click **Next: Source**

  <img src="https://user-images.githubusercontent.com/44923999/178302589-c0843482-0d4d-4626-a224-28cac8771f85.png" width="800" title="Snipped: July 11, 2022" />

* On the resulting **Ingest data** page, **2. Source** tab, complete the form, including:

  Prompt| Entry
  ------ | ------
  **Source type** | Select **From ADLS Gen2 container**
  **Ingestion type** | Select **One-time + Continuous**<br /><br />_Note: The “…+ Continuous” part of this UI element is misleading. Simply running through the four-step wizard will not produce a solution that continuously loads data. There is an additional set of steps that will be described after this “One-Time Ingestion” section._
  **Select source** | Click **Select container**
  **Storage...** | Select your subscription and data lake
  **Container** | Select the container where you have stored the sample file

* Review the resulting **Schema defining file** list, confirm selection of **sample1.csv**, and then click **Next: Schema**

  <img src="https://user-images.githubusercontent.com/44923999/178303001-e318752e-5869-40d3-9298-f936fd8d0cd9.png" width="800" title="Snipped: July 11, 2022" />

* Confirm the generated schema definition on the resulting **Ingest data** page, **3. Schema** tab, modify as needed and then click **Next: Start Ingestion**

  <img src="https://user-images.githubusercontent.com/44923999/178304273-bb4f553f-eaab-411f-b0b6-037775cf972a.png" width="800" title="Snipped: July 11, 2022" />

* Allow time for processing, then on the resulting **Data ingestion completed** page, confirm successful Ingestion Preparation, Ingestion and Data Preview

#### Confirm Success

Complete the following steps:
* Navigate to https://dataexplorer.azure.com/ and then click **Query** in the navigation pane
* Select your cluster and database, then execute the following KQL:
`sample1`

  <img src="https://user-images.githubusercontent.com/44923999/178317352-b82d1183-178e-4aa2-a4a6-7e447b561b73.png" width="800" title="Snipped: July 11, 2022" />

You should see the data from the One-Time Ingestion.

You have completed the One-Time Ingestion... but, **WAIT, you are not yet configured for continuous ingestion... do not click Close yet!**

### Step 3: Continuous Ingestion

Complete the following steps:
* Scroll down on the **Data ingestion completed** page, and click on the **Event grid** link under the **Continuous ingestion** header
<br>  _Note: you can also navigate to your Data Explorer Database, Data Connections and then click **+ Add data connection**_

  <img src="https://user-images.githubusercontent.com/44923999/182691946-c369f08b-4e6f-46be-863d-7118b99d6492.png" width="800" title="Snipped: August 3, 2022" />

* Complete the form on the **Create data connection** pop-out, **Basics** tab<br>
  _Note: Continuous Ingestion **only** supports the use of System and User-Assigned Managed Identity... Service Principal and SAS Token are not available options_
* Click **Next: Ingest properties >**
  
  <img src="https://user-images.githubusercontent.com/44923999/182691406-1995fd5d-0a15-411a-bea2-9e893193a5fd.png" width="800" title="Snipped: August 3, 2022" />

* Complete the form on the **Create data connection** pop-out, **Ingest properties** tab<br>
  _Note: If the Data Format of your source is JSON, consider using MULTILINE JSON instead (to pre-emptively avoid data quality-related ingestion issues)_
  
* Click **Next: Review + create >**, confirm configuration and then click **Create**

#### Confirm Success

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

### Bonus Content

#### Pro Tips

* Consider increasing the speed of ingestion from the default of every five minutes by executing the following KQL:<br />
`.alter database rchaplerded policy ingestionbatching @'{"maximumBatchingTimeSpan":"00:00:30"}'`
* You might also detect ingestion failures by executing the following KQL:<br />
`.show ingestion failures | take 10`
* Last, but not least… if you have **Auto-Stop** activated, it can negatively impact continuous ingestion

### Reference
> https://docs.microsoft.com/en-us/azure/data-explorer/ingest-data-one-click
