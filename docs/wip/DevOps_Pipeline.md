# Setting Up a Manual Workflow in Azure DevOps to Run Azure CLI Commands

## Prerequisites
- An Azure DevOps project
- A repository in Azure DevOps
- Azure CLI installed on the agent
- A service connection with appropriate permissions to Azure

## Steps

### 1. Create a New Azure DevOps Pipeline
1. Go to **Azure DevOps** and navigate to your project
2. Click **Pipelines** in the left menu
3. Select **New Pipeline**
4. Choose **"Use the classic editor"** (or use YAML if preferred)
5. Select your repository and click **Continue**
6. Choose **"Empty job"**

### 2. Add a Manual Trigger
- In the **Agent job** settings:
  - Set **"Enable continuous integration"** to **Off**
  - Under **Triggers**, ensure no automatic triggers are enabled

### 3. Add a Bash or PowerShell Task
- Click **+** to add a task to the **Agent job**
- Select **"Bash"** (for Linux agents) or **"PowerShell"** (for Windows agents)
- Configure the script:
  - For **Bash**, enter:
    ```sh
    az login --identity
    az kusto cluster list --resource-group "<resource-group-name>"
    ```
  - For **PowerShell**, enter:
    ```powershell
    az login --identity
    az kusto cluster list --resource-group "<resource-group-name>"
    ```

### 4. Configure Service Connection
- Go to **Project Settings > Service connections**
- Add a new **Azure Resource Manager** service connection
- Use **Managed Identity** or **Service Principal**
- Assign the required permissions to the identity

### 5. Save and Queue Manually
- Click **Save & Queue** > **Save**
- When ready, click **Run Pipeline**
- The job will execute and display output in the logs

### 6. Optional: Add Pipeline Button to Dashboard
- Go to **Dashboards**
- Click **Edit**
- Add a **Query Tile** or **Pipeline Status** widget
- Link it to the pipeline for quick manual execution

## Notes
- Modify the `az` commands based on what needs to be done
- Use environment variables for credentials if needed
- Ensure the agent has the necessary permissions to access Azure resources
