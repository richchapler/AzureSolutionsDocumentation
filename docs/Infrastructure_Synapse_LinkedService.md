## Synapse >> Linked Service

### Create with Azure Portal

* Navigate to **Synapse Studio**
* Click the **Manage** navigation icon
* Select **Linked services** from the **External connections** grouping in the resulting navigation
* Click **+ New**
* Complete the **New linked service…** pop-out form using the instructions below (for the appropriate Linked Service Type)

  #### Azure Data Lake Storage Gen2

  <img src="https://user-images.githubusercontent.com/44923999/179089406-89004791-8bc1-4dc3-ab4d-33f624238e65.png" width="800" title="Snipped: July 14, 2022" />
  
  #### REST

  <img src="https://user-images.githubusercontent.com/44923999/179222060-d5ec7a7b-b1fa-40c3-9d78-fbb6cf6c7de4.png" width="800" title="Snipped: July 15, 2022" />
  
  Prompt | Entry
  ------ | ------
  **Connect via…** | Confirm default selection, **AutoResolveIntegrationRuntime**
  **Base URL** | ...for Cost Management, modify and enter:<br>`https://management.azure.com/subscriptions/{Subscription Id}/providers/Microsoft.CostManagement/query?api-version=2021-10-01`<br><br>...for Log Analytics, modify and enter:<br>`https://api.loganalytics.io/v1/workspaces/{LogAnalyticsWorkspaceId}/query`<br><br>...for Purview, modify and enter:<br>`https://{Purview Instance Name}.purview.azure.com/scan/datasources/{Purview Data Source Name}?api-version=2022-02-01-preview`
  **Authentication Type** | Select **Anonymous**

* Click **Test connection** and confirm successful connection
* Click **Create**
