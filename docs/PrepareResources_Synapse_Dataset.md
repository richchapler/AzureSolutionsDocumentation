## Dataset
_(aka "Integration Dataset")_

### Create with Azure Portal

* Navigate to **Synapse Studio**
* Click the **Data** navigation icon
* Click **+** and select **Integration dataset** from the resulting dropdown menu
* Complete the **New integration dataset** pop-out form using the instructions below (for the appropriate Dataset Type)

  #### Azure Data Lake Storage Gen2

  <img src="https://user-images.githubusercontent.com/44923999/179225093-b4edee7b-1e39-4e6c-b710-7d07fe9a29ff.png" width="800" title="Snipped: July 15, 2022" />

  #### REST

  <img src="https://user-images.githubusercontent.com/44923999/179225093-b4edee7b-1e39-4e6c-b710-7d07fe9a29ff.png" width="800" title="Snipped: July 15, 2022" />
  
  Prompt | Entry
  ------ | ------
  **Connect via…** | Confirm default selection, **AutoResolveIntegrationRuntime**
  **Base URL** | ...for Cost Management, modify and enter:<br>`https://management.azure.com/subscriptions/{Subscription Id}/providers/Microsoft.CostManagement/query?api-version=2021-10-01`<br><br>...for Log Analytics, modify and enter:<br>`https://api.loganalytics.io/v1/workspaces/{LogAnalyticsWorkspaceId}/query`<br><br>...for Purview, modify and enter:<br>`https://{Purview Instance Name}.purview.azure.com/scan/datasources/{Purview Data Source Name}?api-version=2022-02-01-preview`
  **Authentication Type** | Select **Anonymous**

* Click **Test connection** and confirm successful connection
* Click **Create**
