# Synapse >> Firewall Rules

![image](https://user-images.githubusercontent.com/44923999/185975852-f21da095-6d6d-4259-86d8-6b199c9e3295.png)

[Microsoft Learn: Azure Synapse Analytics](https://learn.microsoft.com/en-us/azure/synapse-analytics/security/synapse-workspace-ip-firewall) should serve as your primary source of information.

## Create with ARM Template
Use this template to add Firewall Rules to Synapse.

_Notes:_<br>
* _Rules should align with your security policies and network configuration._
* _You should maintain a single file with all firewall rules to provide for "bare metal" implementation (and simplify overall management)_

  ```
  {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "resources": [
          {
              "type": "Microsoft.Synapse/workspaces/firewallRules",
              "apiVersion": "[providers('Microsoft.Synapse','workspaces').apiVersions[0]]",
              "name": "[concat(resourceGroup().name,'s','/AllowAllWindowsAzureIps')]",
              "properties": { "startIpAddress": "0.0.0.0", "endIpAddress": "0.0.0.0" }
          },
          {
              "type": "Microsoft.Synapse/workspaces/firewallRules",
              "apiVersion": "[providers('Microsoft.Synapse','workspaces').apiVersions[0]]",
              "name": "[concat(resourceGroup().name,'s','/allowRange')]",
              "properties": { "startIpAddress": "#.#.0.0", "endIpAddress": "#.#.255.255" }
          }
      ]
  }
  ```
