# Data Explorer "Sniff-and-Diff" (using DevOps Pipeline)

## Prerequisites

Ensure you have:
- An Azure DevOps organization ("rchapler")
- A DevOps project ("DataExplorer_Delta")
- Azure CLI installed on the agent machine
- PowerShell Core installed and available in `PATH`
- A service connection with correct Azure permissions
- Administrator access to the agent pool
- A self-hosted agent registered in Azure DevOps

## PowerShell Core

If PowerShell Core (`pwsh`) is not installed or not available in `PATH`, install it using the following steps.

### Verify if PowerShell Core is Installed

Run:
```powershell
pwsh -v
```
If the command is not recognized, proceed with installation.

### Install PowerShell Core

1. Open PowerShell as Administrator.
2. Run:
   ```powershell
   winget install --id Microsoft.PowerShell --source winget --accept-package-agreements --accept-source-agreements
   ```
3. Restart the machine to ensure `pwsh` is available in `PATH`.

### Verify Installation Path

Run:
```powershell
Get-ChildItem "C:\Program Files\PowerShell"
```
If a `7` folder exists, check if `pwsh.exe` is inside:
```powershell
Test-Path "C:\Program Files\PowerShell\7\pwsh.exe"
```

### Add PowerShell Core to `PATH` (If Needed)

If `pwsh.exe` exists but is not found, run:
```powershell
$pwshPath = "C:\Program Files\PowerShell\7\"
$envPath = [System.Environment]::GetEnvironmentVariable("Path", "Machine")
if ($envPath -notlike "*$pwshPath*") {
    [System.Environment]::SetEnvironmentVariable("Path", "$envPath;$pwshPath", "Machine")
}
```
Restart the machine and verify by running:
```powershell
where.exe pwsh
```

## Self-Hosted Agent

To manually trigger workflows in Azure DevOps, you need a self-hosted agent that runs commands on your machine.

### Create and Expand a Personal Access Token (PAT)

1. Go to [https://dev.azure.com/rchapler/_usersSettings/tokens](https://dev.azure.com/rchapler/_usersSettings/tokens).
2. Click "New Token".
3. Set:
   - "Token Name": `SelfHostedAgentToken`
   - "Expiration": 90 days or more.
   - "Scopes":
     - "Build (Read & Execute)"
4. Click "Create" and copy the PAT immediately.

#### Expand the PAT Permissions

5. Go back to the PAT page → Edit `SelfHostedAgentToken`.
6. Enable these additional scopes:
   - "Agent Pools (Read & Manage)"
   - "Project and Team (Read & Write)"
7. Click "Save" and copy the updated PAT.

### Create an Agent Pool and Assign Permissions

1. Go to [https://dev.azure.com/rchapler/_settings/agentpools](https://dev.azure.com/rchapler/_settings/agentpools).
2. Click "New Agent Pool".
3. Set:
   - "Pool Name": `SelfHostedPool`
   - "Auto-provision in all projects": ✅
4. Click "Create".

#### Assign Agent Pool Permissions

1. Click "SelfHostedPool" → "Security".
2. Ensure your DevOps user is listed as an "Administrator".
3. If missing, click "Add":
   - "User": Select your DevOps account.
   - "Role": `Administrator`.
4. Click "Save".

### Install and Configure the Self-Hosted Agent

1. Open PowerShell as Administrator.
2. Verify that no existing agent is running:
   ```powershell
   Get-Process | Where-Object { $_.Path -like "C:\AzureDevOpsAgent\*" }
   ```
3. Verify the agent folder doesn't exist:
   ```powershell
   Test-Path C:\AzureDevOpsAgent
   ```
   - If this returns `True`, delete the folder:
     ```powershell
     Remove-Item -Recurse -Force C:\AzureDevOpsAgent
     ```
4. Go to [https://dev.azure.com/rchapler/_settings/agentpools](https://dev.azure.com/rchapler/_settings/agentpools).
5. Click "SelfHostedPool" → "New Agent".
6. Select "Windows" as the OS.
7. Download the agent ZIP file.
8. Extract it to:
   ```powershell
   Expand-Archive -Path "$HOME\Downloads\vsts-agent-win-x64-4.248.0.zip" -DestinationPath "C:\AzureDevOpsAgent"
   ```
9. Navigate to the extracted agent directory:
   ```powershell
   cd C:\AzureDevOpsAgent
   ```
10. Run the configuration script:
    ```powershell
    .\config.cmd
    ```
11. Enter the following details when prompted:
    ```
    Enter server URL > https://dev.azure.com/rchapler
    Enter authentication type (press enter for PAT) > (Press Enter)
    Enter personal access token > (Paste SelfHostedAgentToken)
    Enter agent pool (press enter for default) > SelfHostedPool
    Enter agent name (press enter for LAPTOP-XXXXX) > (Press Enter)
    Enter work folder (press enter for _work) > (Press Enter)
    Enter run agent as service? (Y/N) (press enter for N) > Y
    Enter enable SERVICE_SID_TYPE_UNRESTRICTED for agent service (Y/N) (press enter for N) > (Press Enter)
    Enter User account to use for the service (press enter for NT AUTHORITY\NETWORK SERVICE) > (Press Enter)
    Enter whether to prevent service starting immediately after configuration is finished? (Y/N) (press enter for N) > (Press Enter)
    ```

### Verify the Agent in Azure DevOps

1. Go to [https://dev.azure.com/rchapler/_settings/agentpools](https://dev.azure.com/rchapler/_settings/agentpools).
2. Click "SelfHostedPool".
3. Ensure the agent appears as "Online" and "Available".

## Service Connection to Azure

Azure DevOps no longer provides a direct option to manually enter a Service Principal ID and Key. Instead, you must use "App Registration (Automatic)", which automatically creates a Service Principal in Azure Active Directory.

### Create a New Azure Service Connection

1. Go to Azure DevOps → "Project Settings" → "Service Connections".
2. Click "New service connection" → "Azure Resource Manager".
3. Select:
   - "Identity Type" → `"App Registration (Automatic)"`
   - "Credential" → `"Workload Identity Federation"`
   - "Scope Level" → `"Subscription"`
   - "Subscription" → Select your Azure subscription.
4. Enter a meaningful name for "Service Connection Name" (e.g., `AzureDataExplorerService`).
5. Check "Grant Access to Pipelines" (Optional).

### Configure the Pipeline

```yaml
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
2. Click "Save and Run" to test the pipeline.
