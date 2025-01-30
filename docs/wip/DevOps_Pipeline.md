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
1. **Go to** [https://dev.azure.com/intlkusto/_settings/agentpools](https://dev.azure.com/intlkusto/_settings/agentpools).
2. Click **SelfHostedPool → New Agent**.
3. Select **Windows** as the OS.
4. **Download the agent ZIP file**.
5. **Extract it to**:
   ```plaintext
   C:\AzureDevOpsAgent
   ```
6. **Open PowerShell as Administrator**.
7. **Navigate to the extracted agent directory**:
   ```powershell
   cd C:\AzureDevOpsAgent
   ```
8. **Run the configuration script**:
   ```powershell
   .\config.cmd
   ```
9. **Enter the following details when prompted**:
   - **Server URL:** `https://dev.azure.com/intlkusto`
   - **Authentication Type:** Press **Enter** (default is PAT)
   - **Enter Personal Access Token (PAT):** Paste the **SelfHostedAgentToken**
   - **Agent Pool Name:** `SelfHostedPool`
   - **Agent Name:** *(Press Enter to use the default)*

### **2.6: Start the Agent**
```powershell
.\run.cmd
```
*(This keeps the agent running in the background.)*

To install it as a **Windows service**, run:
```powershell
.\svcInstall.cmd
.\svcStart.cmd
```

---

## **📌 Step 3: Create a Service Connection to Azure**
### **3.1: Create a Service Principal in Azure**
1. **Go to Azure Portal** → **Azure Active Directory**.
2. Navigate to **App registrations** → **New registration**.
3. **Set:**
   - **Name:** `AzureDevOpsAgentSP`
   - **Supported account types:** "Single tenant"
4. Click **Register**.
5. **Copy the Application (Client) ID and Directory (Tenant) ID**.
6. **Go to Certificates & Secrets → New client secret**.
7. **Copy the generated client secret**.

### **3.2: Assign RBAC Permissions in Azure**
1. **Go to Azure Portal → Resource Groups**.
2. Select the **resource group** where **Azure Data Explorer** exists.
3. Click **Access control (IAM) → Add role assignment**.
4. Assign these roles to the **Service Principal (`AzureDevOpsAgentSP`)**:
   - **"Contributor"**
   - **"Monitoring Reader"**
5. Click **Save**.

### **3.3: Add the Service Connection in Azure DevOps**
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
- 🔹 **Secure authentication via Azure Service Principal**
- 🔹 **RBAC assignments scoped to the Resource Group level**
- 🔹 **A manual Azure DevOps pipeline for running Azure CLI commands**

🚀 **Your Azure DevOps Manual Workflow is now fully set up!** 🚀
