# Data Explorer Delta  
...using DevOps Pipeline  

## Prerequisites  

### Basics:  
- An Azure DevOps organization ("rchapler")  
- A DevOps project ("DataExplorer_Delta")  
- Azure CLI installed on the agent machine  
- Administrator access to the agent pool
- Azure Key Vault
- Application Registration

### Special:  
- PowerShell Core installed and available in `PATH`  
- Azure CLI and PowerShell updates applied  
- Azure CLI Kusto extension installed  
- Microsoft.Azure.Kusto.Data installed via NuGet  
- A self-hosted agent registered in Azure DevOps  
- A service connection with correct Azure permissions  

---

## Special Pre-Requisite: PowerShell Core  

If PowerShell Core (`pwsh`) is not installed or not available in `PATH`, install it.  

### Verify if PowerShell Core is Installed  
```
pwsh -v
```
If the command is not recognized, proceed with installation.  

### Install PowerShell Core  
```
winget install --id Microsoft.PowerShell --source winget --accept-package-agreements --accept-source-agreements
```
Restart the machine to ensure `pwsh` is available in `PATH`.  

### Verify Installation Path  
```
Get-ChildItem "C:\Program Files\PowerShell"
Test-Path "C:\Program Files\PowerShell\7\pwsh.exe"
```

### Add PowerShell Core to `PATH` (If Needed)  
```
$pwshPath = "C:\Program Files\PowerShell\7\"
$envPath = [System.Environment]::GetEnvironmentVariable("Path", "Machine")
if ($envPath -notlike "*$pwshPath*") {
    [System.Environment]::SetEnvironmentVariable("Path", "$envPath;$pwshPath", "Machine")
}
```
Restart and verify:  
```
where.exe pwsh
```

---

## Special Pre-Requisite: Azure CLI and PowerShell Updates  

### Check for Available Updates  
```
az version
az upgrade
$PSVersionTable.PSVersion
winget upgrade --id Microsoft.PowerShell
```
Restart after updating.  

---

## Special Pre-Requisite: Azure CLI Kusto Extension  

### Verify if the Kusto Extension is Installed  
```
az extension list --output table
```
If `kusto` is not listed, install:  
```
az extension add --name kusto
```
Verify installation:  
```
az extension list --output table
az kusto -h
```

---

## Special Pre-Requisite: Microsoft.Azure.Kusto.Data via NuGet  

### 1. Disable VPN (If Applicable)  
If using a VPN, turn it off before installation.  

### 2. Verify Network Connectivity  
```
Test-NetConnection -ComputerName api.nuget.org -Port 443
Test-NetConnection -ComputerName www.nuget.org -Port 443
```
If `www.nuget.org` fails, update DNS:  
```
netsh interface ip set dns "Wi-Fi" static 8.8.8.8
ipconfig /flushdns
```
Restart the adapter and test again.  

### 3. Install NuGet CLI  
```
Invoke-WebRequest -Uri https://dist.nuget.org/win-x86-commandline/latest/nuget.exe -OutFile C:\KustoSDK\nuget.exe
C:\KustoSDK\nuget.exe help
```

### 4. Install Required Packages  
```
C:\KustoSDK\nuget.exe install Microsoft.Azure.Kusto.Data -OutputDirectory C:\KustoSDK
C:\KustoSDK\nuget.exe install Microsoft.Azure.Kusto.Ingest -OutputDirectory C:\KustoSDK
C:\KustoSDK\nuget.exe install Newtonsoft.Json -OutputDirectory C:\KustoSDK
C:\KustoSDK\nuget.exe install Microsoft.IdentityModel.Tokens -OutputDirectory C:\KustoSDK
```

### 5. Locate and Load the DLLs  
```
Get-ChildItem -Path "C:\KustoSDK" -Recurse -Filter "Kusto.Data.dll"
Add-Type -Path "C:\KustoSDK\Microsoft.Azure.Kusto.Data.13.0.0\lib\netstandard2.0\Kusto.Data.dll"
```

### 6. Authenticate with Azure  
```
az login
az account show
$Token = az account get-access-token --resource "https://kusto.windows.net" --query accessToken -o tsv
```

