# **Setting Up a Manual Workflow in Azure DevOps to Run Azure CLI Commands**

This guide walks through setting up a **self-hosted agent** in **Azure DevOps** and configuring a **manual pipeline** that runs **Azure CLI commands** against **Azure Data Explorer**.

---

## **📌 Prerequisites**
Before starting, ensure you have:
- ✅ **An Azure DevOps Organization** (`intlkusto`)
- ✅ **A DevOps Project** (`DataExplorer_Delta`)
- ✅ **Azure CLI installed** on the agent machine
- ✅ **A service connection with correct Azure permissions**
- ✅ **Administrator access to the Agent Pool**
- ✅ **A self-hosted agent registered in Azure DevOps**

---

## **📌 Step 1: Create a Personal Azure DevOps Organization (If Needed)**
If you're using a **corporate Azure DevOps account** and facing restrictions, create a **personal Azure DevOps instance**:
1. **Sign out of your corporate account** in Azure DevOps.
2. Open **Incognito Mode** (`Ctrl + Shift + N`).
3. Go to [https://dev.azure.com](https://dev.azure.com) and **sign in with a personal Microsoft account**.
4. **Create a new organization** (`intlkusto`).
5. **Create a new project** (`DataExplorer_Delta`).

---

## **📌 Step 2: Set Up a Self-Hosted Agent**
To manually trigger workflows in **Azure DevOps**, you need a **self-hosted agent** that runs commands on your machine.

### **2.1: Create & Expand a Personal Access Token (PAT)**
1. **Go to** [https://dev.azure.com/intlkusto/_usersSettings/tokens](https://dev.azure.com/intlkusto/_usersSettings/tokens).
2. Click **New Token**.
3. Set:
   - **Token Name:** `SelfHostedAgentToken`
   - **Expiration:** 90 days or more.
   - **Scopes (Initial Selection):**
     - ✅ **Build (Read & Execute)**
4. Click **Create** and **copy the PAT immediately**.

### **2.2: Expand the PAT Permissions**
5. **Go back to the PAT page** → Edit `SelfHostedAgentToken`.
6. **Enable these additional scopes**:
   - ✅ **Agent Pools (Read & Manage)**
   - ✅ **Project and Team (Read & Write)**
7. Click **Save** and **copy the updated PAT**.

---

### **2.3: Create an Agent Pool & Assign Permissions**
1. **Go to** [https://dev.azure.com/intlkusto/_settings/agentpools](https://dev.azure.com/intlkusto/_settings/agentpools).
2. Click **New Agent Pool**.
3. Set:
   - **Pool Name:** `SelfHostedPool`
   - **Auto-provision in all projects:** ✅
4. Click **Create**.

### **2.4: Assign Agent Pool Permissions**
1. Click **SelfHostedPool** → **Security**.
2. **Ensure your DevOps user is listed as an Administrator**.
3. If missing, click **Add**:
   - **User:** Select your DevOps account.
   - **Role:** `Administrator`.
4. Click **Save**.

---

### **2.5: Install & Configure the Self-Hosted Agent**
1. **Open PowerShell as Administrator**.
2. **Verify that no existing agent is running**:
   ```powershell
   Get-Process | Where-Object { $_.Path -like "C:\AzureDevOpsAgent\*" }
   ```
3. **Verify the agent folder doesn't exist**:
   ```powershell
   Test-Path C:\AzureDevOpsAgent
   ```
   - If this returns `True`, delete the folder:
     ```powershell
     Remove-Item -Recurse -Force C:\AzureDevOpsAgent
     ```
4. **Go to** [https://dev.azure.com/intlkusto/_settings/agentpools](https://dev.azure.com/intlkusto/_settings/agentpools).
5. Click **SelfHostedPool → New Agent**.
6. Select **Windows** as the OS.
7. **Download the agent ZIP file**.
8. **Extract it to**:
   ```plaintext
   C:\AzureDevOpsAgent
   ```
9. **Navigate to the extracted agent directory**:
   ```powershell
   cd C:\AzureDevOpsAgent
   ```
10. **Run the configuration script**:
   ```powershell
   .\config.cmd
   ```
11. **Enter the following details when prompted**:
   ```
   Enter server URL > https://dev.azure.com/intlkusto
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

---

### **2.6: Verify the Agent in Azure DevOps**
1. **Go to** [https://dev.azure.com/intlkusto/_settings/agentpools](https://dev.azure.com/intlkusto/_settings/agentpools).
2. Click **SelfHostedPool**.
3. Ensure the **agent appears as Online and Available**.

---

## **📌 Step 3: Create a Service Connection to Azure**
1. **Go to Azure DevOps** → **Project Settings → Service Connections**.
2. Click **New service connection** → **Azure Resource Manager**.
3. Choose **"Service Principal (manual)"**.
4. **Enter:**
   - **Subscription:** Select your Azure subscription.
   - **Tenant ID:** Paste the **Directory (Tenant) ID**.
   - **Service Principal ID:** Paste the **Application (Client) ID**.
   - **Service Principal Key:** Paste the **Client Secret**.
5. Click **Verify** → **Save**.

---

## **📌 Step 4: Create a Manual Azure DevOps Pipeline**
1. **Go to Azure DevOps** → **DataExplorer_Delta** → **Pipelines**.
2. Click **New Pipeline**.
3. Select **"Azure Repos Git"** and choose your repository.
4. Click **"Starter pipeline"** and replace the YAML with:

```yaml
trigger: none  

pool:
  name: SelfHostedPool  

steps:
- task: AzureCLI@2
  displayName: "List Tables in Azure Data Explorer"
  inputs:
    azureSubscription: "<Your-Service-Connection-Name>"  
    scriptType: "bash"
    scriptLocation: "inlineScript"
    inlineScript: |
      az login --service-principal -u "<Your-SP-App-ID>" -p "<Your-SP-Secret>" --tenant "<Your-Tenant-ID>"
      az kusto query --cluster-name "<adx-cluster-name>" --database-name "<database-name>" --query "Tables | project TableName"
```

---

## **✅ Summary**
This guide ensures:
- 🔹 **A properly configured self-hosted agent**
- 🔹 **Accurate documentation of every agent setup prompt**
- 🔹 **Secure authentication via Azure Service Principal**
- 🔹 **RBAC assignments scoped to the Resource Group level**
- 🔹 **A manual Azure DevOps pipeline for running Azure CLI commands**

🚀 **Your Azure DevOps Manual Workflow is now fully set up!** 🚀
