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

## Exercise 1: Starter Pipeline
In this exercise, we will use a DevOps Pipeline to: 1) archive the existing QA branch, 2) create a new branch from the PROD branch, and 3) complete a pull request from the DEV branch.

### Step 1: Link Subscription

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

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/3c74d5d9-8ce0-43b6-9cb8-72255538f7fb" width="800" title="Snipped: November 29, 2023" />

Click to select the "azureSubscription" secret, then click "Ok".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/66199de0-e600-4fd1-8e46-7524b7b467b3" width="800" title="Snipped: November 29, 2023" />

On the "Properties" page, click "Save".

-----

### Step 2: Create Pipeline

Navigate to Azure DevOps >> Pipelines >> Pipelines.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/4b2f5f9e-530e-4e0b-abb5-a7274a70a0ad" width="800" title="Snipped: November 29, 2023" />

Click "+ Create Pipeline".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/f2c9ed00-82f2-4d1a-8312-daed57020761" width="800" title="Snipped: November 29, 2023" />

On the "Select a repository" page, click on your repository.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/08580679-e6b9-4fd0-b8c8-e069eec21544" width="800" title="Snipped: November 29, 2023" />

Complete the "Inventory your pipeline" form and then click "Configure pipeline".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/7a2c6ea2-47f4-4f16-913c-ac4edcf08c87" width="800" title="Snipped: November 29, 2023" />

On the "Configure your pipeline" page, select "Starter Pipeline" and then click "Review pipeline".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d396c1ea-2843-4b1b-a450-d47b93fc2c4a" width="800" title="Snipped: November 29, 2023" />

Complete the "Review your governed pipeline" page, dropdown "Save and run", and then click "Save".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/ed5bd309-9963-48cd-95c1-40c6c800de29" width="800" title="Snipped: November 29, 2023" />

Review default selections on the "Save and run" pop-out and then click "Save".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/f3b830df-6faf-4e59-b467-dbe8eacf4172" width="800" title="Snipped: November 29, 2023" />

Click "Edit" and on the resulting screen, paste the following YAML:

```
pool:
  vmImage: 'windows-latest'

steps:
- script: az config set extension.use_dynamic_install=yes_without_prompt
  displayName: 'Allow Extensions'
- script: echo $(System.AccessToken) | az devops login
  displayName: 'Login to DevOps'
- task: AzureCLI@2
  displayName: 'Prepare Branches'
  inputs:
    azureSubscription: "MCAPS-Hybrid-REQ-38779-2022-RichardChapler (ed7eaf77-d411-484b-92e6-5cba0b6d8098)"
    scriptType: 'pscore'
    scriptLocation: 'inlineScript'
    inlineScript: |
      $o = "https://dev.azure.com/rchaplerdo"
      $p = "rchaplerdp"
      $r = "rchaplerdr"
      $b = "QA"
      $dt = Get-Date -Format "yyyyMMddHHmmss"
      $branches = az repos ref list --org $o -p $p -r $r --filter "heads/" | ConvertFrom-Json
      $oid = $branches | Where-Object {$_.name -eq "refs/heads/$b"} | Select-Object -ExpandProperty objectId
      az repos ref create --name "refs/heads/$b-$dt" --object-id $oid --project $p --repository $r --organization $o
```
variables:
- group: 'rchaplerdvg'
azureSubscription: $(azureSubscription)
az upgrade
az extension add -n azure-devops
az extension update -n azure-devops
      
      ERROR: TF401027: You need the Git 'CreateBranch' permission to perform this action. Details: identity 'Build\af617e9d-b167-4635-9ddc-21574b369387', scope 'repository'.

### Need to grant these permissions... Build\af617e9d-b167-4635-9ddc-21574b369387, Git 'CreateBranch' permission

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/1b7e9fce-b282-49ec-b3ef-3889ee22b2b2" width="800" title="Snipped: November 29, 2023" />

Logic Explained:
LOREM IPSUM
* `variables` define variables that can be used throughout the pipeline (in this case, the variable group created in Exercise 1)
* `pool...` specifies use of a virtual machine with the latest version of Windows
* `steps` contains pipeline tasks (in this case, just one)
  * `task: AzureCLI@2` runs commands using version 2 of the Azure CLI
    * `inputs` specifies the inputs for the Azure CLI task:
      * `azureSubscription: $(azureSubscription)` specifies the Azure subscription to use (using the previously-set variable)
      * `scriptType: 'pscore'` indicates a PowerShell Core script
      * `scriptLocation: 'inlineScript'` indicates that the script to be executed is written directly in the YAML file
      * `inlineScript` is the actual script, which does the following:
        * Sets variables for the organization URL (`$o`), project (`$p`), repository (`$r`), branch (`$b`), and current datetime (`$dt`)
        * Retrieves the object ID of the specified branch in the repository with the `az repos ref show` command
        * Creates a new branch with the same object ID as the original branch using the `az repos ref create` command
          * The new branch's name is the original branch's name appended with the current datetime

Click "Save".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0badec96-6cb1-47c7-a7b7-d6a4b6b2f03b" width="800" title="Snipped: November 29, 2023" />

#### Confirm Success

Click "Run".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/810058e4-b916-4e51-bdfb-34704324fd8b" width="800" title="Snipped: November 30, 2023" />

On the resulting "Run pipeline" pop-out, review default settings, check "Enable system diagnostics", and then click "Run".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/9b092bda-909e-46fd-bd45-0124d5972002" width="800" title="Snipped: November 30, 2023" />










This pipeline is useful for creating a new branch from an existing one in an Azure DevOps repository. The new branch will have the same commit history as the original branch. The branch name includes a timestamp, which can be useful for keeping track of when the branch was created.





Lorem Ipsum

-----

**Congratulations... you have successfully completed this exercise**

-----
-----

## Reference

* [Use personal access tokens](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate)

To create, list, and manage branches in Azure DevOps using a Personal Access Token (PAT), the token needs to have the appropriate permissions¹. Here are the permissions you might need:

1. **Code (Read & Write)**: This permission allows you to read and write to the code, which includes creating and deleting branches¹.

2. **Code (Manage)**: This permission allows you to manage code repositories, which includes managing branches¹.

When creating a PAT, you can select the scopes for the token to authorize for your specific tasks¹. For example, to create a token to enable a build and release agent to authenticate to Azure DevOps Services, limit your token's scope to Agent Pools (Read & manage)¹.

Please note that you should treat and use a PAT like your password and keep it a secret¹.

I hope this helps! If you have any other questions, feel free to ask. 😊

Source: Conversation with Bing, 11/30/2023
(1) Use personal access tokens - Azure DevOps | Microsoft Learn. https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops.
(2) Azure devops feed with PAT : what permissions are needed?. https://stackoverflow.com/questions/59575282/azure-devops-feed-with-pat-what-permissions-are-needed.
(3) Grant VSTS user access to create new branches - Stack Overflow. https://stackoverflow.com/questions/47516974/grant-vsts-user-access-to-create-new-branches.
(4) Keeping your dependencies updated with Azure Pipelines and Dependabot. https://techcommunity.microsoft.com/t5/azure-devops-blog/keeping-your-dependencies-updated-with-azure-pipelines-and/ba-p/3590020.
(5) Set Git branch security and permissions - Azure Repos. https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-permissions?view=azure-devops.
(6) undefined. https://dev.azure.com/.

tqzrk7eseegip2db5gekj32q6dun4ylvh3ewelrscxyydqeufqoq