### 7. Run a Test Query  
```
$Cluster = "https://<your-cluster-name>.eastus.kusto.windows.net"
$Database = "<your-database-name>"
$Query = "Tables | project TableName"

$ConnectionString = "Data Source=$Cluster;Initial Catalog=$Database;Fed=True;Authorization=Bearer $Token"
$KustoClient = New-Object Microsoft.Azure.Kusto.Data.KustoClient -ArgumentList $ConnectionString
$QueryProvider = $KustoClient.GetQueryProvider()
$QueryResults = $QueryProvider.ExecuteQuery($Query, $null, $null)

$QueryResults.Tables[0].Rows | Format-Table
```

### Alternative: Use `az rest` Instead  
```
az rest --method post `
  --url "https://<your-cluster-name>.eastus.kusto.windows.net/v1/rest/query" `
  --headers "Content-Type=application/json" `
  --body "{ \"db\": \"<your-database-name>\", \"csl\": \"Tables | project TableName\" }"
```

---

### **Special Pre-Requisite: Git Installation and Configuration**  

For the pipeline to commit and push generated `.kql` files to the repository, **Git must be installed and properly configured** on the agent machine.

---

#### **1️⃣ Verify if Git is Installed**  
Run the following command:  
```powershell
where.exe git
```
If no output is returned, Git is **not installed**.

---

#### **2️⃣ Install Git**  
Install Git using **winget**:  
```powershell
winget install --id Git.Git --source winget --accept-package-agreements --accept-source-agreements
```
Alternatively, download and install Git manually from [https://git-scm.com/downloads](https://git-scm.com/downloads), ensuring you enable:
- ✅ "Add Git to PATH"
- ✅ "Enable credential manager"  

After installation, **restart the terminal** and verify:  
```powershell
git --version
```
If Git is installed correctly, it will return the installed version.

---

#### **3️⃣ Ensure Git is in PATH**  
If `git` is installed but not recognized, manually add it to the system **PATH**:
```powershell
$GitPath = "C:\Program Files\Git\bin"
$envPath = [System.Environment]::GetEnvironmentVariable("Path", "Machine")
if ($envPath -notlike "*$GitPath*") {
    [System.Environment]::SetEnvironmentVariable("Path", "$envPath;$GitPath", "Machine")
}
```
**Restart the terminal** and verify:  
```powershell
where.exe git
```

---

#### **4️⃣ Ensure the DevOps Agent Can Access Git**  
If Git is installed under a different user, but the **DevOps Agent** runs as `NT AUTHORITY/NETWORK SERVICE`, Git might not be accessible.  

**Verify that the agent process can find `git.exe`**:  
```powershell
Get-Command git | Select-Object -ExpandProperty Source
```
If this fails, restart the **Azure DevOps Agent** service and try again.

---

#### **5️⃣ Configure Git as a Safe Directory**  
If you see an error like:
```
fatal: detected dubious ownership in repository at 'C:/agent/_work/1/s'
```
Run the following command to mark the DevOps workspace as safe:  
```powershell
git config --global --add safe.directory C:/agent/_work/1/s
```

---

#### **6️⃣ Verify Git is Tracking the Repository**  
Run:
```powershell
cd C:\agent\_work\1\s
git status
```
- If it says **"Not a git repository"**, initialize it:  
  ```powershell
  git init
  git remote add origin https://dev.azure.com/rchapler/DataExplorer_Delta/_git/DataExplorer_Delta
  git fetch
  git checkout main
  ```

---

#### **7️⃣ Configure Git for DevOps Authentication**  
Ensure Git uses the **System.AccessToken** for authentication:
```powershell
git config --global user.email "pipeline@devops.com"
git config --global user.name "Azure DevOps Pipeline"
git config --global credential.helper store
echo "https://user:$(System.AccessToken)@dev.azure.com" | git credential approve
```

---

#### **8️⃣ Manually Push Missing Files**  
If Git is now installed but the pipeline failed to commit `.kql` files, manually push them:  
```powershell
cd C:\agent\_work\1\s
git add -A
git commit -m "Manually adding missing .kql files"
git push origin main
```

---

### **When to Use This Section?**
- If **Git is missing or not recognized** (`git --version` fails)
- If **Git is installed but not in PATH**
- If **DevOps Agent cannot access Git**
- If **Pipelines fail to commit and push files**
- If **"detected dubious ownership"** error appears
- If **Pipeline changes do not appear in the repository**

---

### Special Pre-Requisite: Azure DevOps Repository Permissions  

For the pipeline to commit and push generated `.kql` files back to the repository, ensure that the DevOps Build Service has the correct permissions.  

#### 1️⃣ Verify Build Service Permissions  
1. Navigate to **Azure DevOps** → **Project Settings** → **Repositories**  
2. Select the repository **"DataExplorer_Delta"**  
3. Go to the **Security** tab  
4. Locate **"DataExplorer_Delta Build Service (rchapler)"**  
5. Ensure the following permissions are **set to "Allow"**:  
   - `Contribute`  
   - `Create branch`  
   - `Contribute to pull requests`  
   - `Bypass policies when pushing` (if necessary)  

#### 2️⃣ Confirm and Apply Changes  
- **If a "Save" button appears**, click it  
- **If no "Save" button appears**, refresh the page to confirm that permissions were applied  

#### 3️⃣ Validate with a Manual Push  
After setting permissions, validate push access by running the following on the agent machine:  
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

---

## Special Pre-Requisite: Self-Hosted Agent  

### Create and Expand a Personal Access Token (PAT)  
1. Go to [DevOps Tokens](https://dev.azure.com/rchapler/_usersSettings/tokens).  
2. Click "New Token".  
3. Set:  
   - "Token Name": `SelfHostedAgentToken`  
   - "Expiration": 90 days or more  
   - "Scopes": `"Build (Read & Execute)"`  
4. Click "Create" and copy the PAT.  

### Expand the PAT Permissions  
- Enable `"Agent Pools (Read & Manage)"` and `"Project and Team (Read & Write)"`.  

### Create an Agent Pool and Assign Permissions  
1. Go to [Agent Pools](https://dev.azure.com/rchapler/_settings/agentpools).  
2. Click "New Agent Pool".  
3. Set:  
   - "Pool Name": `SelfHostedPool`  
   - Enable "Auto-provision in all projects".  
4. Assign "Administrator" role to your DevOps account.  

### Install and Configure the Self-Hosted Agent  
```
Test-Path C:\AzureDevOpsAgent
Remove-Item -Recurse -Force C:\AzureDevOpsAgent
```
1. Download and extract:  
```
Expand-Archive -Path "$HOME\Downloads\vsts-agent-win-x64-4.248.0.zip" -DestinationPath "C:\AzureDevOpsAgent"
cd C:\AzureDevOpsAgent
```
2. Run the configuration:  
```
.\config.cmd
```
3. Enter:  
```
Server URL: https://dev.azure.com/rchapler
Auth Type: (Press Enter)
PAT: (Paste SelfHostedAgentToken)
Pool: SelfHostedPool
```

---

## Special Pre-Requisite: Service Connection  

### Create a New Azure Service Connection  
1. Go to "Azure DevOps" → "Project Settings" → "Service Connections".  
2. Click "New service connection" → "Azure Resource Manager".  
3. Select:  
   - "Identity Type" → `"App Registration (Automatic)"`  
   - "Scope Level" → `"Subscription"`  
4. Set "Service Connection Name" (e.g., `AzureDataExplorerService`).  

---

## Special Pre-Requisite: Key Vault, Secrets  

- Navigate to "Secrets" → "Generate/Import"  
- Add the following secrets using these exact names:  
  - `AZURE-TENANT-ID` → Your Azure Tenant ID  
  - `AZURE-SUBSCRIPTION-ID` → Your Azure Subscription ID  
  - `AZURE-CLIENT-ID` → Your App Registration Client ID  
  - `AZURE-CLIENT-SECRET` → Your App Registration Client Secret  
- Click "Save"  

### Grant DevOps Access to Key Vault  

- In Azure Portal, go to Key Vault → "Access control (IAM)"  
- Assign the following roles to the Application Registration created for the Service Connection {e.g., rchapler-DataExplorer_Delta-GUID}:
  - Key Vault Reader
  - Key Vault Secrets User  
- Click Save  

### Create a Variable Group in Azure DevOps  
- Go to Azure DevOps → Pipelines → Library  
- Click "New Variable Group" and name it `Secrets`  
- Enable "Link secrets from an Azure key vault as variables"  
- Select the Azure subscription and the Key Vault (`myKeyVault`)  
- Click "Authorize" to allow Azure Pipelines to set the necessary permissions or manually apply them in the Azure portal  
- Click "Add" and select the secrets from Key Vault  
- Click "Save"  

![image](https://github.com/user-attachments/assets/ac3d697c-c4c3-4327-8f98-32855e19be25)

## Pipeline Definition

```yaml
trigger: none
pool:
  name: SelfHostedPool

