## Purview
_(aka "Microsoft Purview" ... "Azure Purview" is specific to Azure Sentinel)_

### Create with Azure Portal

Complete the following steps:

* Click the menu button in the upper-left corner of the Azure Portal
* Click **+ Create a resource** in the resulting dropdown
* Use the **Search services and marketplace** textbox to search for and select "Microsoft Purview"
* Find the correct result, click the **Create** dropdown, and select "Microsoft Purview"
* Complete the **Create Microsoft Purview account** form

  <img src="https://user-images.githubusercontent.com/44923999/180605435-cbce8c37-faf8-4310-a478-7f1bb4e4d940.png" width="800" title="Snipped: July 23, 2022" />

  Prompt | Entry
  ------ | ------
  **Managed Resource Group** | This resource group will hold ancillary resources created specifically for Purview<br><br>_Note: Because you are likely to have more than one managed Resource Group, consider using a consistent naming pattern {e.g., **<UseCase>mrg-p** for Purview and **<UseCase>mrg-x** for Resource Type X}_

* Click **Review + create**, confirm configuration settings on the resulting page, and then click **Create**
