## [Data Explorer](Infrastructure_DataExplorer.md) >> **Data Connection**

![image](https://user-images.githubusercontent.com/44923999/185688537-f5257f22-eb25-405f-be9b-1cd45c6574f4.png)

### Create with ARM Template

### Prepare Infrastructure

This solution requires:

* **Data Explorer** [cluster](Infrastructure_DataExplorer_Cluster.md) and [database](Infrastructure_DataExplorer_Database.md), with a [table](Infrastructure_DataExplorer_Table.md) and [ingestion mapping](Infrastructure_DataExplorer_IngestionMapping.md)
* [**Storage Account**](Infrastructure_StorageAccount.md)

<br>...and, depending on your source, will also require one of the following:

* [**Event Hub**](Infrastructure_EventHub.md)

### Step 1: Deploy Custom Template

Complete the following steps:

* Click the menu button in the upper-left corner of the Azure Portal and select "**Deploy a custom template**" from the resulting dropdown

  <img src="https://user-images.githubusercontent.com/44923999/182941824-1675b487-e60c-44ba-8a94-0eeaa8ee12af.png" width="800" title="Snipped: August 4, 2022" />

* Click "**Build your own template in the editor**"

  <img src="https://user-images.githubusercontent.com/44923999/182942508-5b378150-abc2-47de-924d-4a4720326fba.png" width="800" title="Snipped: August 4, 2022" />

* Select your data connection type, copy-paste into the "**Edit template**" window, and then modify the selected JSON:

#### Event Grid (Blob Storage)

NEW VERSION, REFINEMENT PENDING...
```
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "automaticCreation": {
            "defaultValue": true,
            "type": "Bool",
            "metadata": {
                "description": "Specifies weather the Event Hub and Event Grid resources be deployed."
            }
        },
        "location": {
            "type": "String",
            "metadata": {
                "description": "Specifies the Azure location for all resources."
            }
        },
        "kustoClusterName": {
            "type": "String",
            "metadata": {
                "description": "Specifies the name of the cluster"
            }
        },
        "kustoDataConnectionDatabaseName": {
            "type": "String",
            "metadata": {
                "description": "Specifies the name of the database"
            }
        },
        "kustoDataConnectionTableName": {
            "defaultValue": "",
            "type": "String",
            "metadata": {
                "description": "Specifies the name of the table"
            }
        },
        "kustoDataConnectionName": {
            "defaultValue": "one_click",
            "type": "String",
            "metadata": {
                "description": "Name of the data connection to create"
            }
        },
        "kustoDataConnectionMappingName": {
            "defaultValue": "",
            "type": "String",
            "metadata": {
                "description": "Specifies the name of the mapping rule"
            }
        },
        "kustoDataConnectionDataFormatType": {
            "defaultValue": "",
            "type": "String",
            "metadata": {
                "description": "Specifies the data format"
            }
        },
        "eventHubSubscription": {
            "defaultValue": "[subscription().subscriptionId]",
            "type": "String",
            "metadata": {
                "description": "Specifies the subscription of the Event Hubs resource."
            }
        },
        "eventHubResourceGroupName": {
            "defaultValue": "[resourceGroup().name]",
            "type": "String",
            "metadata": {
                "description": "Specifies the resource group of the Event Hubs resource."
            }
        },
        "eventHubNamespaceName": {
            "type": "String",
            "metadata": {
                "description": "Specifies the name of the Event Hubs namespace resource. This must be unique"
            }
        },
        "eventHubName": {
            "minLength": 1,
            "type": "String",
            "metadata": {
                "description": "The name of the new Event Hub (autoCreate), NestedTemplate (autoCreate) and the name of its Event Grid subscription (autoCreate)."
            }
        },
        "eventHubConsumerGroupName": {
            "defaultValue": "$Default",
            "type": "String",
            "metadata": {
                "description": "Specifies the name of the Event Hubs consumer group to listen to"
            }
        },
        "addEventhubRoleAssignment": {
            "defaultValue": false,
            "type": "Bool",
            "metadata": {
                "description": "Specifies if to assign"
            }
        },
        "storageName": {
            "type": "String",
            "metadata": {
                "description": "Specifies the storage account to listen to"
            }
        },
        "storageSubscription": {
            "defaultValue": "[subscription().subscriptionId]",
            "type": "String",
            "metadata": {
                "description": "The subscriptionId of the storage"
            }
        },
        "storageResourceGroupName": {
            "defaultValue": "[resourceGroup().name]",
            "type": "String",
            "metadata": {
                "description": "The resourceGroup name of the storage "
            }
        },
        "storageBlobPrefix": {
            "type": "String",
            "metadata": {
                "description": "The prefix of the blobs names that will trigger events - this includes the container name"
            }
        },
        "storageBlobSuffix": {
            "defaultValue": "",
            "type": "String",
            "metadata": {
                "description": "The suffix of the blobs names that will trigger events"
            }
        },
        "storageBlobFilterIsCaseSensitive": {
            "defaultValue": false,
            "type": "Bool",
            "metadata": {
                "description": "Specifies if the prefix and suffix filters are case sensitive"
            }
        },
        "storageBlobEventType": {
            "defaultValue": "Microsoft.Storage.BlobCreated",
            "allowedValues": [
                "Microsoft.Storage.BlobCreated",
                "Microsoft.Storage.BlobRenamed"
            ],
            "type": "String",
            "metadata": {
                "description": "The name of blob storage event type to process."
            }
        },
        "storageBlobHasHeaders": {
            "defaultValue": false,
            "type": "Bool",
            "metadata": {
                "description": "A Boolean value that, if set to true, indicates that ingestion should ignore the first record of the delimited file"
            }
        },
        "kustoDataConnectionManagedIdentityResourceId": {
            "defaultValue": "",
            "type": "String",
            "metadata": {
                "description": "Specifies the resource id of the managed identity"
            }
        },
        "kustoDataConnectionManagedIdentityPrincipalId": {
            "defaultValue": "",
            "type": "String",
            "metadata": {
                "description": "Specifies the principal id of the managed identity"
            }
        },
        "kustoDataConnectionDatabaseRouting": {
            "defaultValue": "Single",
            "allowedValues": [
                "Single",
                "Multi"
            ],
            "type": "String",
            "metadata": {
                "description": "Indication for database routing information, by default only database routing information is allowed"
            }
        },
        "addStorageAccountRoleAssignment": {
            "defaultValue": false,
            "type": "Bool",
            "metadata": {
                "description": "Specifies if to assign"
            }
        },
        "eventGridName": {
            "defaultValue": "",
            "type": "String",
            "metadata": {
                "description": "The EventGrid event subscription name. Used for interaction with the EventGrid resource. If not specified - name is taken from eventHubName parameter."
            }
        }
    },
    "variables": {
        "eventHubDataReceiverRoleDefinitionId": "[concat('/subscriptions/', parameters('eventHubSubscription'), '/providers/Microsoft.Authorization/roleDefinitions/', 'a638d3c7-ab3a-418d-83e6-5f17a39d4fde')]",
        "storageDataReceiverRoleDefinitionId": "[concat('/subscriptions/', parameters('storageSubscription'), '/providers/Microsoft.Authorization/roleDefinitions/', '2a2b9908-6ea1-4ae2-8e65-a410df84e7d1')]",
        "nestedTemplateName": "[concat(parameters('eventHubName'), 'Template-storage-eventSubscription')]",
        "nestedDeployment": "[resourceId(parameters('storageSubscription'), parameters('storageResourceGroupName'), 'Microsoft.Resources/deployments', variables('nestedTemplateName'))]",
        "kustoManagedIdentityEventHubRoleAssignmentId": "[guid(concat(resourceGroup().id, 'AzureEventHubsDataReceiver', parameters('kustoDataConnectionName')))]",
        "kustoManagedIdentityStorageRoleAssignmentId": "[guid(concat(resourceGroup().id, 'StorageBlobDataContributor', parameters('kustoDataConnectionName')))]",
        "eventHubManualResourceId": "[resourceId(parameters('eventHubSubscription'), parameters('eventHubResourceGroupName'), 'Microsoft.EventHub/namespaces/eventhubs', parameters('eventHubNamespaceName'), parameters('eventHubName'))]",
        "eventHubAutoCreateResourceId": "[resourceId('Microsoft.EventHub/namespaces/eventhubs', parameters('eventHubNamespaceName'), parameters('eventHubName'))]",
        "eventHubResourceId": "[if(parameters('automaticCreation'), variables('eventHubAutoCreateResourceId'), variables('eventHubManualResourceId'))]",
        "synapseWorkSpaceAndKustoPool": "[split(parameters('kustoClusterName'), '/')]",
        "isManagedIdentity": "[and(not(empty(parameters('kustoDataConnectionManagedIdentityResourceId'))), not(empty(parameters('kustoDataConnectionManagedIdentityPrincipalId'))))]",
        "storageAccountResourceId": "[resourceId(parameters('storageSubscription'), parameters('storageResourceGroupName'), 'Microsoft.Storage/storageAccounts', parameters('storageName'))]"
    },
    "resources": [
        {
            "type": "Microsoft.EventHub/namespaces",
            "apiVersion": "2017-04-01",
            "name": "[parameters('eventHubNamespaceName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "Standard",
                "tier": "Standard",
                "capacity": 1
            },
            "properties": {
                "isAutoInflateEnabled": false,
                "maximumThroughputUnits": 0
            },
            "condition": "[parameters('automaticCreation')]"
        },
        {
            "type": "Microsoft.EventHub/namespaces/eventhubs",
            "apiVersion": "2017-04-01",
            "name": "[concat(parameters('eventHubNamespaceName'), '/', parameters('eventHubName'))]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[resourceId('Microsoft.EventHub/namespaces', parameters('eventHubNamespaceName'))]"
            ],
            "properties": {
                "messageRetentionInDays": 7,
                "partitionCount": 8
            },
            "condition": "[parameters('automaticCreation')]"
        },
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2018-05-01",
            "name": "[variables('nestedTemplateName')]",
            "dependsOn": [
                "[variables('eventHubAutoCreateResourceId')]"
            ],
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "name": "[concat(parameters('storageName'), '/Microsoft.EventGrid/', if(empty(parameters('eventGridName')), parameters('eventHubName'), parameters('eventGridName')))]",
                            "type": "Microsoft.Storage/storageAccounts/providers/eventSubscriptions",
                            "apiVersion": "2022-06-15",
                            "properties": {
                                "destination": {
                                    "endpointType": "EventHub",
                                    "properties": {
                                        "resourceId": "[variables('eventHubResourceId')]"
                                    }
                                },
                                "filter": {
                                    "subjectBeginsWith": "[parameters('storageBlobPrefix')]",
                                    "subjectEndsWith": "[parameters('storageBlobSuffix')]",
                                    "includedEventTypes": [
                                        "[parameters('storageBlobEventType')]"
                                    ],
                                    "isSubjectCaseSensitive": "[parameters('storageBlobFilterIsCaseSensitive')]"
                                }
                            }
                        }
                    ]
                },
                "parameters": {}
            },
            "subscriptionId": "[parameters('storageSubscription')]",
            "resourceGroup": "[parameters('storageResourceGroupName')]",
            "condition": "[parameters('automaticCreation')]"
        },
        {
            "type": "Microsoft.Kusto/Clusters/Databases/dataConnections",
            "apiVersion": "2022-02-01",
            "name": "[concat(parameters('kustoClusterName'), '/', parameters('kustoDataConnectionDatabaseName'), '/', parameters('kustoDataConnectionName'))]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[variables('nestedDeployment')]"
            ],
            "kind": "EventGrid",
            "properties": {
                "storageAccountResourceId": "[variables('storageAccountResourceId')]",
                "eventHubResourceId": "[variables('eventHubResourceId')]",
                "consumerGroup": "[parameters('eventHubConsumerGroupName')]",
                "tableName": "[parameters('kustoDataConnectionTableName')]",
                "mappingRuleName": "[parameters('kustoDataConnectionMappingName')]",
                "dataFormat": "[parameters('kustoDataConnectionDataFormatType')]",
                "blobStorageEventType": "[parameters('storageBlobEventType')]",
                "ignoreFirstRecord": "[parameters('storageBlobHasHeaders')]",
                "managedIdentityResourceId": "[parameters('kustoDataConnectionManagedIdentityResourceId')]",
                "databaseRouting": "[parameters('kustoDataConnectionDatabaseRouting')]",
                "eventGridResourceId": "[concat(variables('storageAccountResourceId'), '/providers/Microsoft.EventGrid/eventSubscriptions/', if(empty(parameters('eventGridName')), parameters('eventHubName'), parameters('eventGridName')))]"
            }
        },
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2018-05-01",
            "name": "[concat(parameters('eventHubName'), '-eventhub-roleAssignment')]",
            "dependsOn": [
                "[variables('nestedDeployment')]",
                "[resourceId('Microsoft.Kusto/Clusters/Databases/dataConnections', parameters('kustoClusterName'), parameters('kustoDataConnectionDatabaseName'), parameters('kustoDataConnectionName'))]"
            ],
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "type": "Microsoft.EventHub/namespaces/eventhubs/providers/roleAssignments",
                            "apiVersion": "2021-04-01-preview",
                            "name": "[concat(parameters('eventHubNamespaceName'), '/', parameters('eventHubName'), '/Microsoft.Authorization/', variables('kustoManagedIdentityEventHubRoleAssignmentId'))]",
                            "properties": {
                                "roleDefinitionId": "[variables('eventHubDataReceiverRoleDefinitionId')]",
                                "principalId": "[parameters('kustoDataConnectionManagedIdentityPrincipalId')]"
                            }
                        }
                    ]
                },
                "parameters": {}
            },
            "subscriptionId": "[parameters('eventHubSubscription')]",
            "resourceGroup": "[parameters('eventHubResourceGroupName')]",
            "condition": "[and(variables('isManagedIdentity'), parameters('addEventhubRoleAssignment'))]"
        },
        {
            "type": "Microsoft.Resources/deployments",
            "apiVersion": "2018-05-01",
            "name": "[concat(parameters('eventHubName'), '-storage-roleAssignment')]",
            "dependsOn": [
                "[variables('nestedDeployment')]",
                "[resourceId('Microsoft.Kusto/Clusters/Databases/dataConnections', parameters('kustoClusterName'), parameters('kustoDataConnectionDatabaseName'), parameters('kustoDataConnectionName'))]"
            ],
            "properties": {
                "mode": "Incremental",
                "template": {
                    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                    "contentVersion": "1.0.0.0",
                    "parameters": {},
                    "variables": {},
                    "resources": [
                        {
                            "type": "Microsoft.Storage/storageAccounts/providers/roleAssignments",
                            "apiVersion": "2021-04-01-preview",
                            "name": "[concat(parameters('storageName'), '/Microsoft.Authorization/', variables('kustoManagedIdentityStorageRoleAssignmentId'))]",
                            "properties": {
                                "roleDefinitionId": "[variables('storageDataReceiverRoleDefinitionId')]",
                                "principalId": "[parameters('kustoDataConnectionManagedIdentityPrincipalId')]"
                            }
                        }
                    ]
                },
                "parameters": {}
            },
            "subscriptionId": "[parameters('storageSubscription')]",
            "resourceGroup": "[parameters('storageResourceGroupName')]",
            "condition": "[and(variables('isManagedIdentity'), parameters('addStorageAccountRoleAssignment'))]"
        }
    ]
}
```

OLD VERSION...
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
* Confirm values on the resulting "**Custom deployment**" >> "**Deploy from a custom template**" page, **Basics** tab
* Click "**Review + create**", confirm configuration settings on the resulting page, and then click **Create**
