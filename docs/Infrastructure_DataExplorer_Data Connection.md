## Data Explorer >> Data Connection

### Create with ARM Template

_Note: Successful deployment of this ARM template depends on many resources:_
* _Data Explorer >> [Cluster](Infrastructure_DataExplorer_Cluster.md) :: [Database](Infrastructure_DataExplorer_Database.md) :: [Table](Infrastructure_DataExplorer_Table.md) :: [Ingestion Mapping](Infrastructure_DataExplorer_IngestionMapping.md)_
* _[Event Hub](Infrastructure_EventHub.md) >> Namespace :: Hub :: Consumer Group_
* _[Storage Account](Infrastructure_StorageAccount.md)_

![image](https://user-images.githubusercontent.com/44923999/182941824-1675b487-e60c-44ba-8a94-0eeaa8ee12af.png)


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
