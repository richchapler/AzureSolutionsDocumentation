## [Data Explorer](Infrastructure_DataExplorer.md) >> **Data Connection**

![image](https://user-images.githubusercontent.com/44923999/185980410-353cda9e-d0a8-405c-ab1c-409df61e46c4.png)

### Create with ARM Template

### Step #1: Prepare Infrastructure
In this step, we will prepare the Azure resources necessary to complete the exercise.

This solution requires:

* Data Explorer [**Cluster**](Infrastructure_DataExplorer_Cluster.md) and [**Database**](Infrastructure_DataExplorer_Database.md), with a [**Table**](Infrastructure_DataExplorer_Table.md) and [**Ingestion Mapping**](Infrastructure_DataExplorer_IngestionMapping.md)
* Event Grid, [**Event Subscription**](Infrastructure_EventGrid_EventSubscription.md)... dependent on Event Hub and Storage Account
* [**Event Hub**](Infrastructure_EventHub.md)
* [**Storage Account**](Infrastructure_StorageAccount.md) with a container

### Step 2: Deploy Custom Template
In this step, we will deploy a Data Connection on the Data Explorer Database using an ARM Template.

Complete the following steps:

* Click the menu button in the upper-left corner of the Azure Portal and select "**Deploy a custom template**" from the resulting dropdown

  <img src="https://user-images.githubusercontent.com/44923999/182941824-1675b487-e60c-44ba-8a94-0eeaa8ee12af.png" width="800" title="Snipped: August 4, 2022" />

* Click "**Build your own template in the editor**"

  <img src="https://user-images.githubusercontent.com/44923999/182942508-5b378150-abc2-47de-924d-4a4720326fba.png" width="800" title="Snipped: August 4, 2022" />

* Select your data connection type, copy-paste into the "**Edit template**" window, and then modify the selected JSON:

#### Event Grid (Blob Storage)

  ```
  {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
          "DataExplorer_Cluster_Name": { "type": "string", "defaultValue": "[concat(resourceGroup().name,'dec')]" },
          "DataExplorer_Database_Name": { "type": "string", "defaultValue": "[concat(resourceGroup().name,'ded')]" },
          "DataExplorer_DataConnection_Name": { "type": "string", "defaultValue": "[concat(resourceGroup().name,'dedc')]" },
          "DataLake_Name": { "type": "string", "defaultValue": "[concat(resourceGroup().name,'dls')]" },
          "EventHubNamespace_Name": { "type": "string", "defaultValue": "[concat(resourceGroup().name,'ehn')]" },
          "EventHub_Name": { "type": "string", "defaultValue": "[concat(resourceGroup().name,'eh')]" }
      },
      "resources": [{
              "type": "Microsoft.Kusto/Clusters/Databases/DataConnections",
              "apiVersion": "2019-09-07",
              "name": "[concat(parameters('DataExplorer_Cluster_Name'),'/', parameters('DataExplorer_Database_Name'), '/', parameters('DataExplorer_DataConnection_Name'))]",
              "location": "[resourceGroup().location]",
              "tags": { "Environment": "PROD", "CostCenter":"123456" },
              "kind": "EventGrid",
              "properties": {
                  "storageAccountResourceId": "[resourceId(subscription().subscriptionId, resourceGroup().name, 'Microsoft.Storage/storageAccounts', parameters('DataLake_Name'))]",
                  "eventHubResourceId": "[resourceId(subscription().subscriptionId, resourceGroup().name, 'Microsoft.EventHub/namespaces/eventhubs', parameters('EventHubNamespace_Name'), parameters('EventHub_Name'))]",
                  "consumerGroup": "$Default",
                  "tableName": "['t']",
                  "mappingRuleName": "['t_mapping']",
                  "dataFormat": "['json']",
                  "databaseRouting": "Single"
              }
          }
      ]
  }
  ```

* Click **Save**
* Confirm values on the resulting "**Custom deployment**" >> "**Deploy from a custom template**" page, **Basics** tab
* Click "**Review + create**", confirm configuration settings on the resulting page, and then click **Create**

### Step 3: Specify Container
In this step, we will update the deployed Data Connection to specify a container.

Complete the following steps:

* Navigate to your Data Explorer Database and then click "**Data Connections**" in the **Settings** area of the left-hand navigation
* On the resulting page, click on the newly deployed Data Connection

  <img src="https://user-images.githubusercontent.com/44923999/185758730-15af9b22-3920-4622-bc15-a74e8413ce3f.png" width="800" title="Snipped: August 20, 2022" />

* Select "**System-Assigned**" from the "**Assign managed identity**" drop-down
* Expand "**Filter settings**" on the resulting pop-out form and update values for **Prefix**, **Suffix**, and "**Case Sensitive**" as appropriate
* Click **Update**
