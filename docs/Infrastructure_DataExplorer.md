## Data Explorer

### Create with Azure Portal

#### Cluster
_(aka “ADX,” “Azure Data Explorer,” “Kusto”... also, easy confused with Data Explorer Pool functionality in Synapse)_

Complete the following steps:

* Click the menu button in the upper-left corner of the Azure Portal
* Click **+ Create a resource** in the resulting dropdown
* Use the **Search services and marketplace** textbox to search for and select "Azure Data Explorer"
* Find the correct result, select the **Create** dropdown, and then click the appropriate dropdown value
* Complete the **Create an Azure Data Explorer Cluster** form

  <img src="https://user-images.githubusercontent.com/44923999/178290844-95e498e4-f8a4-4b89-8fcf-eb9f035c312c.png" width="800" title="Snipped: July 11, 2022" />

  _Note: No additional configuration is required but consider review of the default values on the remaining tabs._

* Click **Review + create**, confirm configuration settings on the resulting page, and then click **Create**

#### Database
_Note: These instructions can apply to either Data Explorer Clusters or Synapse Data Explorer Pools_

Complete the following steps:

* Navigate to your instance of Data Explorer and click **+ Add database** on the main page
* Complete the resulting **Azure Data Explorer Database** pop-out form and then click **Create**
  
  <img src="https://user-images.githubusercontent.com/44923999/178294501-96d06134-e93c-4bd6-ba67-414c6be5841c.png" width="800" title="Snipped: July 8, 2022" />

#### Table

Complete the following steps:

* Navigate to your instance of Data Explorer and then select Query from the navigation

  <img src="https://user-images.githubusercontent.com/44923999/182936394-a49e10dc-9a28-4e4e-834a-ff432e28cd3e.png" width="800" title="Snipped: August 4, 2022" />

* Modify and **Run** the following KQL:

  ```
  .create table t ( d:dynamic )
  ```

#### Ingestion Mapping

Complete the following steps:

* Navigate to your instance of Data Explorer and then select Query from the navigation

  <img src="https://user-images.githubusercontent.com/44923999/182936856-ef5c140b-2cc1-45a8-ba15-b1a89cca71a7.png" width="800" title="Snipped: August 4, 2022" />

* Modify and **Run** the following KQL:

  ```
  .create table t ingestion json mapping 't_mapping' '[{"column":"d","path":"$","datatype":"dynamic"}]'
  ```
