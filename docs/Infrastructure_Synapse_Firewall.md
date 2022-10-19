## Synapse >> Firewall

### Create with ARM Template
Use this template to instantiate Synapse.
  
_Note: Rules should align with your security policies and network configuration._

  ```
  {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "resources": [
          {
              "type": "Microsoft.Synapse/workspaces",
              "apiVersion": "[providers('Microsoft.Synapse','workspaces').apiVersions[0]]",
              "name": "[concat(resourceGroup().name,'s')]",
              "location": "[resourceGroup().location]",
              "identity": { "type": "SystemAssigned" },
              "properties": {
                  "defaultDataLakeStorage": {
                      "resourceId": "[resourceId('Microsoft.Storage/storageAccounts',concat(resourceGroup().name,'dl'))]",
                      "createManagedPrivateEndpoint": false,
                      "accountUrl": "[concat('https://',resourceGroup().name,'dl.dfs.core.windows.net')]",
                      "filesystem": "concat(resourceGroup().name,'dlfs')"
                  },
                  "connectivityEndpoints": {
                      "web": "[concat('https://web.azuresynapse.net?workspace=%2fsubscriptions%2f',subscription().Id,'%2fresourceGroups%2f',resourceGroup().name,'%2fproviders%2fMicrosoft.Synapse%2fworkspaces%2f',resourceGroup().name,'s')]",
                      "dev": "[concat('https://',resourceGroup().name,'s','.dev.azuresynapse.net')]",
                      "sqlOnDemand": "[concat(resourceGroup().name,'s','-ondemand.sql.azuresynapse.net')]",
                      "sql": "[concat(resourceGroup().name,'s','.sql.azuresynapse.net')]"
                  },
                  "managedResourceGroupName": "[concat(resourceGroup().name,'mrg-s')]",
                  "azureADOnlyAuthentication": true,
                  "publicNetworkAccess": "Enabled"
              }
          }
      ]
  }
  ```
