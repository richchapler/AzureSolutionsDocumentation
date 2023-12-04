# DevOps: Synapse Deployment Automation (WiP)

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/8695974f-8513-4de5-b7cc-d35a4cd89ce5" width="1000" />

## Use Case
* "We want to automate our Synapse deployment process (depicted below)"
  
  <img src="https://github.com/richchapler/AzureSolutions/assets/44923999/780c08a6-57b4-484d-9310-6b269268054d" width="800" title="Snipped: December 4, 2023" />

## Proposed Solution
* Starter Pipeline: Create and test a minimum viable pipeline to demonstrate basic functionality
* "Branch Deployment" Pipeline
* "Integration Runtime" Pipeline
* TBD...

## Solution Requirements
* [DevOps](https://dev.azure.com/) Organization, Project, Repository (dedicated to Synapse), and Branches "DEV", "QA" and "PROD"
  * "Parallel Jobs" enabled
  * Project Settings >> Repos >> Respositories >> Security >> User Permissions... "{repository} Build Service" >> Allow "Create branch"
  * Project Settings >> Repos >> Respositories >> Security >> User Permissions... "{repository} Build Service" >> Allow "Force push..."
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
In this exercise, we will create and test a minimum viable pipeline to demonstrate basic functionality.

### Step 1: Create Service Connection

Navigate to Azure DevOps >> Project >> Project Settings >> Pipelines >> Service Connections.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/63837cf5-e23e-4b11-a940-9f4e9ca828b2" width="800" title="Snipped: November 30, 2023" />

Click "Create service connection".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/82b143c8-4347-4ce4-b327-87143c3f3b4f" width="800" title="Snipped: November 30, 2023" />

On the "New service connection" pop-out, select "Azure Resource Manager" and then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/eeed2c00-86a0-4394-9393-2ad994f19bac" width="800" title="Snipped: November 30, 2023" />

On the "New Azure service connection" pop-out, select "Workload Identity federation (automatic)" and then click "Next".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/13bd959f-5fbe-49d9-9a3c-a16ed892ed32" width="800" title="Snipped: November 30, 2023" />

Complete the resulting "New Azure service connection" pop-out, and then click "Save".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b69f5927-4d11-4596-8f02-410f18882356" width="800" title="Snipped: November 29, 2023" />

-----

### Step 2: Create Pipeline

Navigate to Azure DevOps >> Pipelines >> Pipelines.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/7648bab1-5200-4ac2-ac1e-82ce5a1ce7b0" width="800" title="Snipped: November 30, 2023" />

Click "+ Create Pipeline".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/ac5d27b4-4bcd-41e7-8bc4-b9cc75d7c8f3" width="800" title="Snipped: November 30, 2023" />

On the "Connect" tab, click "Azure Repos Git".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/fbc1ea37-1460-47c3-8fe6-0dd93597c7a5" width="800" title="Snipped: November 30, 2023" />

On the "Select" tab, click to select your repository.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b9e1b932-6336-4ec2-9580-34b8c887fec6" width="800" title="Snipped: November 30, 2023" />

On the "Configure" tab, click "Starter pipeline".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b7a8fcb3-923a-439f-8ca9-b80a33fa3486" width="800" title="Snipped: November 30, 2023" />

On the "Review" tab, click to dropdown "Save and run", and then click "Save".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/9a3a3239-1ef7-4e62-b088-61b308d35e28" width="800" title="Snipped: November 30, 2023" />

On the resulting pop-out, click "Save".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2b200aa8-c298-4070-b2f8-b186c63d675a" width="800" title="Snipped: November 30, 2023" />

Click "Edit".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/4d3fefb4-2a0d-479e-947e-90aeb684b74f" width="800" title="Snipped: November 30, 2023" />

Replace the default YAML with:

```
trigger:
  branches:
    include:
    - DEV

pool:
    vmImage: 'windows-latest'

variables: {o: "https://dev.azure.com/rchapler", p: "devops", r: "synapse", db: "DEV", qb: "QA", pb: "PROD"}

jobs:
- job: archiveQA
  steps:
  - script: echo $(System.AccessToken) | az devops login
    displayName: 'Login to DevOps'
  - task: AzureCLI@2
    displayName: 'Archive QA Branch'
    inputs:
      azureSubscription: "AzureSubscription"
      scriptType: 'pscore'
      scriptLocation: 'inlineScript'
      inlineScript: |
        echo "***** Copying QA Branch to ../archive/QA-{timestamp}"
        $qbid = az repos ref list --org $(o) -p $(p) -r $(r) --filter "heads/$(qb)" | ConvertFrom-Json | Select-Object -ExpandProperty objectId
        $PST = [System.TimeZoneInfo]::ConvertTimeFromUtc([DateTime]::UtcNow, [System.TimeZoneInfo]::FindSystemTimeZoneById("Pacific Standard Time"))
        $dt = Get-Date $PST -Format "yyyyMMdd-HHmmss"
        az repos ref create --name "refs/heads/archive/$(qb)-$dt" --organization $(o) --project $(p) --repository $(r) --object-id $qbid
```

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/037b8941-a975-4e88-9a11-cd85a7972e33" width="800" title="Snipped: December 1, 2023" />

#### Logic Explained

* `archiveQA` is the main job
* `Login to DevOps` logs into Azure DevOps using the System Access Token
* `Archive QA Branch` uses the Azure CLI to archive the QA branch by:
   - Getting the current date and time
   - Listing all the branches in the repository {i.e., `$branches = az repos ref list...`}
   - Finding the object identifier of the current QA branch {i.e., `$oid = $branches | Where-Object {$_.name -eq "refs/heads/$b"}...`}
   - Creating a new branch based on the current QA branch {i.e., `az repos ref create...`}

Click "Save".

#### Confirm Success

Click "Run".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/fc6f9899-dfc4-4eab-9da5-361e33ee1b3b" width="800" title="Snipped: December 1, 2023" />

On the resulting "Run pipeline" pop-out, review default settings, check "Enable system diagnostics", and then click "Run".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/95b99284-33a1-4215-9391-1034dfc2a1ff" width="800" title="Snipped: December 1, 2023" />

Click on the "Deploy_toQA" job.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6a745dad-3fa5-40b6-88b0-d91edde33288" width="800" title="Snipped: December 1, 2023" />

Confirm job run success and then navigate to DevOps >> Repos >> Branches and click on the "All tab".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/4559ae96-5afc-4a3b-8e57-67c955cbaa9c" width="800" title="Snipped: December 1, 2023" />

You can expect to see a new branch named "QA-{datetime}".

-----

**Congratulations... you have successfully completed this exercise**

-----
-----

## Exercise 2: QA Deployment (WiP)
In this exercise, we will create an automated pipeline triggered by a pull request to DEV, that completes the following three tasks:
1) **Archive QA Branch**... make a copy of the current QA branch to a timestamped branch in the "archive" folder
2) **Reset QA Branch**... force copy PROD branch to QA branch
3) **Create a Pull Request**... from DEV branch to QA branch

