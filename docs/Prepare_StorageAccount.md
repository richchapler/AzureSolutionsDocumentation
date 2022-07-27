##Storage Account

### Create with ARM Template

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