variables:
- group: Secrets
- name: Cluster
  value: "rc05dataexplorercluste.westus.kusto.windows.net"
- name: Database
  value: "rc05dataexplorerdatabase"

jobs:
- job: RunPipeline
  displayName: "Data Explorer Delta"
  steps:
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
        $Cluster = "$(Cluster)"

        Write-Host "Starting Azure login..."
        az login --service-principal --username "$ClientId" --password "$ClientSecret" --tenant "$TenantId" --only-show-errors
        az account set --subscription "$SubscriptionId" --only-show-errors

        Write-Host "Retrieving Azure Data Explorer access token..."
        $Token = az account get-access-token --resource "https://$Cluster" --query accessToken -o tsv --only-show-errors

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
    displayName: "Task: Set Timestamp"
    name: SetTimestamp
    inputs:
      azureSubscription: "AzureServiceConnection"
      scriptType: "pscore"
      scriptLocation: "inlineScript"
      inlineScript: |
        echo "##vso[task.setvariable variable=DateString]$(Get-Date -Format 'yyyyMMdd-HHmmss')"

  - task: AzureCLI@2
    displayName: "Task: List Tables"
    inputs:
      azureSubscription: "AzureServiceConnection"
      scriptType: "pscore"
      scriptLocation: "inlineScript"
      inlineScript: |
        Write-Host "Starting Data Explorer query..."
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

        $Cluster = "$(Cluster)"
        $Database = "$(Database)"
        $Query = ".show tables details"

        Write-Host "Querying ADX: $Database with query: $Query"

        $Body = @{
          db  = $Database
          csl = $Query
        } | ConvertTo-Json -Compress

        $EncodedBody = $Body -replace '"', '\"'

        $Headers = @(
          "Authorization=Bearer $Token"
          "Content-Type=application/json"
        )

        az rest --method post --uri "https://$Cluster/v1/rest/query" `
          --headers $Headers `
          --body "$EncodedBody" `
          --only-show-errors

        if ($LASTEXITCODE -ne 0) {
          Write-Host "ERROR: Query failed."
          exit 1
        }

  - task: AzureCLI@2
    displayName: "Task: Iterate thru Tables"
    inputs:
      azureSubscription: "AzureServiceConnection"
      scriptType: "pscore"
      scriptLocation: "inlineScript"
      inlineScript: |
        Write-Host "Retrieving table list from ADX..."
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

        $Cluster = "$(Cluster)"
        $Database = "$(Database)"
        $Query = ".show tables details"

        Write-Host "Executing query to get table list..."
        
        $Body = @{
          db  = $Database
          csl = $Query
        } | ConvertTo-Json -Compress

        $Headers = @{
          "Authorization" = "Bearer $Token"
          "Content-Type"  = "application/json"
        }

        try {
            $Response = Invoke-RestMethod -Uri "https://$Cluster/v1/rest/query" -Method Post -Headers $Headers -Body $Body
            $Tables = $Response.tables[0].rows
        } catch {
            Write-Host "ERROR: Failed to retrieve table list. $_"
            exit 1
        }

        if (-not $Tables -or $Tables.Count -eq 0) {
            Write-Host "No tables found."
            exit 0
        }

        $DateString = "$(DateString)"
        $OutputFolder = "$(Build.SourcesDirectory)/ADX_Tables_$DateString"

        if (!(Test-Path $OutputFolder)) {
            New-Item -ItemType Directory -Path $OutputFolder | Out-Null
        }

        Write-Host "Creating .kql files for tables in $OutputFolder..."
        foreach ($Table in $Tables) {
            $TableName = $Table[0]
            $FilePath = "$OutputFolder\$TableName.kql"
            Write-Host "Creating file: $FilePath"
            New-Item -Path $FilePath -ItemType File -Force | Out-Null
        }

  - task: PowerShell@2
    displayName: "Task: Commit & Push .kql Files"
    inputs:
      targetType: "inline"
      script: |
        cd "$(Build.SourcesDirectory)"

        $DateString = Get-Date -Format "yyyyMMdd-HHmmss"
        $folderPath = "ADX_Tables_$DateString"

        Write-Host "Checking for new .kql files in $folderPath..."

        if (Test-Path $folderPath) {
            git config --global user.email "pipeline@devops.com"
            git config --global user.name "Azure DevOps Pipeline"

            # Configure Git to use a PAT for authentication
            $PatToken = "$(System.AccessToken)"
            git config --global credential.helper store
            echo "https://user:$PatToken@dev.azure.com" | git credential approve

            # Ensure we are on the correct branch and update local branch
            git checkout main
            git fetch origin main
            git reset --hard origin/main

            git add $folderPath/*.kql
            git commit -m "Auto-commit: Adding .kql files for $DateString"
            git push https://user:$PatToken@dev.azure.com/rchapler/DataExplorer_Delta/_git/DataExplorer_Delta main --force

            Write-Host "Successfully committed and pushed .kql files."
        } else {
            Write-Host "No .kql files found to commit."
        }
```

