## Cognitive Search
_(aka "Azure Cognitive Search")_

### Create with Azure Portal

* Click the menu button in the upper-left corner of the Azure Portal
* Click **+ Create a resource** in the resulting dropdown
* Use the **Search services and marketplace** textbox to search for "Azure Cognitive Search" and select the appropriate dropdown value
* Find the correct result, select the **Create** dropdown, and then click the appropriate dropdown value
* Complete the **Create a storage account** form, **Basics** tab
 
  <img src="https://user-images.githubusercontent.com/44923999/178049387-11585534-df7f-430e-9d71-e8414692e66e.png" width="600" title="Snipped: July 8, 2022" />

* On the **Create a Storage Account** form, **Advanced** tab, check **Enable Hierarchical Namespace** in the **Data Lake Storage Gen2** grouping

  <img src="https://user-images.githubusercontent.com/44923999/178049285-9539e65a-4cdb-4b70-a4f0-593cf3c10d46.png" width="600" title="Snipped: July 8, 2022" />

  _Note: No additional configuration is required but consider review of the default values on the remaining tabs._

* Click **Review + create**, confirm configuration settings on the resulting page, and then click **Create**

_Note: When a Storage Account is configured for Data Lake Storage, you will see **Hierarchical Namespace**: Enabled on the Overview page_

### Create with ARM Template

```
{
"$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
"contentVersion": "1.0.0.0",
"resources":
  [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "[providers('Microsoft.Storage','storageAccounts').apiVersions[0]]",
      "name": "[concat(resourceGroup().name,'dl')]",
      "location": "[resourceGroup().location]",
      "sku": { "name": "Standard_LRS" },
      "kind": "StorageV2",
      "properties": { "isHnsEnabled": true }
    }
  ]
}
```

_Notes:_<br>
* _There is only one difference between the ARM templates for Data Lakes and Storage Accounts: the **isHnsEnabled** property_<br>
* _Suffix "dl" does not consider Gen1 vs Gen2_
