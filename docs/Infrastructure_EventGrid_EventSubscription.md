## Event Grid >> Event Subscription

![image](https://user-images.githubusercontent.com/44923999/185961643-9ef32726-fc68-42b7-8c98-bc8cdef9207d.png)

### Create with ARM Template

```
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "DataLake_Name": { "type": "string", "defaultValue": "[concat(resourceGroup().name,'dls')]" },
        "DataLakeContainer_Name": { "type": "string", "defaultValue": "[concat(resourceGroup().name,'dlsc')]" },
        "EventHubNamespace_Name": { "type": "string", "defaultValue": "[concat(resourceGroup().name,'ehn')]" },
        "EventHub_Name": { "type": "string", "defaultValue": "[concat(resourceGroup().name,'eh')]" },
        "EventSubscription_Name": { "type": "string", "defaultValue": "[concat(resourceGroup().name,'eges')]" }
    },
    "resources": [
        {
            "name": "[concat(parameters('DataLake_Name'),'/Microsoft.EventGrid/',parameters('EventSubscription_Name'))]",
            "type": "Microsoft.Storage/storageAccounts/providers/eventSubscriptions",
            "apiVersion": "[providers('Microsoft.EventGrid','eventSubscriptions').apiVersions[0]]",
            "properties": {
                "destination": {
                    "endpointType": "EventHub",
                    "properties": {
                        "resourceId": "[resourceId(subscription().subscriptionId, resourceGroup().name, 'Microsoft.EventHub/namespaces/eventhubs', parameters('EventHubNamespace_Name'), parameters('EventHub_Name'))]"
                    }
                },
                "filter": {
                    "subjectBeginsWith": "[concat('/blobServices/default/containers/',parameters('DataLakeContainer_Name'))]",
                    "subjectEndsWith": ".csv",
                    "includedEventTypes": ["Microsoft.Storage.BlobCreated"]
                }
            }
        }
    ]
}
```
_Note: ARM Template deployment is incremental by default {i.e., an existing resource will not be destroyed and re-created}_
