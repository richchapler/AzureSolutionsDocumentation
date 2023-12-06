# DevOps: Synapse Deployment Automation (WiP)

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/8695974f-8513-4de5-b7cc-d35a4cd89ce5" width="1000" />

## Use Case
* "We want to automate our Synapse deployment process using DevOps"
  
  <img src="https://github.com/richchapler/AzureSolutions/assets/44923999/780c08a6-57b4-484d-9310-6b269268054d" width="800" title="Snipped: December 4, 2023" />

## Proposed Solution
* Starter Pipeline: Create and test a minimum viable pipeline to demonstrate basic functionality
* "Branch Deployment" Pipeline
* "Integration Runtime" Pipeline
* TBD...

## Solution Requirements
* [DevOps](https://dev.azure.com/) Organization, Project, Repository (dedicated to Synapse), and Branches "DEV", "QA" and "PROD"
  * "Parallel Jobs" enabled
  * Project Settings >> Repos >> Respositories >> Security >> User Permissions... "{repository} Build Service"
    * Allow "Create branch"
    * Allow "Force push..."
* "DEV" Environment
  * [Synapse](Infrastructure_Synapse.md)
    * Git Configuration... Repository Type: Azure DevOps Git | Collaboration Branch "DEV"
* "QA" Environment
  * [Synapse](Infrastructure_Synapse.md)
    * Git Configuration... Repository Type: Azure DevOps Git | Collaboration Branch "QA"

_Note: We will instantiate additional resources in Exercise 3_

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

### Step 3: Confirm Success

Click "Run".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/fc6f9899-dfc4-4eab-9da5-361e33ee1b3b" width="800" title="Snipped: December 1, 2023" />

On the resulting "Run pipeline" pop-out, review default settings, check "Enable system diagnostics", and then click "Run".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/95b99284-33a1-4215-9391-1034dfc2a1ff" width="800" title="Snipped: December 1, 2023" />

Click on the "archiveQA" job.

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

### Step 2: Activate Trigger

Click the vertical ellipses in the upper-right of the Pipeline Edit screen and select "Triggers" from the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/264a1a79-3acb-4013-972a-ad16b9c1fec3" width="800" title="Snipped: December 5, 2023" />

On the resulting "Deploy_toQA" page, "Triggers" tab, select the item in the "Continuous integration" grouping.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/bf7ee994-ea15-4c9f-8e4a-d9d7a4e46e38" width="800" title="Snipped: December 5, 2023" />

Complete the popout form, including:

Prompt | Entry
:----- | :-----
**Override the YAML continuous integration trigger**... | Checked
**Enable continuous integration**... | Selected
**Branch filters** >> Type... | DEV

Click the "Save & queue" dropdown and select "Save" from the resulting menu. "Save" again on the resulting popup.

-----

SUPPORT CASE PENDING RE: TRIGGERS

### Step 3: Confirm Success

Navigate to the DEV instance of Synapse Studio and confirm Git Configuration.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b76554b7-52b6-4a86-ac84-38f37d3add2a" width="800" title="Snipped: December 5, 2023" />

<br>Now we will mimic the Synapse deployment process and confirm pipeline functionality.

##### Process Step 1: Recurring Development
_Note: the primary human in this step will be a developer_

Click the branch dropdown and select "+ New branch".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/c959b4fc-23f4-43f0-a455-df65796661d9" width="800" title="Snipped: December 5, 2023" />

Complete the "Create a new branch" popup and click "Create".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/1d35684d-c499-4b25-a008-135a60fcc88d" width="800" title="Snipped: December 5, 2023" />

Navigate to Synapse >> Develop, click "+", and then select "SQL script" from the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d7879cb6-8b91-4b91-9f11-92935254b2a9" width="800" title="Snipped: December 5, 2023" />

On the resulting "SQL script 1" tab, paste the following T-SQL:
```
PRINT 'Hello, World';
```

Our goal is to capture a simple change in our branch, but run the logic if you are interested. Click "Commit all".

##### Process Step 2: Pull Request
_Note: the primary human in this step will be a developer_

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/78061d6e-9e63-4a7b-893e-ba19c5650d83" width="800" title="Snipped: December 5, 2023" />

Click the branch dropdown and select "Create pull request".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d475a571-9e87-4bee-a52e-4d405a5849cd" width="800" title="Snipped: December 5, 2023" />

On the "New pull request" page, confirm destination "DEV", review default values and update as desired, and then click "Create".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/70b02643-52b6-430a-8d73-8d2eeb7a5d6f" width="800" title="Snipped: December 5, 2023" />

##### Process Step 3: Automated Pipeline
_Note: the primary human in this step will be the Deployment Manager... PENDING SUPPORT CASE RE: TRIGGER_

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/6d19881f-69ea-4a50-9f7e-3193356837e0" width="800" title="Snipped: December 5, 2023" />

Continuing on the "Adding sqlscript: SQL script 1" pull request page, review Files, etc. and then click "Complete".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/59b179af-b6a8-4f91-9c8d-011c29360ece" width="800" title="Snipped: December 5, 2023" />

On the resulting "Complete pull request" pop-out, confirm default settings and then click "Complete merge".

-----
THE FOLLOWING STEPS ARE ONLY NECESSARY UNTIL SUPPORT HELPS ME FIGURE OUT HOW TO GET THE TRIGGER TO AUTOMATICALLY START THE PIPELINE UPON COMMIT TO DEV BRANCH

<br>Navigate to the "Deploy_toQA" pipeline.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/86278a91-d839-4831-9c1b-5a6cca9d6042" width="800" title="Snipped: December 5, 2023" />

Click "Run".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/92debc4b-d5cd-4d07-82a3-69bd01f44630" width="800" title="Snipped: December 5, 2023" />

On the resulting "Run pipeline" pop-out, review default settings, check "Enable system diagnostics", and then click "Run".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0dd1b512-6432-44fd-842c-9ca549144eff" width="800" title="Snipped: December 5, 2023" />

Confirm successful processing of the three jobs.

-----

**Congratulations... you have successfully completed this exercise**

-----
-----

## Exercise 3: Integration Runtimes (WiP)
In this exercise, we will add Synapse Integration Runtime handling to the automated pipeline processing.

### Step 1: Instantiate Resources

Instantiate the following resources:
* "DEV" Environment
  * On-prem machine with:
    * [SQL Server](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) configured for SQL Server Authentication
    * Database "dbDEV"
  * [Synapse](Infrastructure_Synapse.md)
    * Integration Runtime "irDEV" installed on on-prem SQL Server  
* "QA" Environment
  * On-prem machine with:
    * [SQL Server](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) configured for SQL Server Authentication
    * Database "dbQA"
  * [Synapse](Infrastructure_Synapse.md)
    * Integration Runtime "irQA" installed on on-prem SQL Server






LOREM IPSUM

-----

**Congratulations... you have successfully completed this exercise**

-----
-----

## Exercise 4: Linked Services (WiP)
In this exercise, we will add Synapse Linked Service handling to the automated pipeline processing.

### Step 1: Instantiate Resources

Instantiate the following resources:
* "DEV" Environment
  * [Synapse](Infrastructure_Synapse.md)
    * Linked Service "lsDEV" using integration runtime "irDEV" and connected to database "dbDEV"
* "QA" Environment
  * [Synapse](Infrastructure_Synapse.md)
    * Integration Runtime "irQA" installed on on-prem SQL Server  

LOREM IPSUM

-----

**Congratulations... you have successfully completed this exercise**

-----
-----
