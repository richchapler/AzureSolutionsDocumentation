## Data Acquisition... from Resource Diagnostics

![image](https://user-images.githubusercontent.com/44923999/185979002-2b54427a-dadf-4076-a682-7c3cac86e2ab.png)

This use case considers requirement statements like:

* "Log Analytics costs more than we want to spend"
* "Log Analytics doesn't retain log data for as long as we need (30d default > 730d max does not match compliance requirements)"
* "Is there a way to pull the log data that we care about into Data Explorer?"

### Prepare Infrastructure
This solution requires the following resources:

* **Data Explorer** >> [Cluster](Infrastructure_DataExplorer_Cluster.md) :: [Database](Infrastructure_DataExplorer_Database.md)
* [**Event Hub**](Infrastructure_EventHub.md)
* [**Storage Account**](Infrastructure_StorageAccount.md)

#### Grant Access (Data Explorer > Event Hub)
Data Explorer requires special permissions to interact with Event Hub.

Complete the following steps:

* Navigate to your Event Hub Namespace, then "**Access Control (IAM)**" in the navigation

  <img src="https://user-images.githubusercontent.com/44923999/184709205-5f6e8ad5-92fe-4577-b759-8e3b2b14dca4.png" width="800" title="Snipped: August 15, 2022" />

* On the resulting screen, click **+ Add** and select "**Add role assignment**" from the resulting dropdown menu

  <img src="https://user-images.githubusercontent.com/44923999/184709389-f74fc089-9169-495b-b03c-94a78eaa34b6.png" width="800" title="Snipped: August 15, 2022" />

* On the resulting "**Add role assignment**" screen, search for and select "**Azure Event Hub Data Receiver**", then click **Next**

  <img src="https://user-images.githubusercontent.com/44923999/184709516-1e43e812-97d9-41ed-b1e9-657e19f974c6.png" width="800" title="Snipped: August 15, 2022" />

* On the resulting "**Add role assignment**" screen, select the "**Managed identity**" radio button, and then click "**+ Select members**"
* On the resulting pop-out, search for and select the managed identity of your Data Explorer Cluster, then click **Select**
* Back on the "**Add role assignment**" screen, click "**Review + assign**"

  <img src="https://user-images.githubusercontent.com/44923999/184709935-d728f0d2-606e-4c2e-aa45-c73b8cf5d6fe.png" width="800" title="Snipped: August 15, 2022" />

* On the resulting "**Add role assignment**" screen, confirm configuration, and then click "**Review + assign**"

### Step 1: Archive to Storage Account
First, we will configure Data Explorer to archive Command logs to a Storage Account (both as demonstration and to support easy ADX ingestion).

_Note: These instructions can apply to any Azure resource type (via **Monitoring** >> "**Diagnostic Settings**")_

<br>Complete the following steps:

* Use the Azure Portal to navigate to your Data Explorer Cluster
* Select "**Diagnostic Settings**" in the **Monitoring** group of the left-hand navigation

  <img src="https://user-images.githubusercontent.com/44923999/184675094-161c4fad-a134-4960-bbff-ee9600f443ae.png" width="800" title="Snipped: August 15, 2022" />

* Click "**+ Add diagnostic setting**"

  <img src="https://user-images.githubusercontent.com/44923999/184688702-e5098cbb-c14a-4124-9437-60a585243403.png" width="800" title="Snipped: August 15, 2022" />

* On the resulting "**Diagnostic setting**" page, complete the form, including:

  Prompt| Entry
  ------ | ------
  **Logs** >> **Categories** | Check **Command** and then confirm the **Retention (days)** default of 0 {i.e., retain archived data forever}
  "**Destination Details**" | Check "**Archive to a storage account**" and then populate **Subscription** and "**Storage account**"

* Click **Save**

#### Confirm Success

Complete the following steps:

* Use the Azure Portal to navigate to your Data Explorer Cluster
* Select **Query** in the left-hand navigation

  <img src="https://user-images.githubusercontent.com/44923999/184689010-d9ce3dcd-eeaf-440f-84cb-daac77dcb28b.png" width="800" title="Snipped: August 15, 2022" />

* Run the following KQL

  ```
  .show tables
  ```

* Confirm expected resultset (in my environment, there were "No Rows to Show" because no tables have been created yet)
* Use the Azure Portal to navigate to your Storage Account
* Select **Containers** in the "**Data storage**" group of the left-hand navigation

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
* Select "**Diagnostic Settings**" in the **Monitoring** group of the left-hand navigation
* Click "**+ Add diagnostic setting**"

  <img src="https://user-images.githubusercontent.com/44923999/184693081-586578cc-b5be-4906-9df8-f7ca6abd97cd.png" width="800" title="Snipped: August 15, 2022" />

* On the resulting "**Diagnostic setting**" page, complete the form, including:

  Prompt| Entry
  ------ | ------
  **Logs** >> **Categories** | Check **Command**
  "**Destination Details**" | Check "**Stream to an event hub**" and then populate **Subscription**, "**Event hub namespace**", and "**Event hub**"

* Click **Save**

  <img src="https://user-images.githubusercontent.com/44923999/184694922-64da9047-918f-4518-9cb5-7a18f2883674.png" width="800" title="Snipped: August 15, 2022" />

  <br>_Note: The Storage Account Diagnostic Settings were useful for demonstration and helped with initial Data Explorer data ingestion, but consider deleting it if you do not need it_

### Step 4: Configure Continuous Ingestion
Finally, we will configure Data Explorer to continuously ingest data from the Event Hub.

<br>Complete the following steps:

* Use the Azure Portal to navigate to your Data Explorer Database
* Select "**Data connections**" in the **Settings** group of the left-hand navigation

  <img src="https://user-images.githubusercontent.com/44923999/184698831-37274a27-f72e-4aba-a044-08033f5ddf87.png" width="800" title="Snipped: August 15, 2022" />

* Click "**+ Add data connection**" and select "**Event Hub**" from the resulting dropdown menu

<img src="https://user-images.githubusercontent.com/44923999/184710490-76c51882-8613-460d-a3b3-ffb0b5808f8b.png" width="800" title="Snipped: August 15, 2022" />

* Complete the form on the **Create data connection** pop-out and then click **Create**<br><br>

  _Notes:_
  * _Continuous Ingestion **only** supports the use of System and User-Assigned Managed Identity... Service Principal and SAS Token are not available options_
  * _If the Data Format of your source is JSON, consider using MULTILINE JSON instead (to pre-emptively avoid data quality-related ingestion issues)_

#### Confirm Success

Complete the following steps:

* Use the Azure Portal to navigate to your Data Explorer Database
* Select "**Query**" in the **Overview** group of the left-hand navigation
* Run the following KQL to remove any previous records in table **t**
  ```
  .clear table t data
  ```
  
* Run the following KQL to confirm that are zero records in table **t**

  ```
  t | count
  ```

* Run the following KQL to create new log data for ingestion

  ```
  .show tables
  ```

* Run the following KQL to confirm that new log entries have been created, were routed through Event Hub, and ingested into Data Explorer

  ```
  t | count
  ```

  <img src="https://user-images.githubusercontent.com/44923999/184713480-e97749b0-a26e-4c16-b4ff-508e9b3a1cd5.png" width="800" title="Snipped: August 15, 2022" />
