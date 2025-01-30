# Setting Up a Manual Workflow in Azure DevOps to Run Azure CLI Commands

## Prerequisites
- An **Azure DevOps project**
- A **repository** in Azure DevOps (even an empty one is required)
- A **service connection** with appropriate permissions to Azure
- **Parallelism enabled** (either through a free parallelism request or a self-hosted agent)

## Steps

### 1. Create a New Azure DevOps YAML Pipeline
1. Go to **Azure DevOps** and navigate to your project
2. Click **Pipelines** in the left menu
3. Select **New Pipeline**
4. Choose **"Azure Repos Git"**
5. Select the repository you just created
6. Click **"Starter pipeline"** (or **"Existing YAML"** if adding later)

### 2. Define the YAML Pipeline
Replace the default YAML with the following:

```yaml
trigger: none  # Prevents automatic execution

pool:
  vmImage: ubuntu-latest  # Change to "windows-latest" for PowerShell

steps:
- task: AzureCLI@2
  displayName: "List Tables in Azure Data Explorer"
  inputs:
    azureSubscription: "<Your-Service-Connection-Name>"  # Replace with your service connection name
    scriptType: "bash"
    scriptLocation: "inlineScript"
    inlineScript: |
      az login --identity
      az kusto query --cluster-name "<adx-cluster-name>" --database-name "<database-name>" --query "Tables | project TableName"
```

### 3. Configure a Service Connection
1. **Go to** **Project Settings > Service Connections**
2. Click **New service connection**
3. Choose **Azure Resource Manager**
4. Select **Service Principal (automatic)** or **Managed Identity**
5. Assign the required **"Kusto Data Reader"** or **"Kusto Data Admin"** role to this identity in **Azure Data Explorer**:
   - Go to **Azure Portal > Azure Data Explorer > Cluster > Access Control (IAM)**
   - Click **"Add role assignment"**
   - Assign **"Kusto Data Reader"** or **"Kusto Data Admin"**
   - Select the **Service Principal** used in Azure DevOps
   - Click **Save**

### 4. Handle Parallelism Limitations
If you encounter an error stating **"No hosted parallelism has been purchased or granted"**, you have two options:

#### **Option 1: Request Free Hosted Parallelism**
1. Fill out the **[Azure Pipelines Parallelism Request Form](https://aka.ms/azpipelines-parallelism-request)**
2. Sign in with your **Azure DevOps account**
3. Provide your **Azure DevOps organization name**
4. Wait for approval (this may take a few hours to a few days)

#### **Option 2: Use a Self-Hosted Agent**
If you need an immediate solution:
1. Go to **Azure DevOps > Organization Settings > Agent Pools**
2. Click **"Add pool"**, name it (e.g., `SelfHostedPool`)
3. Click on the new pool and select **"New agent"**
4. Follow the instructions to **download the agent** and **register it** on your machine
5. Update your pipeline to use this self-hosted agent:
    ```yaml
    pool:
      name: SelfHostedPool  # Use the created agent pool
    ```

### 5. Save and Run the Pipeline Manually
1. Click **Save and Run** to test the pipeline
2. If you need to run it later:
   - Go to **Pipelines** in Azure DevOps
   - Select the pipeline
   - Click **"Run pipeline"** and start it manually

### 6. Optional: Add Pipeline Button to Dashboard
- Go to **Dashboards**
- Click **Edit**
- Add a **Query Tile** or **Pipeline Status** widget
- Link it to the pipeline for quick manual execution

## Notes
- Modify the `az` commands based on what needs to be done
- Use **environment variables** for credentials if needed
- Ensure the **agent** has the necessary permissions to access **Azure Data Explorer**
- If using **PowerShell**, change `scriptType: "bash"` to `scriptType: "ps"`
