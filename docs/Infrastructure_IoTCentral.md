## IoT Central
_(aka "IoT Central Application")_

![image](https://user-images.githubusercontent.com/44923999/185975852-f21da095-6d6d-4259-86d8-6b199c9e3295.png)

### Create with Azure Portal

* Click the menu button in the upper-left corner of the Azure Portal
* Click **+ Create a resource** in the resulting dropdown
* Use the "Search services and marketplace" textbox to search for and select "IoT Central Application"
* Find the correct result, click the Create dropdown, and select "IoT Central Application"
* Complete the **IoT Central Application** form

  <img src="https://user-images.githubusercontent.com/44923999/187219156-29f0553f-57d4-4414-b88d-ac42ea88f6ec.png" width="800" title="Snipped: August 29, 2022" />

  Prompt | Entry
  ------ | ------
  **Managed Resource Group** | This resource group will hold ancillary resources created specifically for Synapse<br><br>_Note: Because you are likely to have more than one managed Resource Group, consider using a consistent naming pattern {e.g., **<UseCase>mrg-s** for Synapse and **<UseCase>mrg-x** for Resource Type X}_
  **Account Name** | Select an existing data lake or click the **Create new** link to create a new data lake
  **File System Name** | Select an existing data lake file system or click the **Create new** link to create a new data lake file system

* Click the **Next: Security >** button and then select the **Use only Azure Active Directory (Azure AD) authentication** radio button
  _Note: Though local authentication may be appropriate to some use cases, I do not typically recommend it_

* Click **Review + create**, confirm configuration settings on the resulting page, and then click **Create**
