# DevOps: Synapse Deployment (WiP)

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/8695974f-8513-4de5-b7cc-d35a4cd89ce5" width="1000" />

## Use Case
* "We want to automate our Synapse deployment process using DevOps"
  
  <img src="https://github.com/richchapler/AzureSolutions/assets/44923999/780c08a6-57b4-484d-9310-6b269268054d" width="800" title="Snipped: December 4, 2023" />

## Proposed Solution
* **Starter Pipeline**: Create and test a minimum viable pipeline to demonstrate basic functionality
* **Branch Deployment**: Create an automated pipeline that: archives the QA branch, resets the QA branch, and 3) creates a pull reqeust
* **Synapse Resources**: Incorporate Synapse Integration Runtimes, Linked Services, Datasets and parameterized Pipelines

## Solution Requirements
* [DevOps](https://dev.azure.com/) Organization, Project, Repository (dedicated to Synapse), and Branches "DEV", "QA" and "PROD"
  * "Parallel Jobs" enabled
  * Project Settings >> Repos >> Respositories >> Security >> User Permissions... "{repository} Build Service"
    * Allow "Create branch"
    * Allow "Force push..."
* "DEV" Environment
  * [Synapse](Infrastructure_Synapse.md)
    * Git Configuration... Repository Type: Azure DevOps Git | Collaboration Branch "DEV"
  * On-prem machine with:
    * [SQL Server](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) configured for SQL Server Authentication and TCP/IP enabled
    * Database "dbDEV"
    * Table: `CREATE TABLE [dbo].[myTable]( [id] [int] NULL, [name] [varchar](64) NULL ) ON [PRIMARY]`
* "QA" Environment
  * [Synapse](Infrastructure_Synapse.md)
    * Git Configuration... Repository Type: Azure DevOps Git | Collaboration Branch "QA"
  * On-prem machine with:
    * [SQL Server](https://www.microsoft.com/en-us/sql-server/sql-server-downloads) configured for SQL Server Authentication and TCP/IP enabled
    * Database "dbQA"
    * Table: `CREATE TABLE [dbo].[myTable]( [id] [int] NULL, [name] [varchar](64) NULL ) ON [PRIMARY]`

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

-----

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

## Exercise 2: Branch Deployment
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

-----

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

-----

### Step 3: Confirm Success

Navigate to the DEV instance of Synapse Studio and confirm Git Configuration.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/b76554b7-52b6-4a86-ac84-38f37d3add2a" width="800" title="Snipped: December 5, 2023" />

<br>Now we will mimic the Synapse deployment process and confirm pipeline functionality.

##### Recurring Development
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

##### Pull Request
_Note: the primary human in this step will be a developer_

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/78061d6e-9e63-4a7b-893e-ba19c5650d83" width="800" title="Snipped: December 5, 2023" />

Click the branch dropdown and select "Create pull request".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d475a571-9e87-4bee-a52e-4d405a5849cd" width="800" title="Snipped: December 5, 2023" />

On the "New pull request" page, confirm destination "DEV", review default values and update as desired, and then click "Create".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/70b02643-52b6-430a-8d73-8d2eeb7a5d6f" width="800" title="Snipped: December 5, 2023" />

##### Automated Pipeline
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

## Exercise 3: Synapse Resources
In this exercise, we will incorporate Synapse Integration Runtimes, Linked Services, Datasets and parameterized Pipelines.

### Step 1: Integration Runtimes

#### DEV Instance

Navigate to the DEV instance of Synapse Studio and create a new working branch.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/7bdd4455-5b2b-4bce-9547-46db3a368e0f" width="800" title="Snipped: December 6, 2023" />

Navigate to "Integration runtimes" and click "+ New".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/c0d09842-bd91-494d-9c64-4091f23812a7" width="800" title="Snipped: December 6, 2023" />

On the "Integration runtime setup" popout, click "Azure, Self-Hosted" and then "Continue".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/5ae30f52-ba48-4f9f-bde2-dfabde391d96" width="800" title="Snipped: December 6, 2023" />

On the next "Integration runtime setup" popout, click "Self-Hosted" and then "Continue".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d4d95808-bbe5-4a79-af8c-265bea8c6389" width="800" title="Snipped: December 6, 2023" />

On the next "Integraton runtime setup" popout, enter a "Name" and then click "Create".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/fa30dfd2-a18a-4c47-9742-2b75cd7d0333" width="800" title="Snipped: December 6, 2023" />

On the next "Integration runtime setup" popout, click "Option 1: Express setup" >> "Click here to launch..."
<br>Install Microsoft Integration Runtime with the downloaded installer.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/22b42762-6943-44a1-bcf8-d3ae2ee2f8e9" width="500" title="Snipped: December 6, 2023" />

When installation is complete, return to Synapse Studio and confirm that the new Integration Runtime is "Status: Running".

-----

#### Pull Request

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/adce3cbd-61eb-41d6-8f4a-a9c4341053ae" width="800" title="Snipped: December 6, 2023" />

Click the branch dropdown, and click "Create pull request".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/bb8c85bd-119d-4a5a-9f3e-5fcd4bd8b4dc" width="800" title="Snipped: December 6, 2023" />

On the Azure DevOps >> "New pull request" page, review default settings and then click "Create".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0574f79f-08b0-41aa-9986-1f1dd6bbaffa" width="800" title="Snipped: December 6, 2023" />

On the Azure DevOps >> "Adding integrationRuntime..." pull request page, click "Complete".

_Note: We are skipping normal DevOps processes like review, approval, tie to work items for the purpose of demonstration only_

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/decd9f75-ba45-40dc-9a56-6ec5598aea08" width="800" title="Snipped: December 6, 2023" />

On the "Complete pull request" popout, review / update default settings and then click "Complete merge".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/86b49554-c1ff-4ac6-9694-38948e75d7f6" width="800" title="Snipped: December 6, 2023" />

Navigate to "Repos" >> "Files" and select the "DEV" branch.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/1c0febcf-d36b-4284-a362-c9029db774d6" width="800" title="Snipped: December 6, 2023" />

You should see a file in the "integrationRuntime" folder named "ir-DEV.json".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/5c560b1b-bc7c-4e5a-b77a-e748faedb337" width="800" title="Snipped: December 6, 2023" />

-----

#### Automated Pipeline

Navigate to the "Deploy_toQA" pipeline and click "Run". On the resulting "Run pipeline" popout, click "Run" again.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/66ce9b96-c986-4da1-8257-e061f11d93f2" width="800" title="Snipped: December 6, 2023" />

When the pipeline completes successfully, navigate to the pull request at Repos >> Pull requests >> Active.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/34f6ee96-dc0c-499e-9eee-74fca0203136" width="800" title="Snipped: December 6, 2023" />

Click "Complete" and on the resulting "Complete pull request" popout, click "Complete merge" and then confirm success.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/766f65d9-9ed6-4ab4-b963-d9ef271631b3" width="800" title="Snipped: December 6, 2023" />

-----

#### QA Instance

Navigate to the QA instance of Synapse Studio, select the QA branch, then Manage >> Integration >> Integration runtimes.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d43537ae-89c9-4132-8028-eb32a8156d6c" width="800" title="Snipped: December 6, 2023" />

You will see "ir-DEV" and note that it is "Status: Not Found".
<br>This is because an integration runtime can only be associated with a single instance of Synapse (in this case, "rchaplers-dev").
<br>Follow the instructions in Step 1 to instantiate a QA runtime.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/a39d164e-1ceb-48e8-9abf-fcee944f5b3d" width="800" title="Snipped: December 6, 2023" />

Navigate to DevOps >> "Repos" >> "Files" and select the "QA" branch. Click on "integrationRuntime".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/aaa316c8-e941-4f3d-ba72-330ef917d78d" width="800" title="Snipped: December 6, 2023" />

Roll-over file "ir-QA.json", click the vertical ellipses, and select "Download" from the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/2d9d4e16-873c-41a9-800b-81088e0b5d2e" width="800" title="Snipped: December 6, 2023" />

Switch to the "DEV" branch, click the vertical ellipses, and select "Upload file(s)" from the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/c1e53328-9f45-4069-ac1b-cafc65fe764b" width="800" title="Snipped: December 6, 2023" />

On the "Commit" popout, "Browse" to the downloaded "ir-QA.json" file, and then click "Commit".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/9c74e10a-a8a6-427c-9729-fba1a2696f51" width="800" title="Snipped: December 6, 2023" />

Run the "Deploy_toQA" pipeline and complete the resulting pull request.
<br>"ir-DEV" and "ir-QA" will exist in both environments, but only be "Status: Running" in the related environment.
<br>We will address use of the correct Integration Runtime in Exercise 4.

-----

### Step 2: Linked Services

#### DEV Instance

Navigate to the DEV instance of Synapse Studio, then Manage >> External Connections >> Linked Services, create a new working branch, and then click "+ New".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/c51e0d01-a1b3-405c-a4e5-ea4e9c6fa2cb" width="800" title="Snipped: December 7, 2023" />

On the "New linked service" popout, search for and select "SQL server", then click "Continue".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/0d18d59e-5c1b-469d-adaa-d7b1a992756b" width="800" title="Snipped: December 7, 2023" />

Complete the "New linked service" popout form, including:

Prompt | Entry
:----- | :-----
**Connect via integration runtime**... | ir-DEV
**Server name**... | localhost
**Database name**... | dbDEV
**Authentication type**... | SQL authentication

Click "Test Connection", confirm success and then click "Commit".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/882880ae-827d-4ee5-8945-e7d927096936" width="800" title="Snipped: December 7, 2023" />

When prompted "Linked service will be published immediately..." click "OK".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/ac025ed0-985f-4900-9230-f42e8fda8494" width="800" title="Snipped: December 7, 2023" />

Create and complete a pull request to move changes to the DEV branch.
<br>Then, trigger the "Deploy_toQA" pipeline in DevOps and complete the resulting pull request.

-----

#### QA Instance

_Note: Since it is not possible to parameterize the "Connect via integration runtime" reference in Synapse, we must create Linked Services and Integration Datasets for DEV, QA, and PROD environments and then parameterize Pipelines... to achieve this, we will mimic our creation of environmentally-specific Integration Runtimes_

Navigate to the QA instance of Synapse Studio, then Manage >> External Connections >> Linked Services and repeat the process to create a "dbQA" Linked Service.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d1f47a2e-9c44-4288-813d-32d35458c11f" width="800" title="Snipped: December 7, 2023" />

Navigate to DevOps >> "Repos" >> "Files" and select the "QA" branch. Click on "linkedService".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/1b575f73-43b3-4e59-8c77-17b79210723a" width="800" title="Snipped: December 7, 2023" />

Roll-over file "dbQA.json", click the vertical ellipses, and select "Download" from the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/3ba70440-ef6b-42ee-9c34-82e282c198aa" width="800" title="Snipped: December 7, 2023" />

Switch to the "DEV" branch, click the vertical ellipses, and select "Upload file(s)" from the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d224d141-2749-42db-9fb5-802d9e5be036" width="800" title="Snipped: December 7, 2023" />

On the "Commit" popout, "Browse" to the downloaded "dbQA.json" file, and then click "Commit".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/fc22b962-8ff2-40a1-acb0-3b789854e53b" width="800" title="Snipped: December 7, 2023" />

Run the "Deploy_toQA" pipeline and complete the resulting pull request. "dbDEV" and "dbQA" will exist in both environments.

-----

### Step 3: Datasets

#### DEV Instance

Navigate to the DEV instance of Synapse Studio and create a new working branch.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/29b6eacf-24c1-488c-98c7-16eef11a4594" width="800" title="Snipped: December 7, 2023" />

Navigate to "Data" >> "Linked", click "+" and then "Integration dataset" from the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/68312e4c-0115-490a-940e-fff03376d9fc" width="800" title="Snipped: December 7, 2023" />

On the "New integration dataset" popout, search for and select "SQL server", then click "Continue".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/51c0e586-80bb-47cf-89d3-aa2e73c8cc1d" width="800" title="Snipped: December 7, 2023" />

On the "Set properties" popout, select the "dbDEV" Linked Service, "myTable" table, then click "OK".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/e2f47dcd-d76a-4562-bc0c-5e13e8eb2b9a" width="800" title="Snipped: December 7, 2023" />

Click "Test Connection", confirm success and then click "Commit".
<br>Create and complete a pull request to move changes to the DEV branch.
<br>Then, trigger the "Deploy_toQA" pipeline in DevOps and complete the resulting pull request.

-----

#### QA Instance

_Note: Since it is not possible to parameterize the "Linked Service" reference in Synapse, we must create Linked Services and Integration Datasets for DEV, QA, and PROD environments and then parameterize Pipelines... to achieve this, we will mimic our creation of environmentally-specific Integration Runtimes_

Navigate to the QA instance of Synapse Studio, then Manage >> External Connections >> Linked Services and repeat the process to create a "dbQA" Linked Service.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d1f47a2e-9c44-4288-813d-32d35458c11f" width="800" title="Snipped: December 7, 2023" />

Navigate to DevOps >> "Repos" >> "Files" and select the "QA" branch. Click on "linkedService".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/1b575f73-43b3-4e59-8c77-17b79210723a" width="800" title="Snipped: December 7, 2023" />

Roll-over file "dbQA.json", click the vertical ellipses, and select "Download" from the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/3ba70440-ef6b-42ee-9c34-82e282c198aa" width="800" title="Snipped: December 7, 2023" />

Switch to the "DEV" branch, click the vertical ellipses, and select "Upload file(s)" from the resulting menu.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/d224d141-2749-42db-9fb5-802d9e5be036" width="800" title="Snipped: December 7, 2023" />

On the "Commit" popout, "Browse" to the downloaded "dbQA.json" file, and then click "Commit".

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/fc22b962-8ff2-40a1-acb0-3b789854e53b" width="800" title="Snipped: December 7, 2023" />

Run the "Deploy_toQA" pipeline and complete the resulting pull request. "dbDEV" and "dbQA" will exist in both environments.

-----

### Step 4: Parameterized Pipeline

LOREM IPSUM

-----
-----

#### Parameterization

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/5ca24da5-ff41-4868-9e1d-038f892b2cee" width="800" title="Snipped: December 7, 2023" />

On the "Parameters" tab, click "+ New" and complete the resulting form:

Prompt | Entry
:----- | :-----
**Name** | Environment
**Type** | String
**Default value** | DEV

Click the `{ }` icon in the upper-left of the UI to view the JSON code representation of the resource.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/26559417-d590-4ccd-bbc0-68b9026d6f5e" width="800" title="Snipped: December 7, 2023" />

Click on the "Database name" input and then click the "Add dynamic content" link.

<img src="https://github.com/richchapler/AzureSolutions/assets/44923999/fe1cfd85-4b68-47dc-9c7d-e09a1b260982" width="800" title="Snipped: December 7, 2023" />


```
db@{if(contains(linkedService().WorkspaceName,'dev'), 'DEV', 'QA')}

```

Re-enter password and tehn Test Connection



-----

**Congratulations... you have successfully completed this exercise**

-----
-----
