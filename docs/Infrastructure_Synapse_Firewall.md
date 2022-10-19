## Synapse, Firewall

### Create with ARM Template
Use this template to instantiate Synapse.
  
_Notes:_<br>
*	_This section depends on pre-instantiation of a Data Lake and a code repository (DevOps or GitHub)_
*	_To instantiate Synapse on a new subscription, we must ensure that Subscription > Resource Provider > Microsoft.SQL has been registered_
*	_Repository configuration {i.e., "workspaceRepositoryConfiguration"} is not included in the JSON because it is not possible to programmatically set the related permissions {e.g., GitHub Personal Access Token}; this key must be addressed manually_
*	_If you disable "publicNetworkAccess", you must also provide VNET configuration_

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
