# Deploy Synapse using Azure DevOps

## Challenges
* Deploying from DEV to QA... cannot parameterize integration runtime naming

## Prepare Environment

* [DevOps](https://dev.azure.com/) with Organization, Project, Repository (dedicated to Synapse), and Branches "DEV" and "QA"
* [Key Vault](https://learn.microsoft.com/en-us/azure/key-vault)
* "DEV" Environment
  * On-prem machine with:
    * [SQL Server](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) configured for SQL Server Authentication
    * Database "dbDEV"
  * [Synapse](Infrastructure_Synapse.md)
    * Git Configuration... Repository Type: Azure DevOps Git | Collaboration Branch "DEV"
    * Integration Runtime "irDEV" installed on on-prem SQL Server  
    * Linked Service "lsDEV" using integration runtime "irDEV" and connected to database "dbDEV"
* "QA" Environment
  * On-prem machine with:
    * [SQL Server](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) configured for SQL Server Authentication
    * Database "dbQA"
  * [Synapse](Infrastructure_Synapse.md)
    * Git Configuration... Repository Type: Azure DevOps Git | Collaboration Branch "QA"
    * Integration Runtime "irQA" installed on on-prem SQL Server  

-----
-----

## Exercise 1: Create Pipeline
In this exercise, we will use a DevOps Pipeline to: 1) archive the existing QA branch, 2) create a new branch from the PROD branch, and 3) complete a pull request from the DEV branch.

### Step 1: Create Starter Pipeline

Navigate to Azure DevOps >> Pipelines >> Pipelines.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/4b2f5f9e-530e-4e0b-abb5-a7274a70a0ad" width="800" title="Snipped: November 29, 2023" />

Click "+ Create Pipeline".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/f2c9ed00-82f2-4d1a-8312-daed57020761" width="800" title="Snipped: November 29, 2023" />

On the "Select a repository" page, click on your repository.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/08580679-e6b9-4fd0-b8c8-e069eec21544" width="800" title="Snipped: November 29, 2023" />

Complete the "Inventory your pipeline" form and then click "Configure pipeline".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/7a2c6ea2-47f4-4f16-913c-ac4edcf08c87" width="800" title="Snipped: November 29, 2023" />

On the "Configure your pipeline" page, select "Starter Pipeline" and then click "Review pipeline".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/beb54c28-61c0-473b-9cfb-06bef3e853a5" width="800" title="Snipped: November 29, 2023" />

Complete the "Review your governed pipeline" page and then click "Save and run".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/4062ce4d-ff23-47f0-aa7d-80421978d234" width="800" title="Snipped: November 29, 2023" />

Review default selections on the "Save and run" pop-out and then click "Save and run".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b140d488-b6b0-4bf0-b702-70eb0812f508" width="800" title="Snipped: November 29, 2023" />

Monitor progress and confirm success.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/3047b4f9-60a5-43bf-a8ff-caa6f3778deb" width="800" title="Snipped: November 29, 2023" />

You can expect email notification.

-----

### Step 2: Add Key Vault Secret

#### "azureSubscription"

Navigate to Azure DevOps >> Project >> Project Settings >> Pipelines >> Service Connections.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b85a6960-5827-4578-97bc-55dc366862e0" width="800" title="Snipped: November 29, 2023" />

Copy the name of the Service Connection to your Azure Subscription.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/9d2b0d78-76dd-4a3d-a738-29d2fb86d818" width="800" title="Snipped: November 29, 2023" />

Navigate to your Key Vault >> Objects >> Secrets and click "+ Generate/Import".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e1046dc4-1f3c-401a-aa20-f33c45d16710" width="800" title="Snipped: November 29, 2023" />

Complete the "Create a secret" form, including:

Prompt | Entry
:----- | :-----
Name | azureSubscription
Secret Value | Previously copied Service Connection Name

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/61067fc1-02b1-4b4b-b471-b0095937ae70" width="800" title="Snipped: November 29, 2023" />

-----

### Step 3: Create Variable Group

Navigate to Azure DevOps >> Pipelines >> Library.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/9a8677a7-2df5-4eb4-b5a4-5a898157ab2c" width="800" title="Snipped: November 29, 2023" />

Click "+ Variable group".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d83881b4-f506-4785-b7cb-02f48e345ed3" width="800" title="Snipped: November 29, 2023" />

Complete the "Properties" form, including:

Prompt | Entry
:----- | :-----
Link secrets from an Azure key vault... | Active
Azure Subscription | Select and Authorize
Key Vault Name | Select and Authorize<br><sub>_Note: Assign the "Key Vaults Secrets User" role to your Azure DevOps identity to successfully Authorize_</sub>

Click "+ Add".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/4058af94-3595-4526-a68a-ec6d7172520b" width="800" title="Snipped: November 29, 2023" />

Click to select the "SubscriptionGUID" secret and then click "Ok".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0f321460-0304-4da5-81c2-2c6f24f7972c" width="800" title="Snipped: November 29, 2023" />

On the "Properties" page, click "Save".

-----

### Step 4: Edit Pipeline YAML

Navigate to Azure DevOps >> Pipelines >> Pipelines, select your pipeline and click "Edit".


```
pool:
  vmImage: 'windows-latest'

steps:
- task: AzureCLI@2
  inputs:
    azureSubscription: $SubscriptionGUID
    scriptType: 'pscore'
    scriptLocation: 'inlineScript'
    inlineScript: |
      $o = "https://dev.azure.com/rchaplerdo"
      $p = "rchaplerdp"
      $r = "rchaplerdr"
      $b = "QA"
      $dt = Get-Date -Format "yyyyMMdd_HHmmss"

      $oid = az repos ref show --name "refs/heads/$b" --query objectId --output tsv --project $p --repository $r --organization $o
      az repos ref create --name "refs/heads/$b_$dt" --object-id $oid --project $p --repository $r --organization $o
```

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/c123b930-43ce-4dd1-9159-1ff31d1be558" width="800" title="Snipped: November 29, 2023" />


Lorem Ipsum

-----

**Congratulations... you have successfully completed this exercise**

-----
-----
