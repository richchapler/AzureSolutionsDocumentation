[Prepare Resources](PrepareResources.md) > **Resource Group**

## Create with Azure Portal

* Click the menu button in the upper-left corner of the Azure Portal
* Click **+ Create a resource** in the resulting dropdown
* Use the **Search services and marketplace** textbox to search for **resource group**
* Find the correct result, select the **Create** dropdown, and then click the appropriate dropdown value
* Complete the resulting **Create a resource group** form, **Basics** tab

  <img src="https://user-images.githubusercontent.com/44923999/178361671-5564b3a2-8297-4f72-a3dd-878cb75f2f7e.png" width="800" title="Snipped: July 11, 2022" />

  _Note: No additional configuration is required but consider review of the default values on the remaining tabs._

* Click **Review + create**, review configuration, and then click **Create**

## Create with ARM Template

Resource Groups must be deployed at the subscription-level {i.e., scope = "subscription"}.

```
{
    "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": { "ResourceGroupName": { "type": "string" } },
    "resources": [
        {
            "type": "Microsoft.Resources/resourceGroups",
            "apiVersion": "[providers('Microsoft.Resources','resourceGroups').apiVersions[0]]",
            "name": "[parameters('ResourceGroupName')]",
            "location": "westus3"
        }
    ]
}
