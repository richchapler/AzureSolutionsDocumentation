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

![Uploading image.png…]()


### Link the Variable Group in Your Pipeline  
Add this to your pipeline YAML:  
```yaml
variables:
- group: mySecrets
```  
This automatically injects Key Vault secrets into the pipeline for authentication.
---

## Pipeline Definition  
```
- task: AzureCLI@2
  displayName: "Authenticate & Query Azure Data Explorer via API"
  inputs:
    azureSubscription: "AzureServiceConnection"
    scriptType: "pscore"
    scriptLocation: "inlineScript"
    inlineScript: |
      # Ensure authentication to the correct tenant
      $TenantId = "<YOUR_TENANT_ID>"  # Replace with actual Tenant ID
      $SubscriptionId = "<YOUR_SUBSCRIPTION_ID>"  # Replace if necessary

      az login --service-principal --username "<YOUR_APP_ID>" --password "<YOUR_APP_SECRET>" --tenant $TenantId
      az account set --subscription $SubscriptionId

      # Get Azure Data Explorer access token
      $Token = az account get-access-token --resource "https://help.kusto.windows.net" --query accessToken -o tsv

      # Define Kusto API endpoint
      $Cluster = "https://rc05dataexplorercluste.eastus.kusto.windows.net"
      $Database = "rc05dataexplorerdatabase"
      $Query = "Tables | project TableName"

      # Prepare API request body
      $Body = @{
        db = $Database
        csl = $Query
      } | ConvertTo-Json -Depth 3

      # Set API headers
      $Headers = @{
        "Authorization" = "Bearer $Token"
        "Content-Type"  = "application/json"
      }

      # Send request to Kusto Data Explorer API
      try {
        $Response = Invoke-RestMethod -Uri "$Cluster/v1/rest/query" -Method Post -Headers $Headers -Body $Body
        $Response.tables[0].rows | Format-Table
      } catch {
        Write-Host "ERROR: $_"
        exit 1
      }
```

---

## Review and Run  
1. Proceed to "Review".  
2. Click "Save and Run" to test the pipeline. 🚀  
