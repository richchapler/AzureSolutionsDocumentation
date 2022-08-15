## Data Acquisition... Resource Diagnostics

Requirement statements might include:

* "Log Analytics costs more than we want to spend"
* "Log Analytics doesn't retain log data for as long as we need (30d default > 730d max does not match compliance requirements)"

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* **Data Explorer** >> [Cluster](Infrastructure_DataExplorer_Cluster.md) :: [Database](Infrastructure_DataExplorer_Database.md)
* [**Event Hub**](Infrastructure_EventHub.md)
* [**Storage Account**](Infrastructure_StorageAccount.md)

### Step 2: Archive to Storage Account
First, we will configure Data Explorer to archive Command logs to a Storage Account (both as demonstration and to support easy ADX ingestion).

_Note: These instructions can apply to any Azure resource type (via **Monitoring** >> **Diagnostic Settings**)_

<br>Complete the following steps:

* Use the Azure Portal to navigate to your Data Explorer Cluster
* Select **Diagnostic Settings** in the **Monitoring** group of the left-hand navigation

  <img src="https://user-images.githubusercontent.com/44923999/184675094-161c4fad-a134-4960-bbff-ee9600f443ae.png" width="800" title="Snipped: August 15, 2022" />

* Click **+ Add diagnostic setting**

  <img src="https://user-images.githubusercontent.com/44923999/184688702-e5098cbb-c14a-4124-9437-60a585243403.png" width="800" title="Snipped: August 15, 2022" />

* On the resulting **Diagnostic setting** page, complete the form, including:

  Prompt| Entry
  ------ | ------
  **Logs** >> **Categories** | Check **Command** and then confirm the **Retention (days)** default of 0 {i.e., retain archived data forever}
  **Destination Details** | Check **Archive to a storage account** and then populate **Subscription** and **Storage account**

* Click **Save**

#### Confirm Success

Complete the following steps:

* Use the Azure Portal to navigate to your Data Explorer Cluster
* Select **Query** in the left-hand navigation

  <img src="https://user-images.githubusercontent.com/44923999/184689010-d9ce3dcd-eeaf-440f-84cb-daac77dcb28b.png" width="800" title="Snipped: August 15, 2022" />

* Paste the following KQL and then click Run

  ```
  .show tables
  ```

* Confirm expected resultset (in my environment, there were "No Rows to Show" because no tables have been created yet)
* Use the Azure Portal to navigate to your Storage Account
* Select **Containers** in the **Data storage** group of the left-hand navigation

  <img src="https://user-images.githubusercontent.com/44923999/184689998-5b0d8303-b6d3-4800-9785-2af23c645d93.png" width="800" title="Snipped: August 15, 2022" />

* Confirm creation / population of container "**insights-log-command**" and sub-folders

  <img src="https://user-images.githubusercontent.com/44923999/184690314-08d54000-38eb-48a4-b952-78d909bce6c3.png" width="800" title="Snipped: August 15, 2022" />

* When you get to the bottom of the folder hierarchy, **Download** the ".json" file

  _Note: You could use this JSON file to divine the schema information necessary to create the Data Explorer [Table](Infrastructure_DataExplorer_Table.md) and [Ingestion Mapping](Infrastructure_DataExplorer_IngestionMapping.md) or you can use the [One-Click Ingestion](Data_OneClickIngestion.md) wizard... I recommend the latter_

  <img src="https://user-images.githubusercontent.com/44923999/184691673-dfbf8d4c-fc62-4202-a5cb-805df64002e8.png" width="800" title="Snipped: August 15, 2022" />

* Whichever method you employ, you should end the exercise with a table and ingestion mapping

  <img src="https://user-images.githubusercontent.com/44923999/184692023-143b8339-7192-486b-84a9-78692706af02.png" width="800" title="Snipped: August 15, 2022" />

### Step 3: Stream to an Event Hub
Next, we will configure Data Explorer to stream Command logs to an Event Hub.

<br>Complete the following steps:

* Use the Azure Portal to navigate to your Data Explorer Cluster
* Select **Diagnostic Settings** in the **Monitoring** group of the left-hand navigation
* Click **+ Add diagnostic setting**

  <img src="https://user-images.githubusercontent.com/44923999/184693081-586578cc-b5be-4906-9df8-f7ca6abd97cd.png" width="800" title="Snipped: August 15, 2022" />

* On the resulting **Diagnostic setting** page, complete the form, including:

  Prompt| Entry
  ------ | ------
  **Logs** >> **Categories** | Check **Command**
  **Destination Details** | Check **Stream to an event hub** and then populate **Subscription**, **Event hub namespace**, and **Event hub**

* Click **Save**

  <img src="https://user-images.githubusercontent.com/44923999/184694922-64da9047-918f-4518-9cb5-7a18f2883674.png" width="800" title="Snipped: August 15, 2022" />

<br>  _Note: The Storage Account Diagnostic Settings were useful for demonstration and helped with initial Data Explorer data ingestion, but consider deleting it if you do not need it_

### Step 4: Configure Continuous Ingestion
Finally, we will configure Data Explorer to continuously ingest data from the Event Hub.

<br>Complete the following steps:
