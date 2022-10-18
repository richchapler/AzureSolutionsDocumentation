## Synapse
_(aka "Azure Synapse Analytics", "Synapse Analytics Workspace")_

![image](https://user-images.githubusercontent.com/44923999/185975852-f21da095-6d6d-4259-86d8-6b199c9e3295.png)

### Create with Azure Portal

* Click the menu button in the upper-left corner of the Azure Portal
* Click **+ Create a resource** in the resulting dropdown
* Use the **Search services and marketplace** textbox to search for "synapse" and select the appropriate dropdown value
* Find the correct result, select the **Create** dropdown, and then click the appropriate dropdown value
* Complete the **Create Synapse workspace** form

  <img src="https://user-images.githubusercontent.com/44923999/179077930-cd2745c0-3d14-4db8-b8ae-a37cb18295a1.png" width="800" title="Snipped: July 14, 2022" />

  Prompt | Entry
  ------ | ------
  **Managed Resource Group** | This resource group will hold ancillary resources created specifically for Synapse<br><br>_Note: Because you are likely to have more than one managed Resource Group, consider using a consistent naming pattern {e.g., **<UseCase>mrg-s** for Synapse and **<UseCase>mrg-x** for Resource Type X}_
  **Account Name** | Select an existing data lake or click the **Create new** link to create a new data lake
  **File System Name** | Select an existing data lake file system or click the **Create new** link to create a new data lake file system

* Click the **Next: Security >** button and then select the **Use only Azure Active Directory (Azure AD) authentication** radio button
  _Note: Though local authentication may be appropriate to some use cases, I do not typically recommend it_

* Click **Review + create**, confirm configuration settings on the resulting page, and then click **Create**

### Create with ARM Template
  
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
