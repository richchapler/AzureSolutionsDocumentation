# Infrastructure: Storage Account

_(aka "Blob Storage")_<br>

![image](https://user-images.githubusercontent.com/44923999/185975618-ca3fc3ee-152a-47f7-b65e-7d8e19ebde5d.png)

The [Microsoft Learn: Azure Blob Storage](https://learn.microsoft.com/en-us/azure/storage/blobs/) documentation should serve as your primary source of information.

## Create with ARM Template

```
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "[providers('Microsoft.Storage','storageAccounts').apiVersions[0]]",
      "name": "[concat(resourceGroup().name,'sa')]",
      "location": "[resourceGroup().location]",
      "sku": { "name": "Standard_LRS" },
      "kind": "StorageV2"
    }
  ]
}
```
_Note: ARM Template deployment is incremental by default {i.e., an existing resource will not be destroyed and re-created}_
