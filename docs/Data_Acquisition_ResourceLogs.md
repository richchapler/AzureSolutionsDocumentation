## Data Acquisition... Resource Logs

Requirement statements might include:

* "Log Analytics costs more than we want to spend"
* "Log Analytics doesn't retain log data for as long as we need (30d default > 730d max does not match compliance requirements)"

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* **Data Explorer** >> [Cluster](Infrastructure_DataExplorer_Cluster.md) :: [Database](Infrastructure_DataExplorer_Database.md)
* [**Event Hub**](Infrastructure_EventHub.md)
* [**Storage Account**](Infrastructure_StorageAccount.md)

### Step 2: Archive to Storage Account
_Note: These instructions can apply to any Azure resource type (via **Monitoring** >> **Diagnostic Settings**); for this exercise, however, we will capture logs from Data Explorer_

Complete the following steps:

* Use the Azure Portal to navigate to your Data Explorer Cluster
* Select **Diagnostic Settings** in the **Monitoring** group of the left-hand navigation

  <img src="https://user-images.githubusercontent.com/44923999/184675094-161c4fad-a134-4960-bbff-ee9600f443ae.png" width="800" title="Snipped: August 15, 2022" />

* Click **+ Add diagnostic setting**

  <img src="https://user-images.githubusercontent.com/44923999/184676139-3abbbecd-1f54-454c-bdd0-fe1f354483bf.png" width="800" title="Snipped: August 15, 2022" />

* Click **+ Add diagnostic setting**
