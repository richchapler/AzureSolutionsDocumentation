# Data Explorer Delta  

...using Azure DevOps  

## Resource Requirements

### Azure

Instantiate the following resources in Azure:

- Application Registration: `{prefix}ar`
- Data Explorer Clusters
  - Development: `{prefix}dec-dev.westus.kusto.windows.net`
  - Production: `{prefix}dec-prd.westus.kusto.windows.net`
- Data Explorer Databases
  - Development: `{prefix}ded-dev`
  - Production: `{prefix}ded-prd`
- Key Vault (shared): `{prefix}kv` with the following secrets:
  - `AZURE-TENANT-ID`
  - `AZURE-SUBSCRIPTION-ID`
  - `AZURE-CLIENT-ID`
  - `AZURE-CLIENT-SECRET`

_Note: Describing Azure resources first because at least Key Vault will be a necessary dependency_

### On-Prem Machine

Prepare an on-prem machine (physical or virtual) and install the following resources in order:

1. [PowerShell](https://richchapler.github.io/AzureSolutionsDocumentation/artifacts/PowerShell.html) - Required for executing scripts and ensuring compatibility with Azure CLI  
2. **Azure CLI** - Ensures availability for pipeline authentication and command execution  
3. **Kusto Extension** - Required for querying Data Explorer  
4. **Microsoft.Azure.Kusto.Data** - Installed via NuGet and required for Kusto client operations and authentication  
5. **Git** - Needed for repository tracking and pipeline commits  
6. **Self-Hosted Agent** - Registered in Azure DevOps and required for executing the pipeline on a dedicated machine  
7. **Service Connection** - Configured in Azure DevOps with correct permissions to authenticate and access necessary Azure resources

_Note: Items without links are detailed in the sections below_

------------------------- -------------------------

## Azure CLI  

Azure CLI is required for pipeline execution, authentication, and querying Data Explorer.  

### Verify Azure CLI Installation

Open PowerShell as an administrator and execute the following command:  

```powershell
where.exe az
```

- If a path is returned, Azure CLI is installed
- If no output is returned, install Azure CLI 

### Install and Upgrade Azure CLI

Execute the following command to install Azure CLI:  

```powershell
winget install --id Microsoft.AzureCLI --source winget --accept-package-agreements --accept-source-agreements
```

Restart the PowerShell terminal and verify:

```powershell
where.exe az
```

If Azure CLI is successfully installed, update it:  


```powershell
az upgrade
```

If `az upgrade` fails, use `winget`:  


```powershell
winget upgrade --id Microsoft.AzureCLI
```

Restart the PowerShell terminal and verify:  


```powershell
where.exe az
```

### Ensure Azure CLI is Recognized  

If `az` is installed but not recognized, manually add it to `PATH`:  


```powershell
$AzPath = "C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\wbin"
$envPath = [System.Environment]::GetEnvironmentVariable("Path", "Machine")
if ($envPath -notlike "*$AzPath*") { [System.Environment]::SetEnvironmentVariable("Path", "$envPath;$AzPath", "Machine") }
```

Restart the PowerShell terminal and verify again:  


```powershell
where.exe az
```

If `az` is still not recognized, restart the machine.

------------------------- -------------------------

## Kusto Extension  

Check if the Kusto extension is installed:  

```powershell
az extension list --output table
```

If `kusto` is not listed, install it:  

```powershell
az extension add --name kusto
```

Restart the PowerShell terminal and verify installation:  

```powershell
az extension list --output table
az kusto -h
```

If `kusto` is still not recognized, restart the PowerShell terminal and check again. If the issue persists, restart the machine.

------------------------- -------------------------

## Microsoft.Azure.Kusto.Data

 ...via NuGet  

### 1. Disable VPN (if applicable)  

If using a VPN, turn it off before installation.  

### 2. Verify Network Connectivity  

```powershell
Test-NetConnection -ComputerName api.nuget.org -Port 443
Test-NetConnection -ComputerName www.nuget.org -Port 443
```

### 2.1 (Conditional) Update DNS if Test-NetConnection fails  

```powershell
netsh interface ip set dns "Wi-Fi" static 8.8.8.8
ipconfig /flushdns
```

### 2.1.1 (Conditional) Restart the network adapter  

```powershell
Disable-NetAdapter -Name "Wi-Fi" -Confirm:$false
Start-Sleep -Seconds 5
Enable-NetAdapter -Name "Wi-Fi"
```

### 2.1.2 (Conditional) Verify connectivity again  

```powershell
Test-NetConnection -ComputerName api.nuget.org -Port 443
Test-NetConnection -ComputerName www.nuget.org -Port 443
```

### 3. Install NuGet CLI  

#### 3.1 Ensure the Installation Directory Exists  

```powershell
New-Item -ItemType Directory -Path C:\KustoSDK -Force
```

#### 3.1.1 Verify that the directory was created  

```powershell
Test-Path C:\KustoSDK
```

If `False`, manually create the directory and check permissions.

#### 3.2 Download NuGet CLI  

```powershell
Invoke-WebRequest -Uri https://dist.nuget.org/win-x86-commandline/latest/nuget.exe -OutFile C:\KustoSDK\nuget.exe
```

#### 3.2.1 Verify that nuget.exe was downloaded  

```powershell
Test-Path C:\KustoSDK\nuget.exe
```

### 3.2.2 (Conditional) If False, troubleshoot  

Check internet connectivity:  

```powershell
Test-NetConnection api.nuget.org -Port 443
```

Check file permissions for `C:\KustoSDK`  

Check security policies blocking `Invoke-WebRequest`  

If security policies are blocking execution, unblock the file:  

```powershell
Unblock-File -Path C:\KustoSDK\nuget.exe
```

### 3.3 Verify NuGet CLI Execution  

```powershell
C:\KustoSDK\nuget.exe help
```

### 3.3.1 (Conditional) If nuget.exe is not recognized, add C:\KustoSDK to PATH  

```powershell
$NuGetPath = "C:\KustoSDK"
$envPath = [System.Environment]::GetEnvironmentVariable("Path", "Machine")
if ($envPath -notlike "*$NuGetPath*") { [System.Environment]::SetEnvironmentVariable("Path", "$envPath;$NuGetPath", "Machine") }
```

### 3.3.2 Restart the PowerShell terminal and verify  

```powershell
where.exe nuget
```

### 3.3.3 (Conditional) If nuget.exe is still not recognized  

Check if `nuget.exe` is found under `C:\KustoSDK`:  

```powershell
where.exe /R C:\KustoSDK nuget.exe
```

If `nuget.exe` exists but is not recognized, manually add it to the System PATH:  

1. Open System Properties (`sysdm.cpl` in `Run` dialog)  

2. Go to Advanced → Environment Variables  

3. Under System Variables, locate Path and click Edit  

4. Click New, add:  

   ```
   C:\KustoSDK
   ```

5. Click OK on all dialogs  

6. Restart PowerShell and try:  

```powershell
where.exe nuget
```

If `nuget.exe` is still not recognized, restart the machine and try again:  

```powershell
shutdown /r /t 0
```

### 4. Install Required Packages  

```powershell
C:\KustoSDK\nuget.exe install Microsoft.Azure.Kusto.Data -OutputDirectory C:\KustoSDK
C:\KustoSDK\nuget.exe install Microsoft.Azure.Kusto.Ingest -OutputDirectory C:\KustoSDK
C:\KustoSDK\nuget.exe install Newtonsoft.Json -OutputDirectory C:\KustoSDK
C:\KustoSDK\nuget.exe install Microsoft.IdentityModel.Tokens -OutputDirectory C:\KustoSDK
```

### 5. Locate and Load the DLLs  

#### 5.1 Verify Installed Packages  

Check that the required packages were installed successfully:  

```powershell
Get-ChildItem -Path "C:\KustoSDK" -Recurse -Filter "*.dll"
```

#### 5.1.1 (Conditional) If `Kusto.Data.dll` is Missing, Reinstall Required Packages  

```powershell
C:\KustoSDK\nuget.exe install Microsoft.Azure.Kusto.Data -OutputDirectory C:\KustoSDK
C:\KustoSDK\nuget.exe install Microsoft.Azure.Kusto.Ingest -OutputDirectory C:\KustoSDK
C:\KustoSDK\nuget.exe install Newtonsoft.Json -OutputDirectory C:\KustoSDK
C:\KustoSDK\nuget.exe install Microsoft.IdentityModel.Tokens -OutputDirectory C:\KustoSDK
```

#### 5.2 Locate the Correct Path for `Kusto.Data.dll`  

```powershell
Get-ChildItem -Path "C:\KustoSDK" -Recurse -Filter "Kusto.Data.dll" | Select-Object FullName
```

If `Kusto.Data.dll` is missing, manually inspect `C:\KustoSDK` to ensure the expected folder structure exists.

#### 5.3 Update the DLL Path If Needed  

```powershell
$KustoDllPath = (Get-ChildItem -Path "C:\KustoSDK" -Recurse -Filter "Kusto.Data.dll").FullName
if ($KustoDllPath) {
    Add-Type -Path $KustoDllPath
} else {
    Write-Host "ERROR: Kusto.Data.dll not found in C:\KustoSDK. Verify installation."
    exit 1
}
```

#### 5.3.1 (Conditional) If `Kusto.Data.dll` is Still Missing  

- Ensure NuGet successfully installed the libraries  
- Check for connectivity issues with NuGet (`Test-NetConnection api.nuget.org -Port 443`)  
- Verify `C:\KustoSDK` has the correct structure  

If needed, force a reinstall of `Microsoft.Azure.Kusto.Data`:  

```powershell
Remove-Item -Recurse -Force C:\KustoSDK\Microsoft.Azure.Kusto.Data*
C:\KustoSDK\nuget.exe install Microsoft.Azure.Kusto.Data -OutputDirectory C:\KustoSDK
```

Then, retry locating the DLL:  

```powershell
Get-ChildItem -Path "C:\KustoSDK" -Recurse -Filter "Kusto.Data.dll" | Select-Object FullName
```

Once the correct path is confirmed, re-run `Add-Type` with the verified DLL path.

------------------------- -------------------------

## Git  

Git must be installed and configured before setting up the DevOps agent. This ensures that the pipeline machine can execute Git commands, but repository tracking will be configured later after the self-hosted agent is set up.

### Already Installed?

```powershell
where.exe git
```

### If not, install...

Install Git using `winget`:  

```powershell
winget install --id Git.Git --source winget --accept-package-agreements --accept-source-agreements
```

### 3. Verify Installation  

Restart the PowerShell terminal and check:  

```powershell
git --version
```

If Git is installed correctly, it will return the installed version.

### 4. Ensure Git is in `PATH`  

If `git` is installed but not recognized, manually add it to the system PATH:

```powershell
$GitPath = "C:\Program Files\Git\bin"
$envPath = [System.Environment]::GetEnvironmentVariable("Path", "Machine")
if ($envPath -notlike "*$GitPath*") { [System.Environment]::SetEnvironmentVariable("Path", "$envPath;$GitPath", "Machine") }
```

Restart the PowerShell terminal and verify:

```powershell
where.exe git
```

------------------------- -------------------------

## Azure DevOps

Before setting up the self-hosted agent and pipelines, Azure DevOps must be fully configured.

### 1. Create an Azure DevOps Organization

1. Navigate to [Azure DevOps](https://dev.azure.com/)
2. Click "Create new organization"
3. Sign in with your Microsoft account
4. Follow the prompts and name the organization (e.g., `"rchapler"`)
5. Select a region close to your Azure resources
6. Click "Continue"

------

### 2. Create a New DevOps Project

1. Once inside the DevOps organization, click "New project"
2. Provide a name (e.g., `"DataExplorer_Delta"`)
3. Set visibility to `"Private"`
4. Click "Create"

------

### 3. Create and Clone the Git Repository

1. Inside the project, go to `"Repos"`
2. Click `"Initialize with a README"` (if not already created)
3. Click `"Clone"` → Copy the HTTPS URL

#### Choose a Suitable Directory

Decide on a location for storing repositories:

```powershell
New-Item -ItemType Directory -Path C:\Projects -Force 
cd C:\Projects
```

Alternatively, use the user’s home directory:

```powershell
cd $HOME
New-Item -ItemType Directory -Path "$HOME\Projects" -Force
cd "$HOME\Projects"
```

#### Clone the Repository

```sh
git clone https://InternationalMotors@dev.azure.com/InternationalMotors/DataExplorerDelta/_git/DataExplorerDelta
cd DataExplorerDelta
```

------

### 4. Configure the Repository for Pipeline Execution

1. Add a `.gitignore` file to exclude unnecessary files

2. Create a new branch (optional)

   ```sh
   git checkout -b dev
   git push -u origin dev
   ```

------

### 5. Configure Azure DevOps Repository Permissions

For the pipeline to commit and push generated `.kql` files back to the repository, ensure the **DevOps Build Service** has the correct permissions.

#### Verify Build Service Permissions

1. Navigate to **Azure DevOps** → **Project Settings** → **Repositories**

2. Select the repository `"DataExplorer_Delta"`

3. Go to the **Security** tab

4. Locate `"DataExplorer_Delta Build Service (rchapler)"`

5. Set the following permissions to 

   ```
   "Allow"
   ```

   - `Contribute`
   - `Create branch`
   - `Contribute to pull requests`
   - `Bypass policies when pushing` (if necessary)

6. If a `"Save"` button appears, click it

7. If no `"Save"` button appears, refresh the page to confirm the permissions were applied

------

### 6. Validate Repository Permissions

After setting permissions, validate push access by running the following on the **agent machine**:

```sh
git config --global user.email "pipeline@devops.com"
git config --global user.name "Azure DevOps Pipeline"

git checkout main
git pull origin main
echo "Test Commit" > test_file.txt
git add test_file.txt
git commit -m "Test commit from pipeline"
git push origin main
```

- If authentication fails, verify that `System.AccessToken` is being used correctly in the pipeline
- If permission errors occur, recheck the repository settings

------------------------- -------------------------

## Self-Hosted Agent  

### Create and Expand a Personal Access Token (PAT)  

1. Go to User Settings >> [DevOps Tokens](https://dev.azure.com/rchapler/_usersSettings/tokens).  
2. Click "New Token".  
3. Set:  
   - "Token Name": `SelfHostedAgentToken`  
   - "Expiration": 90 days or more  
   - "Scopes": `"Build (Read & Execute)"`  
4. Click "Create" and copy the PAT.  

### Expand the PAT Permissions  

- Enable `"Agent Pools (Read & Manage)"` and `"Project and Team (Read & Write)"`.  

### Create an Agent Pool and Assign Permissions  

1. Go to Organization Settings >> [Agent Pools](https://dev.azure.com/rchapler/_settings/agentpools).  
2. Click "Add Pool".  
3. Set:  
   - "Pool Name": `SelfHostedPool`  
   - Enable "Auto-provision in all projects".  
4. Confirm SelfHostedPool >> Security, "Administrator" role on your personal DevOps account.  

### Install and Configure the Self-Hosted Agent

Navigate to [Microsoft Azure DevOps Agents](https://github.com/microsoft/azure-pipelines-agent/releases) and download the latest `vsts-agent-win-x64-{version}.zip` to `C:\Users\rchapler\Downloads\`

Note the version number for later use.

Open PowerShell as an Administrator and execute the following command:

```powershell
Test-Path C:\AzureDevOpsAgent
```

Download and extract:  

```powershell
Expand-Archive -Path "$HOME\Downloads\vsts-agent-win-x64-{version}.zip" -DestinationPath "C:\AzureDevOpsAgent"
```

Run the configuration:  

```powershell
C:\AzureDevOpsAgent\config.cmd
```

In the resulting "Azure Pipelines" interface, enter the following values:  

```
Enter agent pool (press enter for default) > SelfHostedPool
Enter agent name (press enter for DESKTOP-LF1SRQU) >
..
Enter work folder (press enter for _work) >
..
Enter run agent as service? (Y/N) (press enter for N) > Y
Enter enable SERVICE_SID_TYPE_UNRESTRICTED for agent service (Y/N) (press enter for N) > Y
Enter User account to use for the service (press enter for NT AUTHORITY\NETWORK SERVICE) >
..
Enter whether to prevent service starting immediately after configuration is finished? (Y/N) (press enter for N) > Y
```

------------------------- -------------------------

## Service Connection  

### Create a New Azure Service Connection  

1. Go to "Azure DevOps" → "Project Settings" → "Service Connections".  

2. Click "New service connection" → "Azure Resource Manager".  

3. Select:  
   - "Identity Type" → `"App Registration (Automatic)"`  
   - "Scope Level" → `"Subscription"`  

4. Set "Service Connection Name" (e.g., `AzureServiceConnection`)

This will create a new Application Registration for the Service Connection {e.g., `rchapler-DataExplorer_Delta-GUID}`... the generated name will match your configuration}

### Grant DevOps Access to Key Vault  

- In Azure Portal, go to Key Vault → "Access control (IAM)"  

- Assign the following roles to the Application Registration created for the Service Connection:

  - Key Vault Reader
  - Key Vault Secrets User

- Click Save

  ----

### Create a Variable Group in Azure DevOps  

- Go to Azure DevOps → Pipelines → Library  
- Click "New Variable Group" and name it `Secrets`  
- Enable "Link secrets from an Azure key vault as variables"  
- Select the Azure subscription and the Key Vault {prefix}kv  
- Click "Authorize" to allow Azure Pipelines to set the necessary permissions or manually apply them in the Azure portal  
- Click "+ Add" and select the following Key Vault Secrets: 
  - `AZURE-TENANT-ID`
  - `AZURE-SUBSCRIPTION-ID`
  - `AZURE-CLIENT-ID`
  - `AZURE-CLIENT-SECRET` 

- Click "Save"  

## Pipeline: `DataExplorer_Capture.yml`

```yaml
trigger: none

pool:
  name: SelfHostedPool

variables:
- group: Secrets
- name: Cluster
  value: "{prefix}dec-dev.westus.kusto.windows.net"
- name: Database
  value: "{prefix}ded-dev"

jobs:
- job: RunPipeline
  displayName: "Data Explorer Capture"
  steps:
    - checkout: self
      persistCredentials: true

    - task: AzureCLI@2
      displayName: "Task: Authenticate"
      name: GetToken
      inputs:
        azureSubscription: "AzureServiceConnection"
        scriptType: "pscore"
        scriptLocation: "inlineScript"
        inlineScript: |
          $TenantId = $env:AZURE_TENANT_ID
          $SubscriptionId = $env:AZURE_SUBSCRIPTION_ID
          $ClientId = $env:AZURE_CLIENT_ID
          $ClientSecret = $env:AZURE_CLIENT_SECRET

          Write-Host "Starting Azure login..."
          az login --service-principal --username "$ClientId" --password "$ClientSecret" --tenant "$TenantId" --only-show-errors
          az account set --subscription "$SubscriptionId" --only-show-errors

          Write-Host "Retrieving Azure Data Explorer access token..."
          $Token = az account get-access-token --resource "https://$(Cluster)" --query accessToken -o tsv --only-show-errors

          if ([string]::IsNullOrEmpty($Token)) {
              Write-Host "ERROR: Failed to retrieve access token."
              exit 1
          }

          Write-Host "Access token retrieved successfully."
          echo "##vso[task.setvariable variable=ADO_TOKEN]$Token"

          $TokenPath = "$env:AGENT_TEMPDIRECTORY\ado_token.txt"
          $Token | Out-File -FilePath $TokenPath -Encoding utf8
          Write-Host "Token written to: $TokenPath"

    - task: AzureCLI@2
      displayName: "Task: Verify Configuration"
      inputs:
        azureSubscription: "AzureServiceConnection"
        scriptType: "pscore"
        scriptLocation: "inlineScript"
        inlineScript: |
          Write-Host "Starting configuration verification..."

          $Database = "$(Database)"
          $TokenPath = "$env:AGENT_TEMPDIRECTORY\ado_token.txt"

          if (Test-Path $TokenPath) {
              Write-Host "Reading token from file..."
              $Token = (Get-Content -Path $TokenPath -Raw).Trim()
          } else {
              Write-Host "ERROR: Token file not found at $TokenPath"
              exit 1
          }

          if ([string]::IsNullOrEmpty($Token)) {
              Write-Host "ERROR: ADO_TOKEN is empty. Authentication might have failed."
              exit 1
          }

          $ClusterHost = "$(Cluster)" -replace "^https://",""

          Write-Host "Checking DNS Resolution..."
          $DnsCheck = nslookup $ClusterHost

          if ($DnsCheck -match "Non-existent domain") {
              Write-Host "ERROR: DNS resolution failed for $ClusterHost, retrying..."
              Start-Sleep -Seconds 5
              $DnsCheck = nslookup $ClusterHost
          }

          if ($DnsCheck -match "Non-existent domain") {
              Write-Host "ERROR: DNS resolution failed for $ClusterHost after retry."
              exit 1
          } else {
              Write-Host "DNS resolution successful: $DnsCheck"
          }

          Write-Host "Checking Network Connectivity..."
          $Retries = 3
          $Success = $false

          for ($i = 1; $i -le $Retries; $i++) {
              $NetCheck = Test-NetConnection -ComputerName $ClusterHost -Port 443

              if ($NetCheck.TcpTestSucceeded) {
                  Write-Host "Network connectivity to $ClusterHost on port 443: SUCCESS"
                  $Success = $true
                  break
              } else {
                  Write-Host "WARNING: Network connectivity test failed. Retrying ($i/$Retries)..."
                  Start-Sleep -Seconds 5
              }
          }

          if (-not $Success) {
              Write-Host "ERROR: Network connectivity to $ClusterHost on port 443 FAILED after retries."
              exit 1
          }

          Write-Host "Checking Azure Authentication..."
          $AuthCheck = az account show --query user.name -o tsv --only-show-errors
          if ([string]::IsNullOrEmpty($AuthCheck)) {
              Write-Host "ERROR: Authentication check failed."
              exit 1
          } else {
              Write-Host "Authentication successful. Logged in as: $AuthCheck"
          }

          Write-Host "Checking ADX Query Permissions..."
          $Query = ".show tables details"
          $Body = @"
          {
              "db": "$Database",
              "csl": "$Query"
          }
          "@

          $Headers = @{}
          $Headers["Authorization"] = "Bearer $Token"
          $Headers["Content-Type"]  = "application/json"

          try {
              $Response = Invoke-RestMethod -Uri "https://$(Cluster)/v1/rest/query" -Method Post -Headers $Headers -Body $Body
              Write-Host "ADX Query Permissions: SUCCESS"
          } catch {
              Write-Host "ERROR: ADX query test failed. $_"
              exit 1
          }

          Write-Host "Configuration verification completed."

    - task: PowerShell@2
      displayName: "Task: Generate KQL Files"
      inputs:
        targetType: "filePath"
        filePath: "$(Build.SourcesDirectory)/scripts/generate_kql.ps1"

    - task: PowerShell@2
      displayName: "Task: Commit & Push .kql Files"
      inputs:
        targetType: "filePath"
        filePath: "$(Build.SourcesDirectory)/scripts/commit_kql.ps1"
        arguments: "-SourceDir '$(Build.SourcesDirectory)' -BranchName '$(Build.SourceBranchName)'"
```

### Script: `commit_kql.ps1`

```powershell
param(
    [Parameter(Mandatory=$false)]
    [string]$SourceDir = $env:BUILD_SOURCESDIRECTORY,

    [Parameter(Mandatory=$false)]
    [string]$BranchName = "main"
)

# Change to the source directory.
cd $SourceDir

# Locate the fixed "tables" folder.
$devOpsFolder = Join-Path $SourceDir "tables"
if (-not (Test-Path $devOpsFolder)) {
    Write-Host "No 'tables' folder found. Skipping commit."
    exit 0
}

Write-Host "Located 'tables' folder: $devOpsFolder"

# Configure Git.
git config --global user.email "pipeline@devops.com"
git config --global user.name "Azure DevOps Pipeline"

# Ensure Git ignores line-ending changes (CRLF vs LF)
git config --global core.autocrlf false
git config --global diff.renamelimit 0

# If SYSTEM_ACCESSTOKEN is available, update the remote URL to include it.
if ($env:SYSTEM_ACCESSTOKEN) {
    Write-Host "SYSTEM_ACCESSTOKEN is available; updating remote URL..."
    $repoUrl = "https://$env:SYSTEM_ACCESSTOKEN@dev.azure.com/rchapler/DataExplorer_Delta/_git/DataExplorer_Delta"
    git remote set-url origin $repoUrl
} else {
    Write-Host "SYSTEM_ACCESSTOKEN not available. Ensure 'Allow scripts to access OAuth token' is enabled."
}

# Ensure we pull the latest changes to avoid conflicts
git pull origin $BranchName --rebase

# Stage all .kql files (overwrite existing)
git add "$devOpsFolder\*.kql"

# Check if there are actual changes before committing
git diff --cached --quiet
if ($LASTEXITCODE -eq 0) {
    Write-Host "No changes detected. Skipping commit."
    exit 0
}

# Commit the changes.
$commitMessage = "Auto-commit: Updating KQL files in 'tables' folder"
git commit -m $commitMessage

# Push the commit to the specified branch.
git push origin HEAD:$BranchName

Write-Host "Commit and push completed successfully."

```

### Script: `generate_kql.ps1`

```powershell
# generate_kql.ps1
Write-Host "Retrieving table list from ADX..."
$TokenPath = "$env:AGENT_TEMPDIRECTORY\ado_token.txt"

if (Test-Path $TokenPath) {
    Write-Host "Reading token from file..."
    $Token = (Get-Content -Path $TokenPath -Raw).Trim()
} else {
    Write-Host "ERROR: Token file not found at $TokenPath"
    exit 1
}

$Database = $env:Database
$ListQuery = ".show tables details"

Write-Host "Querying ADX for table list..."
$Body = @{
    db  = $Database
    csl = $ListQuery
} | ConvertTo-Json -Compress

$Headers = @{
    "Authorization" = "Bearer $Token"
    "Content-Type"  = "application/json"
}

try {
    $Response = Invoke-RestMethod -Uri "https://$env:Cluster/v1/rest/query" -Method Post -Headers $Headers -Body $Body
    $Tables = $Response.tables[0].rows
} catch {
    Write-Host "ERROR: Failed to retrieve table list. $_"
    exit 1
}

if (-not $Tables -or $Tables.Count -eq 0) {
    Write-Host "No tables found."
    exit 0
}

# Create a dated output folder for archival
$Timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
$OutputFolder = "$env:BUILD_SOURCESDIRECTORY/ADX_Tables_$Timestamp"
if (!(Test-Path $OutputFolder)) {
    New-Item -ItemType Directory -Path $OutputFolder | Out-Null
}

# Create a fixed folder for DevOps (always named "tables")
$DevOpsFolder = "$env:BUILD_SOURCESDIRECTORY/tables"
if (!(Test-Path $DevOpsFolder)) {
    New-Item -ItemType Directory -Path $DevOpsFolder | Out-Null
}

foreach ($Table in $Tables) {
    $TableName = $Table[0]
    Write-Host "Processing table: $TableName"

    # Query for the schema using the second query
    $SchemaQuery = ".show table $TableName cslschema"
    $SchemaBody = @{
        db  = $Database
        csl = $SchemaQuery
    } | ConvertTo-Json -Compress

    try {
        $SchemaResponse = Invoke-RestMethod -Uri "https://$env:Cluster/v1/rest/query" -Method Post -Headers $Headers -Body $SchemaBody

        # Extract only the schema definition from Rows[0][1]
        if ($SchemaResponse.tables[0].rows.Count -gt 0) {
            $SchemaString = $SchemaResponse.tables[0].rows[0][1]
        } else {
            Write-Host "ERROR: Schema not found for $TableName."
            continue
        }

        # Build final KQL command
        $FinalKql = ".create table $TableName ($SchemaString)"
    } catch {
        Write-Host "ERROR: Failed to retrieve schema for table $TableName. $_"
        continue
    }

    # File paths
    $FilePath = "$OutputFolder\$TableName.kql"
    $DevOpsFilePath = "$DevOpsFolder\$TableName.kql"

    # Overwrite existing .kql file instead of creating a new version
    Write-Host "Writing KQL file for $TableName to $DevOpsFilePath"
    Set-Content -Path $DevOpsFilePath -Value $FinalKql

    # Also write the file to the dated output folder for archival
    Write-Host "Archiving KQL file for $TableName at $FilePath"
    Set-Content -Path $FilePath -Value $FinalKql
}

```

## Appendix

### Testing API Query Logic

This section covers manual API testing using PowerShell and Azure CLI to verify that authentication and query execution are working correctly.

#### Retrieve a Fresh Access Token

Before testing, obtain a fresh access token:

```powershell
$Cluster = "https://{prefix}dec.westus.kusto.windows.net"
$Token = az account get-access-token --resource "$Cluster" --query accessToken -o tsv --only-show-errors
Write-Host "Access Token Retrieved"
```

- Ensure that the correct cluster URL is used (`westus`).
- This command should return a valid access token. If it fails, verify authentication settings.

------------------------- -------------------------

#### Manually Run the ADX Query

Using the token retrieved, execute an API request to run the `.show tables details` query:

```powershell
$Database = "rc05dataexplorerdatabase"
$Query = ".show tables details"

$Body = @{
    db  = $Database
    csl = $Query
} | ConvertTo-Json -Depth 3

$Headers = @{
    "Authorization" = "Bearer $Token"
    "Content-Type"  = "application/json"
}

Write-Host "Querying ADX..."
try {
    $Response = Invoke-RestMethod -Uri "$Cluster/v1/rest/query" -Method Post -Headers $Headers -Body $Body
    Write-Host "Query Response:"
    $Response.tables[0].rows | Format-Table
} catch {
    Write-Host "ERROR: $_"
    exit 1
}
```

- If successful, this will list all tables with details.
- If 403 Forbidden occurs, ensure that the service principal or user identity has at least `"AllDatabasesAdmin"` or `"Viewer"` role.

------------------------- -------------------------

#### Verify Role Assignments

Check the current permissions for the user or service principal running the query:

```sh
az kusto database-principal-assignment list \
    --cluster-name "{prefix}dec-dev" \
    --database-name "{prefix}ded-dev" \
    --query "[].{Principal:principalId, Role:role}" -o table
```

If the user is missing the required roles, assign the `"Viewer"` role:

```sh
az kusto database-principal-assignment create \
    --cluster-name "{{prefix}}dec-dev" \
    --database-name "rc05dataexplorerdatabase" \
    --principal-id "<YOUR_USER_OBJECT_ID>" \
    --principal-type "User" \
    --role "Viewer" \
    --tenant-id "<YOUR_TENANT_ID>" \
    --name "ADX-User-Viewer-Role"
```

------------------------- -------------------------

#### Debugging Issues

##### Token Issues

- If the token is empty, confirm that authentication is working:

  ```sh
  az login
  az account show --query user.name -o tsv
  ```

- If the token is expired, retrieve a fresh token and retry.

##### 403 Forbidden

- Ensure that the correct identity is retrieving the token (`az account show`)
- Verify the identity has necessary role permissions (`az kusto database-principal-assignment list`)