---

## Testing API Query Logic
This section covers manual API testing using PowerShell and Azure CLI to verify that authentication and query execution are working correctly.

### 1️⃣ Retrieve a Fresh Access Token
Before testing, obtain a fresh access token:
```powershell
$Cluster = "https://rc05dataexplorercluster.westus.kusto.windows.net"
$Token = az account get-access-token --resource "$Cluster" --query accessToken -o tsv --only-show-errors
Write-Host "Access Token Retrieved"
```
- Ensure that the correct cluster URL is used (`westus`).
- This command should return a valid access token. If it fails, verify authentication settings.

---

### 2️⃣ Manually Run the ADX Query
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

---

### 3️⃣ Verify Role Assignments
Check the current permissions for the user or service principal running the query:
```sh
az kusto database-principal-assignment list \
    --cluster-name "rc05dataexplorercluster" \
    --database-name "rc05dataexplorerdatabase" \
    --query "[].{Principal:principalId, Role:role}" -o table
```
If the user is missing the required roles, assign the `"Viewer"` role:
```sh
az kusto database-principal-assignment create \
    --cluster-name "rc05dataexplorercluster" \
    --database-name "rc05dataexplorerdatabase" \
    --principal-id "<YOUR_USER_OBJECT_ID>" \
    --principal-type "User" \
    --role "Viewer" \
    --tenant-id "<YOUR_TENANT_ID>" \
    --name "ADX-User-Viewer-Role"
```

---

### 4️⃣ Debugging Issues
#### Token Issues
- If the token is empty, confirm that authentication is working:
  ```sh
  az login
  az account show --query user.name -o tsv
  ```
- If the token is expired, retrieve a fresh token and retry.

#### 403 Forbidden
- Ensure that the correct identity is retrieving the token (`az account show`).
- Verify the identity has necessary role permissions (`az kusto database-principal-assignment list`).

---

### Conclusion
This process ensures that:
- The Azure authentication process is working correctly.
- The token has the required permissions to execute ADX queries.
- The correct API endpoint and payload format are being used.

🚀 Run these steps before deploying pipeline changes to validate API query logic manually.

---

## Review and Run  
1. Proceed to "Review".  
2. Click "Save and Run" to test the pipeline. 🚀  
