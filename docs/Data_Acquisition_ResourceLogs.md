## Data Acquisition... Resource Logs

Requirement statements might include:

* "Log Analytics costs more than we want to spend"
* "Log Analytics doesn't retain log data for as long as we need (30d default > 730d max does not match compliance requirements)"

### Step 1: Prepare Infrastructure
This solution requires the following resources:

* **Data Explorer** >> [Cluster](Infrastructure_DataExplorer_Cluster.md) :: [Database](Infrastructure_DataExplorer_Database.md)
* [**Event Hub**](Infrastructure_EventHub.md)
* [**Storage Account**](Infrastructure_StorageAccount.md)

### Step 2: Diagnostic Settings
_Note: These instructions can apply to any Azure resource type (via **Monitoring** >> **Diagnostic Settings**); for this exercise, however, we will capture logs from Data Explorer_

Complete the following steps:

* Ipsum

  <img src="https://user-images.githubusercontent.com/44923999/180606347-670321a8-896f-41fe-afe6-0dfdb7d87d61.png" width="800" title="Snipped: July 23, 2022" />
