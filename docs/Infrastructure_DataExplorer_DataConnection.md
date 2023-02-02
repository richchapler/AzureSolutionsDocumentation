## [Data Explorer](Infrastructure_DataExplorer.md) >> **Data Connection**

![image](https://user-images.githubusercontent.com/44923999/185980410-353cda9e-d0a8-405c-ab1c-409df61e46c4.png)

### Create with Azure Data Explorer Web UI

#### Step 1: Prepare Infrastructure
In this step, we will prepare the Azure resources necessary to complete the exercise. This solution requires:

* [**Data Explorer**](https://learn.microsoft.com/en-us/azure/data-explorer/) [Cluster](https://learn.microsoft.com/en-us/azure/data-explorer/create-cluster-database-portal) and [Database](https://learn.microsoft.com/en-us/azure/data-explorer/create-cluster-database-portal)

### Step 2: Ingest Data
In this step, we will use the "**Ingest Data**" wizard to create a Data Connection.

* Browse to https://dataexplorer.azure.com/oneclick

  <img src="https://user-images.githubusercontent.com/44923999/187456543-fe895720-7296-446c-9237-a691c2aeef91.png" width="800" title="Snipped: August 30, 2022" />

* Click on "**Ingest Data**"

  <img src="https://user-images.githubusercontent.com/44923999/187457253-05dc0963-82b0-451a-9c02-e3b6aba5b998.png" width="800" title="Snipped: August 30, 2022" />
  
* Complete the "**Ingest data**" > **Destination** form and then click "**Next: Source**"

  <img src="https://user-images.githubusercontent.com/44923999/187460352-89f9a327-f32a-4f4d-bdc8-9883edfb7c3e.png" width="800" title="Snipped: August 30, 2022" />

* Complete the "**Ingest data**" > **Source** form and then click "**Next: Schema**"

  <img src="https://user-images.githubusercontent.com/44923999/187460699-e6bfa46e-6dba-4a2e-b6e3-528685ec1c93.png" width="800" title="Snipped: August 30, 2022" />

* Complete the "**Ingest data**" > **Schema** form and then click "**Start Ingestion**"

  <img src="https://user-images.githubusercontent.com/44923999/187460984-fbbcc9ef-7479-4eb1-a824-c8646a2f9f1e.png" width="800" title="Snipped: August 30, 2022" />

* Confirm success on the "**Continuous ingestion ... established**" page

### Create with ARM Template

#### Step 1: Prepare Infrastructure
In this step, we will prepare the Azure resources necessary to complete the exercise. This solution requires:

* Data Explorer [**Table**](Infrastructure_DataExplorer_Table.md) and [**Ingestion Mapping**](Infrastructure_DataExplorer_IngestionMapping.md)
* Event Grid, [**Event Subscription**](Infrastructure_EventGrid_EventSubscription.md)... dependent on Event Hub and Storage Account
* [**Event Hub**](Infrastructure_EventHub.md)
* [**Storage Account**](Infrastructure_StorageAccount.md) with a container

#### Step 2: Deploy Custom Template
In this step, we will deploy a Data Connection on the Data Explorer Database using an ARM Template.

* Click the menu button in the upper-left corner of the Azure Portal and select "**Deploy a custom template**" from the resulting dropdown

  <img src="https://user-images.githubusercontent.com/44923999/182941824-1675b487-e60c-44ba-8a94-0eeaa8ee12af.png" width="800" title="Snipped: August 4, 2022" />

* Click "**Build your own template in the editor**"

  <img src="https://user-images.githubusercontent.com/44923999/182942508-5b378150-abc2-47de-924d-4a4720326fba.png" width="800" title="Snipped: August 4, 2022" />

* Select your data connection type, copy-paste into the "**Edit template**" window, and then modify the selected JSON:

##### Event Grid (Blob Storage)

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
          "EventHub_Name": { "type": "string", "defaultValue": "[concat(resourceGroup().name,'eh')]" },
          "EventSubscription_Name": { "type": "string", "defaultValue": "[concat(resourceGroup().name,'eges')]" }
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
                  "databaseRouting": "Single",
                  "eventGridResourceId": "[concat(resourceId(subscription().subscriptionId, resourceGroup().name, 'Microsoft.Storage/storageAccounts', parameters('DataLake_Name')), '/providers/Microsoft.EventGrid/eventSubscriptions/', parameters('EventSubscription_Name'))]"
              }
          }
      ]
  }
  ```

* Click **Save** 
* Confirm values on the resulting "**Custom deployment**" >> "**Deploy from a custom template**" page, **Basics** tab
* Click "**Review + create**", confirm configuration settings on the resulting page, and then click **Create**

#### Step 3: Assign Managed Identity
In this step, we will assign a Managed Instance to the deployed Data Connection.

* Navigate to your Data Explorer Database and then click "**Data Connections**" in the **Settings** grouping of the left-hand navigation
* On the resulting page, click on the newly deployed Data Connection

  <img src="https://user-images.githubusercontent.com/44923999/186000371-07f1c31e-483c-4976-a9ac-e3124745aee7.png" width="800" title="Snipped: August 22, 2022" />

* Select "**System-Assigned**" from the "**Assign managed identity**" drop-down
* Click **Update**
