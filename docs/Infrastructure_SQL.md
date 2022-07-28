## SQL
_(aka “Azure SQL”, “Azure SQL Database”, “Azure SQL Database Server”, “Azure SQL Server”)_

### Create with Azure Portal

#### Server

* Click the menu button in the upper-left corner of the Azure Portal
* Click **+ Create a resource** in the resulting dropdown
* Use the **Search services and marketplace** textbox to search for "azure sql" and select the appropriate dropdown value
* Find the correct result, select the **Create** dropdown, and then click the appropriate dropdown value

  <img src="https://user-images.githubusercontent.com/44923999/180610756-3263f92c-a42d-4d2e-b841-2a5d32432237.png" width="600" title="Snipped: July 23, 2022" />

* On the **Select SQL deployment option** > **How do you plan to use the service?** page, select **Database server** from the **Resource type** dropdown and then click **Create**<br>
 
  <img src="https://user-images.githubusercontent.com/44923999/180610666-c30e773c-7184-40c6-8669-84deea5252ed.png" width="600" title="Snipped: July 23, 2022" />

* Complete the form on the **Create SQL Database Server** page, **Basics** tab, including:

    Prompt | Entry
    :----- | :-----
    **Authentication Method** | Select **Use only Azure Active Directory (Azure AD) authentication**<br><br>_Note: I do not recommend using SQL authentication, though it may be appropriate for some environments and use cases_
    **Set Azure AD Admin** | Click the **Set admin** link and on the resulting pop-out, search for and select the appropriate user

  _Note: No additional configuration is required but consider review of the default values on the remaining tabs._

* Navigate to the **Networking** tab and set **Allow Azure services** to Yes (if appropriate for your environment and use case)

* Click **Review + create**, confirm configuration settings on the resulting page, and then click **Create**

#### Database

* On the **Overview** page of the new Azure SQL Database Server, click **+ Create database**

  <img src="https://user-images.githubusercontent.com/44923999/180611032-07d6c068-5a13-462b-a88a-7e3b3c465e83.png" width="600" title="Snipped: July 23, 2022" />

* Complete the form on the **Create SQL Database** page, **Basics** tab

  _Note: No additional configuration is required but consider review of the default values on the remaining tabs._
  
* Click **Review + create**, confirm configuration settings on the resulting page, and then click **Create**

#### Sample Data
You can find sample data (and instructions for use) at https://docs.microsoft.com/en-us/sql/samples/adventureworks-install-configure?view=sql-server-ver16&tabs=ssms
