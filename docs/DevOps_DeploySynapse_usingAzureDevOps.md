# Deploy Synapse using Azure DevOps

## Challenges
* Deploying from DEV to QA... cannot parameterize integration runtime naming

## Prepare Environment

* [DevOps](https://dev.azure.com/) with Organization, Project, Repository (dedicated to Synapse), and Branches "DEV" and "QA"
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

### Step 2: Create Variable Group

Navigate to Azure DevOps >> Pipelines >> Library.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/9a8677a7-2df5-4eb4-b5a4-5a898157ab2c" width="800" title="Snipped: November 29, 2023" />

Click "+ Variable group".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2b724210-ba53-421c-93cb-165be5a37ec1" width="800" title="Snipped: November 29, 2023" />

Complete the "Properties" form, including:

Prompt | Entry
:----- | :-----
Link secrets from an Azure key vault as variable | Active
Azure Subscription | Select and Authorize
Key Vault Name | Select and Authorize

_Note: You must assign the "Key Vaults Secrets User" role to your managed identity for Azure DevOps {e.g., "rchaplerdo-rchaplerdp-GUID"} in order to successfully Authorize on the "Key Vault Name" row_



Lorem Ipsum

-----

**Congratulations... you have successfully completed this exercise**

-----
-----
