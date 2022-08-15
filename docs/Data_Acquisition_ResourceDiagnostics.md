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

First, we will configure Data Explorer to archive Query-related logs to a Storage Account.

_Note: These instructions can apply to any Azure resource type (via **Monitoring** >> **Diagnostic Settings**); for this exercise, however, we will capture logs from Data Explorer_

Complete the following steps:

* Use the Azure Portal to navigate to your Data Explorer Cluster
* Select **Diagnostic Settings** in the **Monitoring** group of the left-hand navigation

  <img src="https://user-images.githubusercontent.com/44923999/184675094-161c4fad-a134-4960-bbff-ee9600f443ae.png" width="800" title="Snipped: August 15, 2022" />

* Click **+ Add diagnostic setting**

  <img src="https://user-images.githubusercontent.com/44923999/184676669-a78ff2ba-18d1-4512-adee-9f9461104f21.png" width="800" title="Snipped: August 15, 2022" />

* On the resulting **Diagnostic setting** page, complete the form, including:

  Prompt| Entry
  ------ | ------
  **Logs** >> **Categories** | Check **Query** and then confirm the **Retention (days)** default of 0 {i.e., retain archived data forever}
  **Destination Details** | Check **Archive to a storage account** and then populate **Subscription** and **Storage account**
