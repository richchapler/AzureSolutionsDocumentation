# Synapse
_(aka "Azure Synapse Analytics", "Synapse Analytics Workspace")_

![image](https://user-images.githubusercontent.com/44923999/185975852-f21da095-6d6d-4259-86d8-6b199c9e3295.png)

_Note: Use [Microsoft Learn: Azure Synapse Analytics](https://learn.microsoft.com/en-us/azure/synapse-analytics/) documentation for most questions. The documentation below covers additional, specific guidance around topics like use of Azure Active Directory, firewall, etc._

## Specific Guidance
### Managed Resource Group
**Recommendation**: Provide a name aligned with your naming standard

* If you do not specify a name for the Managed Resource Group, Synapse will automatically generate a name on your behalf (and odds are good that it will not align with your naming standards)
* Because you are likely to have more than one managed Resource Group, consider using a consistent naming pattern {e.g., **<UseCase>mrgs** for Synapse and **<UseCase>mrgx** for Resource Type X

### Security >> Authentication
**Recommendation**: Use only Azure Active Directory
 
* Advantages of using only Azure Active Directory...
  * **Centralized user management**: Azure AD allows for centralized user management, making it easier to manage user accounts and permissions across different systems and applications
  * **Improved security**: using Azure AD for authentication enhances security by providing additional layers of authentication and authorization, and by reducing the risk of password-related security incidents
  * **Single sign-on (SSO)**: Azure AD enables single sign-on (SSO) across different systems and applications, making it easier for users to access the resources they need without having to remember multiple usernames and passwords
  * **Scalability**: Azure AD is a scalable solution that can handle a large number of user accounts and authentication requests, making it suitable for use in large organizations
  * **Integration with other Azure services**: Azure AD can be integrated with other Azure services, such as Azure Functions, Power Apps, and Microsoft 365, allowing for a more seamless user experience

* Disadvantages of using only Azure Active Directory...
  * **Dependency on Azure AD**: using only Azure AD authentication creates a dependency on the availability and performance of Azure AD, which can impact the availability and performance of your Azure Synapse instance
  * **Cost**: using Azure AD for authentication may incur additional costs, depending on the specific Azure AD plan you are using
  * **Integration challenges**: integrating Azure AD with your existing systems and applications may require additional time and effort, particularly if you need to integrate with systems that use different authentication methods
  * **Complexity**: using Azure AD for authentication can increase the complexity of managing user accounts and permissions, especially in large organizations with many users

### Networking >> "**Allow connections from all IP addresses**"
**Recommendation**: Prevent the creation of a firewall rule that opens 0.0.0.0 to 255.255.255.255; set individual firewall rules instead
  
* Advantages of allowing connections from all IP addresses
  * **Accessibility**: This option allows anyone with the proper credentials to access the Synapse instance from anywhere, as long as they have an internet connection
  * **Convenience**: This option can be convenient for users who need to access the Synapse instance from different locations, as they will not have to worry about the IP address of their device changing

* Disadvantages of allowing connections from all IP addresses
  * **Security**: Allowing connections from all IP addresses can increase the security risks, as it leaves the Synapse instance more vulnerable to unauthorized access
  * **Performance**: Allowing connections from all IP addresses can have a negative impact on performance, as it can result in a large number of incoming connections that need to be processed
  * **Cost**: Allowing connections from all IP addresses may result in increased costs, as it may require additional resources to process the incoming connections

## Create with ARM Template
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
