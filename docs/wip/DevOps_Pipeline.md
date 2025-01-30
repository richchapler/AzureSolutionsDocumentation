# **Setting Up a Self-Hosted Azure DevOps Agent for Azure Data Explorer**

This guide walks through setting up a **self-hosted agent** in **Azure DevOps** to execute **Azure CLI commands** against **Azure Data Explorer**.

---

## **🚀 Prerequisites**
Before starting, ensure you have:
- ✅ **An Azure DevOps Organization** (`intlkusto`)
- ✅ **A DevOps Project** (`DataExplorer_Delta`)
- ✅ **Azure CLI installed** on the agent machine
- ✅ **A service connection with correct Azure permissions**
- ✅ **Administrator access to the Agent Pool**

---

## **📌 Step 1: Create a Personal Azure DevOps Organization (If Needed)**
If you're using a **corporate Azure DevOps account** and facing restrictions, create a **personal Azure DevOps instance**:
1. **Sign out of your corporate account** in Azure DevOps.
2. Open **Incognito Mode** (`Ctrl + Shift + N`).
3. Go to [https://dev.azure.com](https://dev.azure.com) and **sign in with a personal Microsoft account**.
4. **Create a new organization** (`intlkusto`).
5. **Create a new project** (`DataExplorer_Delta`).

---

## **📌 Step 2: Create & Expand a Personal Access Token (PAT)**
A **Personal Access Token (PAT)** is required for agent authentication.

### **2.1: Create the PAT**
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

## **📌 Step 3: Create an Agent Pool & Assign Permissions**
1. **Go to** [https://dev.azure.com/intlkusto/_settings/agentpools](https://dev.azure.com/intlkusto/_settings/agentpools).
2. Click **New Agent Pool**.
3. Set:
   - **Pool Name:** `SelfHostedPool`
   - **Auto-provision in all projects:** ✅
4. Click **Create**.

### **3.1: Assign Agent Pool Permissions**
1. Click **SelfHostedPool** → **Security**.
2. **Ensure your DevOps user is listed as an Administrator**.
3. If missing, click **Add**:
   - **User:** Select your DevOps account.
   - **Role:** `Administrator`.
4. Click **Save**.

---

## **📌 Step 4: Install & Configure the Self-Hosted Agent**
### **4.1: Remove Any Existing Agent Installations**
1. **Open PowerShell as Administrator**.
2. **Check for any existing agent processes**:
   ```powershell
   Get-Process | Where-Object { $_.Path -like "C:\AzureDevOpsAgent\*" }
   ```
3. **If processes exist, stop them**:
   ```powershell
   Stop-Process -Name "Agent.Listener" -Force -ErrorAction SilentlyContinue
   Stop-Process -Name "Agent.Worker" -Force -ErrorAction SilentlyContinue
   ```
4. **If the agent is installed as a service, stop it**:
   ```powershell
   Stop-Service -Name "vstsagent.intlkusto.SelfHostedPool" -Force -ErrorAction SilentlyContinue
   ```
5. **Delete any old agent files**:
   ```powershell
   Remove-Item -Recurse -Force C:\AzureDevOpsAgent
   ```
6. **Restart your machine**.

---

### **4.2: Download & Configure the Agent**
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

---

## **📌 Step 5: Start the Agent**
### **For Temporary Execution (Manual Start)**
```powershell
.\run.cmd
```
*(The agent will stay online as long as this PowerShell window remains open.)*

### **For Automatic Execution (Install as a Windows Service)**
```powershell
.\svcInstall.cmd
.\svcStart.cmd
```

---

## **📌 Step 6: Verify the Agent in Azure DevOps**
1. **Go to** [https://dev.azure.com/intlkusto/_settings/agentpools](https://dev.azure.com/intlkusto/_settings/agentpools).
2. Click **SelfHostedPool**.
3. Ensure the **agent appears as Online and Available**.

---

## **📌 Step 7: Create a Service Connection to Azure**
### **7.1: Create a Service Principal in Azure**
1. **Go to Azure Portal** → **Azure Active Directory**.
2. Navigate to **App registrations** → **New registration**.
3. **Set:**
   - **Name:** `AzureDevOpsAgentSP`
   - **Supported account types:** "Single tenant"
   - **Redirect URI:** Leave blank
4. Click **Register**.
5. **Copy the Application (Client) ID and Directory (Tenant) ID**.
6. **Go to Certificates & Secrets → New client secret**.
7. **Copy the generated client secret**.

---

### **7.2: Assign RBAC Permissions in Azure**
1. **Go to Azure Portal → Resource Groups**.
2. Select the **resource group** where **Azure Data Explorer** exists.
3. Click **Access control (IAM) → Add role assignment**.
4. Assign these roles to the **Service Principal (`AzureDevOpsAgentSP`)**:
   - **"Contributor"**
   - **"Monitoring Reader"**
5. Click **Save**.

---

### **7.3: Add the Service Connection in Azure DevOps**
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

## **✅ Summary**
This guide ensures:
- 🔹 **Correct PAT permissions (expanded after creation)**
- 🔹 **Full agent installation with troubleshooting steps**
- 🔹 **Proper Azure authentication via Service Principal**
- 🔹 **RBAC assignments scoped to the Resource Group level**

🚀 **Your Azure DevOps Self-Hosted Agent is now fully configured!** 🚀
