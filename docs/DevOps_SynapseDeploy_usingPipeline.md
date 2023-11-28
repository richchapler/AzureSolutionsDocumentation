# Deploy Synapse Changes using a DevOps Pipeline

## Prepare Environment

* [**DevOps**](https://dev.azure.com/) with Organization, Project, Repository (dedicated to Synapse), and Branches "DEV" and "QA"
* [**Key Vault**](https://learn.microsoft.com/en-us/azure/key-vault) ???
* "DEV" Environment
  * [**Synapse**](Infrastructure_Synapse.md)
    * Git Configuration... Collaboration Branch "DEV"
    * Integration Runtime
* "QA" Environment
  * [**Synapse**](Infrastructure_Synapse.md)
    * Git Configuration... Collaboration Branch "QA"
    * Integration Runtime
