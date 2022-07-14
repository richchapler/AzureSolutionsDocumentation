## Data Lake
_(aka "ADLS", "Azure Data Lake Storage", "Storage Account")_

## Create with Azure Portal

* Click the menu button in the upper-left corner of the Azure Portal
* Click **+ Create a resource** in the resulting dropdown
* Use the **Search services and marketplace** textbox to search for "storage account" and select the appropriate dropdown value
* Find the correct result, select the **Create** dropdown, and then click the appropriate dropdown value
* Complete the **Create a storage account** form, **Basics** tab
 
  <img src="https://user-images.githubusercontent.com/44923999/178049387-11585534-df7f-430e-9d71-e8414692e66e.png" width="600" title="Snipped: July 8, 2022" />

* On the **Create a Storage Account** form, **Advanced** tab, check **Enable Hierarchical Namespace** in the **Data Lake Storage Gen2** grouping

  <img src="https://user-images.githubusercontent.com/44923999/178049285-9539e65a-4cdb-4b70-a4f0-593cf3c10d46.png" width="600" title="Snipped: July 8, 2022" />

  _Note: No additional configuration is required but consider review of the default values on the remaining tabs._

* Click **Review + create**, review configuration, and then click **Create**

_Note: When a Storage Account is configured for Data Lake Storage, you will see **Hierarchical Namespace**: Enabled on the Overview page_

## See Also
> [Data Lake Container](PrepareResources_DataLake_Container.md)

> [Sample Data](PrepareResources_DataLake_SampleData.md)

> [Storage Explorer](PrepareResources_StorageAccount.md)
