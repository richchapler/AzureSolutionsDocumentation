## Setup Azure Synapse Link for Dataverse

Resources
* Power Platform Environment
  * with Sample Apps and Data deployed
  * Dataverse
    * Only tables with "Track changes" enabled
* Synapse ... in East US, East US 2 or East US 2 EUAP
  * Data Lake Storage Gen2
    * File System
    * Storage Blob Data Contributor role
    * Use only Azure Active Directory (Azure AD) authentication
    * Allow connections from all IP addresses
    * Spark pool

* https://make.preview.powerapps.com/
* Click on Tables >> Account
* Export >> Link to Azure Synapse

![image](https://github.com/richchapler/AzureSolutions/assets/44923999/2e461a2a-b48d-4d89-a74d-ef81b7127896)


Sign in to https://make.powerapps.com and select your preferred environment.
On the left navigation pane, expand the Dataverse menu.
Select Azure Synapse Link.
On the command bar, click New link 1.
You can also find more detailed instructions on how to connect Dataverse to your Azure Synapse Workspace or Azure Data Lake Storage Gen2, manage table data, monitor your Azure Synapse Link, and more on the Microsoft Learn website 23.

Is there anything else you would like to know?

## Reference

* []()