### Step 1: Update Pipeline

Navigate to Azure DevOps >> Pipelines >> Pipelines and replace the existing YAML with:

```
trigger:
  branches:
    include:
    - DEV

pool:
    vmImage: 'windows-latest'

variables: {o: "https://dev.azure.com/rchapler", p: "devops", r: "synapse", db: "DEV", qb: "QA", pb: "PROD"}

jobs:
- job: archiveQA
  steps:
  - script: echo $(System.AccessToken) | az devops login
    displayName: 'Login to DevOps'
  - task: AzureCLI@2
    displayName: 'Archive QA Branch'
    inputs:
      azureSubscription: "AzureSubscription"
      scriptType: 'pscore'
      scriptLocation: 'inlineScript'
      inlineScript: |
        echo "***** Copying QA Branch to ../archive/QA-{timestamp}"
        $qbid = az repos ref list --org $(o) -p $(p) -r $(r) --filter "heads/$(qb)" | ConvertFrom-Json | Select-Object -ExpandProperty objectId
        $PST = [System.TimeZoneInfo]::ConvertTimeFromUtc([DateTime]::UtcNow, [System.TimeZoneInfo]::FindSystemTimeZoneById("Pacific Standard Time"))
        $dt = Get-Date $PST -Format "yyyyMMdd-HHmmss"
        az repos ref create --name "refs/heads/archive/$(qb)-$dt" --organization $(o) --project $(p) --repository $(r) --object-id $qbid

- job: resetQA # possible using Git commands, but not CLI/PowerShell or UI
  dependsOn: archiveQA
  steps:
  - task: AzureCLI@2
    displayName: 'Reset QA from PROD'
    inputs:
      azureSubscription: "AzureSubscription"
      scriptType: 'pscore'
      scriptLocation: 'inlineScript'
      inlineScript: |
        echo "***** Resetting QA from PROD"
        git -c http.extraheader="AUTHORIZATION: bearer $(System.AccessToken)" fetch
        git -c http.extraheader="AUTHORIZATION: bearer $(System.AccessToken)" checkout QA
        git -c http.extraheader="AUTHORIZATION: bearer $(System.AccessToken)" reset --hard origin/PROD
        git -c http.extraheader="AUTHORIZATION: bearer $(System.AccessToken)" push origin QA --force

- job: pullrequestDEVtoQA
  dependsOn: resetQA
  steps:
  - script: echo $(System.AccessToken) | az devops login
    displayName: 'Login to DevOps'
  - task: AzureCLI@2
    displayName: 'Create Pull Request'
    inputs:
      azureSubscription: "AzureSubscription"
      scriptType: 'pscore'
      scriptLocation: 'inlineScript'
      inlineScript: |
        echo "***** Creating Pull Request (from DEV to QA)"
        az repos pr create --title "DEV >> QA" --organization $(o) --project $(p) --repository $(r) --source-branch "$(db)" --target-branch "$(qb)"
```

#### Logic Explained

* `trigger` specifies that the pipeline should be triggered when there are changes to the `DEV` branch
* `pool` specifies that the latest Windows virtual machine image should be used for the pipeline
* `variables` defines several variables that are used throughout the pipeline
* `jobs` contains the jobs that make up the pipeline; each job has a series of steps that are executed in order
  * `archiveQA` logs into DevOps and archives the QA branch by copying the QA branch to an archive folder with a timestamp
  * `resetQA` resets the QA branch from the PROD branch using Git commands to:
    1) fetch the branches
    2) checkout the QA branch
    3) reset the QA branch to match the PROD branch
    4) force push the changes to the QA branch
  * `pullrequestDEVtoQA` creates a pull request from the DEV branch to the QA branch
