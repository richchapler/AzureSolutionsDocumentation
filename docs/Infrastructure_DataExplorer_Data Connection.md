## Data Explorer >> Data Connection

### Create with ARM Template

_Note: Successful deployment of this ARM template depends on many resources:_

#### Step 1: Prepare Infrastructure

This solution requires the following resource:

* **Data Explorer** >> [Cluster](Infrastructure_DataExplorer_Cluster.md) :: [Database](Infrastructure_DataExplorer_Database.md) :: [Table](Infrastructure_DataExplorer_Table.md) :: [Ingestion Mapping](Infrastructure_DataExplorer_IngestionMapping.md)
* [Event Hub](Infrastructure_EventHub.md) >> Namespace :: Hub :: Consumer Group
* [Storage Account](Infrastructure_StorageAccount.md)

#### Step 2: Prepare Sample Data

Complete the following steps:

* Click the menu button in the upper-left corner of the Azure Portal and select **Deploy a custom template** from the resulting dropdown

  <img src="https://user-images.githubusercontent.com/44923999/182941824-1675b487-e60c-44ba-8a94-0eeaa8ee12af.png" width="800" title="Snipped: August 4, 2022" />

* Click **Build your own template in the editor**

  <img src="https://user-images.githubusercontent.com/44923999/182942508-5b378150-abc2-47de-924d-4a4720326fba.png" width="800" title="Snipped: August 4, 2022" />

* Paste and modify the following JSON:

  ```
  {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
        "resources": [{
              "type": "Microsoft.Kusto/Clusters/Databases/DataConnections",
              "apiVersion": "2019-09-07",
              "name": "rchaplerdec/rchaplerded/rchaplerdedc",
              "location": "[resourceGroup().location]",
              "tags": { "Environment": "PROD", "CostCenter":"123456" },
              "kind": "EventGrid",
              "properties": {
                  "storageAccountResourceId": "[resourceId(subscription().subscriptionId, resourceGroup().name, 'Microsoft.Storage/storageAccounts', 'rchaplersa')]",
                  "eventHubResourceId": "[resourceId(subscription().subscriptionId, resourceGroup().name, 'Microsoft.EventHub/namespaces/eventhubs', 'rchaplerehn', 'rchaplereh')]",
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
* Confirm values on the resulting **Custom deployment** >> **Deploy from a custom template** page, **Basics** tab
* Click **Review + create**
