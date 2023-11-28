# Deploy Synapse Changes using a DevOps Pipeline

## Challenges
* Deploying from DEV to QA... cannot parameterize integration runtime naming

## Prepare Environment

* [DevOps](https://dev.azure.com/) with Organization, Project, Repository (dedicated to Synapse), and Branches "DEV" and "QA"
* [Key Vault](https://learn.microsoft.com/en-us/azure/key-vault) ???
* "DEV" Environment
  * [Synapse](Infrastructure_Synapse.md)
    * Git Configuration... Repository Type: Azure DevOps Git | Collaboration Branch "DEV"
    * Integration Runtime "irDEV" on an on-prem machine with [SQL Server](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) / Database "dbDEV"
    * Linked Service "lsDEV" that uses integration runtime "irDEV" and is connected to database "dbDEV"
* "QA" Environment
  * [Synapse](Infrastructure_Synapse.md)
    * Git Configuration... Repository Type: Azure DevOps Git | Collaboration Branch "QA"
    * Integration Runtime "irQA" on an on-prem machine with [SQL Server](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) / Database "dbQA"
