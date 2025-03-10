# Data Factory + SSIS

------------------------- ------------------------- -------------------------

## Exercise 1: Configure On‑Prem

### Pre-Requisites

- Virtual Machine
  - SQL Server installed and Integration Services enabled
  - Ensure the VM can communicate with Azure by allowing outbound connections
  - Configure firewall settings to permit necessary traffic for the Integration Runtime

------------------------- -------------------------

### Install Visual Studio

Download and install Visual Studio 2022 Community from [Visual Studio Downloads](https://visualstudio.microsoft.com/downloads/)
- In the Visual Studio Installer, check to include the "Data storage and processing" workload

Launch Visual Studio and navigate to Extensions → Manage Extensions
- Search for and download "SQL Server Integration Services Projects 2022"
- Close Visual Studio and run "Microsoft.DataTools.IntegrationServices" from the "Downloads" folder
- Close and restart Visual Studio after installation to activate the SSIS extension

Once installed, immediately configure your project compatibility settings to ensure Azure compatibility:
- In Visual Studio, go to "Tools" → "Options"
- Under "Business Intelligence Designers" → "Integration Services Designers," set the "TargetServerVersion" to "SQL Server 2017"

This ensures that your SSIS packages are compatible with Azure-SSIS Integration Runtime, which specifically requires SQL Server 2017 compatibility.

------------------------- -------------------------

### Prepare Sample Data

Launch SQL Server Management Studio, connect to your local database, and then click "New Query".

Create demonstration tables and data by executing the following T-SQL:

```sql
CREATE DATABASE SSISDemoDB
GO

USE SSISDemoDB
GO

CREATE TABLE dbo.DemoProducts (
    ProductID INT IDENTITY(1,1) PRIMARY KEY,
    ProductName VARCHAR(100) NOT NULL,
    Quantity INT NOT NULL,
    UnitPrice DECIMAL(10,2) NOT NULL
)
GO

INSERT INTO dbo.DemoProducts (ProductName, Quantity, UnitPrice)
VALUES
('Widget A', 100, 9.99),
('Widget B', 50, 14.99),
('Widget C', 25, 29.99)
GO
```

------------------------- -------------------------

### Create Integration Services Project

Launch Visual Studio and click "Create a new project"  
- On the "Create a new project" popup, search for and select "Integration Services Project" then click "Next"  
- Name your project (for example, SSISDemoPipeline), confirm location, and click "Create"

In the Data Flow Designer, "Package.dtsx" package should be opened by default
- Drag a "Data Flow Task" onto the "Control Flow" canvas from the "SSIS Toolbox"
- Double-click the Data Flow Task to switch to the Data Flow designer

Drag "OLE DB Source" from "SSIS Toolbox" >> "Other Sources" onto the canvas  
- Double-click to open the "OLE DB Source Editor"
- Click "New" to create an "OLE DB Connection Manager":
  - Provider: `Native OLE DB\Microsoft OLE DB Driver for SQL Server`
  - Server or file name: `.` or `localhost`
  - Initial Catalog: `SSISDemoDB`
- Click "Test Connection" to confirm success then click "OK"
- Complete "OLE DB Source Editor" configuration:
  - OLE DB Connection Manager: `LocalHost.SSISDemoDB`
  - Data Access Mode: `Table or view`
  - Name of the Table or the View: `[dbo].[DemoProducts]`
- Click "Preview" to confirm success then click "OK"

Drag "Flat File Destination" from "SSIS Toolbox" >> "Other Destinations" onto the canvas  
- Create a data path from "OLE DB Source" to "Flat File Destination"
- Double-click to open the "Flat File Destination Editor"
- Click "New" to create an "Flat File Connection Manager":
  - Flat File Format: `Delimited`
- On the "Flat File Connection Manager Editor", browse to the "Downloads" folder, enter File Name `DemoProducts`, and then click "OK"
- Click "Mappings" and confirm success

Test the package by clicking "Start".

-------------------------

#### Expected Output

```
SSIS package "C:\Users\rchapler\source\repos\SSISDemoPipeline\SSISDemoPipeline\Package.dtsx" starting.
Information: 0x4004300A at Data Flow Task, SSIS.Pipeline: Validation phase is beginning.
Information: 0x4004300A at Data Flow Task, SSIS.Pipeline: Validation phase is beginning.
Information: 0x40043006 at Data Flow Task, SSIS.Pipeline: Prepare for Execute phase is beginning.
...
SSIS package "C:\Users\rchapler\source\repos\SSISDemoPipeline\SSISDemoPipeline\Package.dtsx" finished: Success.
```

------------------------- ------------------------- ------------------------- -------------------------

## Exercise 2: Configure Azure

### Pre-Requisites

- Azure subscription with appropriate permissions {e.g., `contributor`}
- Azure SQL Server
  - System-Assigned Identity enabled
  - Configured for both Entra and SQL Authentication
- Azure Data Factory with System-Assigned Managed Identity

------------------------- -------------------------

### `master` Database Permissions  

Open SQL Server Management Studio (SSMS), connect to the Azure SQL Server, and execute the following T-SQL on the `master` database:  

```sql
CREATE USER [{prefix}df] FROM EXTERNAL PROVIDER;
ALTER ROLE dbmanager ADD MEMBER [{prefix}df];
```

------------------------- -------------------------

### Integration Runtime

Open Data Factory Studio and navigate to "Manage" >> "Integration Runtimes"
- Click "+ New" under "Integration runtimes" and on the "Integration runtime setup" popout, choose "Azure‑SSIS" as the runtime type then click "Continue"
- Complete the "General settings" form, then click "Continue"

Complete the "Deployment settings" form, including:
- `Create SSIS catalog (SSISDB) hosted by Azure SQL Database server...`: CHECKED since we don't already have an existing SSISDB in Azure SQL
  - What it does: Automatically creates (or configures) an SSISDB database to store and manage your SSIS packages
  - Why you’d want it: If you don’t already have an SSISDB, checking this box ensures you have a centralized repository for deploying and executing packages in the cloud
  - If you don’t check it: You must have an existing SSISDB in Azure SQL Database and plan to manage it yourself
- `Catalog database server endpoint`: Select your Azure SQL Database Server endpoint (for example, `{prefix}ss.database.windows.net`)
  - What it does: Identifies the Azure SQL Database server where SSISDB will be hosted
- `Use Microsoft Entra authentication...`: UNCHECKED (since we'll use SQL authentication)
  - If unchecked: You’ll authenticate with the SQL server using SQL Authentication credentials
  - Why you'd want it: Simpler configuration for demonstration scenarios or environments without Azure AD setup
  - If you enable it: Authentication would be managed through Azure AD instead
- `Admin username`: Enter your SQL Authentication username for the Azure SQL Database server
  - `Admin password`: Enter your SQL Authentication password
- `Use dual standby Azure‑SSIS Integration Runtime...`: UNCHECKED since this is only for demonstration
  - What it does: Sets up a standby IR in a paired region for high availability/disaster recovery
  - Why you’d want it: Ensures minimal downtime if there’s a regional outage
  - If you don’t enable it: You’ll have a single IR instance without automatic failover
- `Catalog database service tier`: "S1" since this is only for demonstration
  - What it does: Determines performance, cost, and resource limits (e.g., S1, S2, etc.)
  - Why you’d want it: Higher tiers can handle more transactions and concurrency, but cost more. For testing or light workloads, S1 is typically sufficient
- `Create package stores...`: UNCHECKED since this is only for demonstration  
  - What it does: Lets you configure additional storage locations (e.g., file system, Azure Files, SQL) to hold SSIS packages outside of SSISDB
  - Why you’d want it: If you have custom package deployment needs or want to manage packages outside of SSISDB
  - If you don’t enable it: Packages are deployed solely to SSISDB, which is sufficient for most scenarios

Click "Test Connection" and confirm success, then click "Continue".

Confirm "Advanced Settings" configuration and then click "Continue".

Review "Summary" and then click "Create".

Wait for `Status` to change to "Running".

------------------------- -------------------------

### `SSISDB` Database Permissions  

Open SQL Server Management Studio (SSMS), connect to the Azure SQL Server, and execute the following T-SQL on the `SSISDB` database:  

```sql
CREATE USER [{prefix}df] FROM EXTERNAL PROVIDER;
ALTER ROLE ssis_admin ADD MEMBER [{prefix}df];
```

------------------------- ------------------------- ------------------------- -------------------------

## Exercise 3: Deploy Package

### Deploy Packages

- In Visual Studio, Solution Explorer, right-click the `SSISDemoPipeline` project and click "Deploy"
- On the resulting "Integration Services Deployment Wizard" interface, navigate to "Select Deployment Target"
  - Click deployment target "SSIS in Data Factory" and click "Next"
- Complete the "Select Destination" form:
  - Server name: `{prefix}ss.database.windows.net`
  - Authentication: `SQL Server Authentication` (along with Login and Password)
- Click "Connect" and then the resulting "Browse" button
  - On the resulting "Browse for Folder or Project", select the `SSISDB` folder
  - Click "New folder" and on the resulting "Create New Folder" enter Name `SSISDemoPipeline`
  - Click to select the `SSISDemoPipeline` and then click "OK"
- Back on the "Select Destination" form, click "Next >"
- Review selections, then click "Deploy", monitor progress, and then click "Close"

------------------------- -------------------------

### Verify Deployment

- Open SQL Server Management Studio and connect to the Azure SQL Server  
- Expand `Integration Services Catalogs → SSISDB` and confirm that your project and packages appear  
- Run the following queries to verify deployment:  

  - Check available folders:  
    ```sql
    SELECT name FROM SSISDB.catalog.folders
    ```  
  - Check available projects:  
    ```sql
    SELECT name FROM SSISDB.catalog.projects
    ```  
  - Check available packages:  
    ```sql
    SELECT name FROM SSISDB.catalog.packages
    ```  

- If the expected values do not appear, confirm that the deployment process completed successfully  

------------------------- ------------------------- ------------------------- -------------------------

## Exercise 4: Create Pipeline

- Navigate to Azure Data Factory Studio >> "Author" and create a new pipeline
- Drag an "Execute SSIS Package" activity onto the pipeline canvas and configure:
  - Azure-SSIS IR: `Azure-SSIS`
  - Windows authentication: UNCHECKED
  - Package location: `SSISDB`
  - Folder: `SSISDemoPipeline`
  - Project: `SSISDemoPipeline`
  - Package: `Package.dtsx`
  - Environment: BLANK
  - Logging Level: `Basic`

Click "Debug" to test the pipeline execution.

### Monitor and Verify

- Navigate to the "Monitor" tab in Data Factory Studio.
- Confirm the successful execution of your pipeline, indicated by a status of "Succeeded."

### Post-Deployment Considerations

- Update connection managers or environment references in your SSIS packages to point explicitly to Azure resources.
- Document any configuration changes or issues encountered during the migration and deployment process.

