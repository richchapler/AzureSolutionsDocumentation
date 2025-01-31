Here is the full updated documentation with the **Kusto NuGet installation and configuration** section included.

---

# Data Explorer Delta  
...using DevOps Pipeline  

## Prerequisites  

### Basics:  
- An Azure DevOps organization ("rchapler")  
- A DevOps project ("DataExplorer_Delta")  
- Azure CLI installed on the agent machine  
- Administrator access to the agent pool  

### Special:
- PowerShell Core installed and available in `PATH`
- Azure CLI and PowerShell updates applied
- Azure CLI Kusto extension installed
- Microsoft.Azure.Kusto.Data installed via NuGet
- A self-hosted agent registered in Azure DevOps
- A service connection with correct Azure permissions

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

## Special Pre-Requisite: Azure CLI and PowerShell Updates  

### Check for Available Updates  
```
az version
az upgrade
$PSVersionTable.PSVersion
winget upgrade --id Microsoft.PowerShell
```
Restart after updating.  

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

## Special Pre-Requisite: Service Connection  

### Create a New Azure Service Connection  
1. Go to "Azure DevOps" → "Project Settings" → "Service Connections".  
2. Click "New service connection" → "Azure Resource Manager".  
3. Select:  
   - "Identity Type" → `"App Registration (Automatic)"`  
   - "Scope Level" → `"Subscription"`  
4. Set "Service Connection Name" (e.g., `AzureDataExplorerService`).  

## Special Pre-Requisite: Microsoft.Azure.Kusto.Data via NuGet  

### 1. Disable VPN (If Applicable)  
If using a VPN, **turn it off** before installation.  

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

## Pipeline Definition  
```
trigger: none
pool:
  name: SelfHostedPool

steps:
- task: AzureCLI@2
  displayName: "Authenticate & List Tables in Azure Data Explorer"
  inputs:
    azureSubscription: "AzureServiceConnection"
    scriptType: "powershell"
    scriptLocation: "inlineScript"
    inlineScript: |
      az account show
      az kusto query --cluster-name "rc05dataexplorercluste" --database-name "rc05dataexplorerdatabase" --query "Tables | project TableName"
```

## Review and Run  
1. Proceed to "Review".  
2. Click "Save and Run" to test the pipeline. 🚀
