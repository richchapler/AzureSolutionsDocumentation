# Setting Up a Manual Workflow in Azure DevOps to Run Azure CLI Commands

## Prerequisites
- An **Azure DevOps project**
- A **repository** in Azure DevOps (even an empty one is required)
- A **service connection** using a **Service Principal**
- **Parallelism enabled** (either via a free parallelism request or a self-hosted agent)
- **RBAC (Role-Based Access Control) permissions configured at the resource group level**
- **Kusto (ADX) database permissions assigned**

---

## **1. Set Up a Service Principal for Authentication**

### **Step 1: Create a Service Principal in Azure**
1. Open **Azure Portal** → Navigate to **Azure Active Directory**.
2. Go to **App registrations** → Click **New registration**.
   - **Name:** `AzureDevOpsAgentSP`
   - **Supported account types:** Select **"Accounts in this organizational directory only"**.
   - **Redirect URI:** Leave it blank.
   - Click **Register**.
3. Copy the **Application (Client) ID** and **Directory (Tenant) ID**.
4. Go to **Certificates & secrets** → **New client secret**.
5. Copy the generated **client secret** (it won’t be shown again).

---

## **2. Assign RBAC Permissions at the Resource Group Level**
1. Open **Azure Portal** → Go to **Resource Groups**.
2. Select the **resource group** that contains the Azure Data Explorer cluster.
3. Click **Access control (IAM)** → **Add role assignment**.
4. Assign the following roles to **`AzureDevOpsAgentSP`**:
   - **"Contributor"** (for managing resources in the resource group)
   - **"Monitoring Reader"** (for logging and monitoring)
5. Select **"Assign access to: User, group, or service principal"**.
6. Search for **`AzureDevOpsAgentSP`** and select it.
7. Click **Save**.

---

## **3. Assign Permissions in Azure Data Explorer (ADX)**
Since **Azure RBAC does not have a built-in Kusto Data Reader role**, assign permissions directly in **Azure Data Explorer (ADX)**.

1. Open **Azure Portal** → Navigate to **Azure Data Explorer (ADX)**.
2. Select the **ADX Cluster**.
3. Click **Databases** and select the **database** you want to grant access to.
4. Click **Permissions** → **Add Role Assignment**.
5. Assign the **Service Principal (`AzureDevOpsAgentSP`)** one of the following roles:
   - **Database User** (Read-only access to execute queries)
   - **Database Admin** (If the pipeline needs to modify data)
6. Click **Save**.

---

## **4. Set Up a Self-Hosted Agent in Azure DevOps (Windows)**

### **Step 4.1: Create an Agent Pool in Azure DevOps**
1. **Go to Azure DevOps** → **Organization Settings**.
2. Click **Agent Pools** → **New Agent Pool**.
3. Enter the following details:
   - **Pool Name:** `SelfHostedPool`
   - **Auto-provision this agent pool in all projects** ✅ *(Ensures all projects can use it.)*
4. Click **Create**.

### **Step 4.2: Configure Agent Pool Security**
1. **Go to Azure DevOps** → **Organization Settings** → **Agent Pools**.
2. Click on `SelfHostedPool`.
3. Open the **Security** tab.
4. Add the **Service Principal (`AzureDevOpsAgentSP`)** and assign:
   - ✅ **Administrator** *(This grants full control, no other roles are needed.)*
5. Click **Save**.

### **Step 4.3: Download and Extract the Self-Hosted Agent**
1. **Go to Azure DevOps** → **Organization Settings** → **Agent Pools**.
2. Select **SelfHostedPool** → Click **New Agent**.
3. Choose **Windows** as the operating system.
4. Download the **agent ZIP file**.
5. **Extract** the ZIP file to:
   ```plaintext
   C:\AzureDevOpsAgent\
   ```
   *(Ensure the path does not contain spaces.)*

### **Step 4.4: Configure the Agent**
1. **Open PowerShell as Administrator**.
2. Navigate to the extracted agent directory:
   ```powershell
   cd C:\AzureDevOpsAgent
   ```
3. Run the configuration script:
   ```powershell
   .\config.cmd
   ```
4. Enter the required details:
   - **Server URL:** `https://dev.azure.com/<your-org>`
   - **Authentication Type:** `Service Principal`
   - **Agent Pool Name:** `SelfHostedPool`

### **Step 4.5: Start the Agent**
#### **For Temporary Execution (Manual Start)**
```powershell
.\run.cmd
```
*(The agent will stay online as long as this PowerShell window remains open.)*

#### **For Automatic Execution (Install as a Windows Service)**
To ensure the agent starts automatically after a reboot:
```powershell
.\svcInstall.cmd
.\svcStart.cmd
```

### **Step 4.6: Verify the Agent in Azure DevOps**
1. **Go to Azure DevOps** → **Organization Settings** → **Agent Pools**.
2. Click **SelfHostedPool**.
3. Ensure that the agent appears as **Online** and **Available**.

---

## **5. Create a YAML Pipeline to Run Azure CLI Commands**

### **Step 5.1: Create a New YAML Pipeline**
1. **Go to Azure DevOps** → **Pipelines**.
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

### **Step 5.2: Run the Pipeline**
1. Click **"Save and Run"**.
2. Manually trigger the pipeline via **Azure DevOps > Pipelines > Run Pipeline**.

---

## **6. Handle Parallelism Limitations**
If you encounter **"No hosted parallelism has been purchased or granted"**, you have two options:

### **Option 1: Request Free Hosted Parallelism**
1. Fill out the **[Azure Pipelines Parallelism Request Form](https://aka.ms/azpipelines-parallelism-request)**.
2. Wait for approval (this may take a few hours to a few days).

### **Option 2: Use a Self-Hosted Agent**
Since we already set up a **self-hosted agent**, this is the preferred solution.

---

## **7. Optional: Add Pipeline Button to Dashboard**
- Go to **Dashboards**.
- Click **Edit**.
- Add a **Query Tile** or **Pipeline Status** widget.
- Link it to the pipeline for quick manual execution.
