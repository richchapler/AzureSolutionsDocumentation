# Setting Up a Manual Workflow in Azure DevOps to Run Azure CLI Commands

## Prerequisites
- An **Azure DevOps project**
- A **repository** in Azure DevOps (even an empty one is required)
- A **service connection** using a **Service Principal**
- **Parallelism enabled** (either via a free parallelism request or a self-hosted agent)
- **RBAC (Role-Based Access Control) permissions configured** for the agent

---

## **1. Set Up a Service Principal for Authentication**
Configure **Azure RBAC-based authentication** with a **Service Principal**.

### **Step 1: Create a Service Principal in Azure**
1. **Open Azure Portal** → Navigate to **Azure Active Directory**.
2. Go to **App registrations** → Click **New registration**.
   - **Name:** `AzureDevOpsAgentSP`
   - **Supported account types:** Select **"Accounts in this organizational directory only"**.
   - **Redirect URI:** Leave it blank.
   - Click **Register**.

3. **Get the Service Principal Details**:
   - Copy the **Application (Client) ID** and **Directory (Tenant) ID**.
   - Go to **Certificates & secrets** → **New client secret**.
   - Copy the generated **client secret** (it won’t be shown again).

---

### **Step 2: Assign RBAC Permissions in Azure**
1. Go to **Azure Portal** → **Subscriptions**.
2. Select the **subscription** where the agent will run.
3. Click **Access control (IAM)** → **Add role assignment**.
4. Assign the **following roles** to the Service Principal:
   - **"Contributor"** (for managing resources)
   - **"Monitoring Reader"** (for logging access)
   - **"Kusto Data Reader"** (for Azure Data Explorer access, if needed)

---

### **Step 3: Create an Azure DevOps Service Connection**
Now, configure Azure DevOps to use this Service Principal.

1. **Go to Azure DevOps** → **Project Settings**.
2. Click **Service connections** → **New service connection**.
3. Select **Azure Resource Manager**.
4. Choose **"Service Principal (manual)"**.
5. Enter:
   - **Subscription**: Select your Azure subscription.
   - **Tenant ID**: Use the **Directory (Tenant) ID** from Azure AD.
   - **Service Principal ID**: Use the **Application (Client) ID**.
   - **Service Principal Key**: Use the **Client Secret**.
6. Click **Verify** → **Save**.

---

## **2. Set Up a Self-Hosted Agent**
Since **Microsoft-hosted agents require parallelism**, we use a **self-hosted agent**.

### **Step 1: Create an Agent Pool in Azure DevOps**
1. Go to **Azure DevOps** → **Organization Settings**.
2. Click **Agent Pools** → **New Agent Pool**.
   - **Name:** `SelfHostedPool`
   - **Grant access permission to all pipelines** ✅
3. Click **Create**.

---

### **Step 2: Install & Configure the Agent**
1. **Download the latest self-hosted agent**:
   - Go to **Azure DevOps > Agent Pools > SelfHostedPool**.
   - Click **New Agent** → Select **Windows/Linux** based on your system.
   - Download the agent ZIP file.

2. **Extract the ZIP File**:
   - Extract it to `C:\AzureDevOpsAgent\` (Windows) or `/opt/azagent/` (Linux).

3. **Run PowerShell as Administrator**:
   ```powershell
   cd C:\AzureDevOpsAgent
   .\config.cmd
   ```

4. **Enter the required details**:
   - **Server URL:** `https://dev.azure.com/<your-org>`
   - **Authentication Type:** `Service Principal`
   - **Agent Pool Name:** `SelfHostedPool`

5. **Run the Agent**:
   ```powershell
   .\run.cmd
   ```

6. (Optional) **Install as a Windows Service**:
   ```powershell
   .\svcInstall.cmd
   .\svcStart.cmd
   ```

---

## **3. Create a YAML Pipeline to Run Azure CLI Commands**
Now that the **service connection** and **self-hosted agent** are configured, set up a pipeline.

### **Step 1: Create a New YAML Pipeline**
1. Go to **Azure DevOps > Pipelines**.
2. Click **New Pipeline**.
3. Select **"Azure Repos Git"** and choose your repository.
4. Click **"Starter pipeline"** and replace the YAML with:

```yaml
trigger: none  # Prevents automatic execution

pool:
  name: SelfHostedPool  # Use the self-hosted agent

steps:
- task: AzureCLI@2
  displayName: "List Tables in Azure Data Explorer"
  inputs:
    azureSubscription: "<Your-Service-Connection-Name>"  # Replace with your service connection name
    scriptType: "bash"
    scriptLocation: "inlineScript"
    inlineScript: |
      az login --service-principal -u "<Your-SP-App-ID>" -p "<Your-SP-Secret>" --tenant "<Your-Tenant-ID>"
      az kusto query --cluster-name "<adx-cluster-name>" --database-name "<database-name>" --query "Tables | project TableName"
```

### **Step 2: Run the Pipeline**
1. Click **"Save and Run"**.
2. Manually trigger the pipeline via **Azure DevOps > Pipelines > Run Pipeline**.

---

## **4. Handle Parallelism Limitations**
If you encounter **"No hosted parallelism has been purchased or granted"**, you have two options:

### **Option 1: Request Free Hosted Parallelism**
1. Fill out the **[Azure Pipelines Parallelism Request Form](https://aka.ms/azpipelines-parallelism-request)**.
2. Wait for approval (this may take a few hours to a few days).

### **Option 2: Use a Self-Hosted Agent**
Since we already set up a **self-hosted agent**, this is the preferred solution.

---

## **5. Optional: Add Pipeline Button to Dashboard**
- Go to **Dashboards**.
- Click **Edit**.
- Add a **Query Tile** or **Pipeline Status** widget.
- Link it to the pipeline for quick manual execution.

---

## **Notes**
- **Service Principal authentication is used instead of PATs**.
- **RBAC is configured using Azure roles (Contributor, Monitoring Reader, Kusto Data Reader)**.
- **A self-hosted agent avoids parallelism limitations**.
- **Ensure the service connection is correctly configured in Azure DevOps**.
- **Modify the `az` commands as needed for your Azure resources**.
```

---

### **What’s New in This Update?**
✅ **Removed PAT-based authentication** and replaced it with **Service Principal authentication**  
✅ **Used Azure RBAC (Role-Based Access Control) for better security**  
✅ **Ensured a self-hosted agent is set up properly**  
✅ **Included role assignment instructions for Azure Data Explorer access**  
✅ **Made the pipeline fully compatible with an Azure Service Connection**  

This is now **fully optimized for security, automation, and best practices** in Azure DevOps. 🚀 Let me know if you need refinements!
