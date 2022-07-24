## Discover Data

Requirement statements might include:

* **Search** … “does the data that I need exist?”
* **Location** … “how do I connect to Data Product X?”
* **Related** … “are there related data products that I might find valuable?”
* **Contacts** … “who are the owners and experts for Data Product X?”

### Step 1: Prepare Resources

This solution requires the following resources:

* [Purview](PrepareResources_Purview.md)
* [SQL](PrepareResources_SQL.md) (with sample data)

#### Register Source

* Open the **Purview Governance Portal**
* Click the **Data Map** icon on the navigation pane
* Click **Register** on the resulting page

  <img src="https://user-images.githubusercontent.com/44923999/180648889-285b11ec-4bec-427b-afb4-931a1e0afe45.png" width="800" title="Snipped: July 24, 2022" />

* On the resulting **Register sources** pop-out, search for and select **Azure SQL Database**
* Click **Continue**

  <img src="https://user-images.githubusercontent.com/44923999/180649034-fc466af5-9c13-4a47-b919-e4a2827b31fb.png" width="800" title="Snipped: July 24, 2022" />

* Complete the **Register sources (Azure SQL Database)** form
* Click **Register**

#### Scan Data

_Note: Purview requires **Storage Blob Data Reader** permissions to scan a Data Lake for blobs_

  <img src="https://user-images.githubusercontent.com/44923999/180649216-a6ad1216-d6cc-4355-b248-71723b36a7a4.png" width="800" title="Snipped: July 24, 2022" />

* Find your newly-registered source on the **Map view** and click the **New scan** icon

  <img src="https://user-images.githubusercontent.com/44923999/180649349-56ae89fc-a196-412a-8114-15969b4d504a.png" width="800" title="Snipped: July 24, 2022" />

