# Data Factory + SSIS

------------------------- ------------------------- -------------------------

## Exercise 1: Configure On‑Prem

### Pre-Requisite Resources

Provision a Virtual Machine:
- Set up a Windows Server virtual machine
- Install SQL Server with the Integration Services feature enabled
  
Network Configuration:
- Ensure the VM can communicate with Azure by allowing outbound connections
- Configure firewall settings to permit necessary traffic for the Integration Runtime

------------------------- -------------------------

### Visual Studio

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

### SQL Server Management Studio

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

### SQL Server Integration Services

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

------------------------- -------------------------

### Section Wrap-Up

In this section, we:
- Set up our on‑prem environment by provisioning a Windows Server VM with SQL Server Integration Services
- Configured Visual Studio with the required data tools
- Created demonstration data using SSMS
- Developed and tested an SSIS package locally, ensuring that all components work together as expected

Next, we'll migrate these SSIS packages to the cloud by deploying them to an Azure‑hosted SSISDB and configuring Azure Data Factory to execute them via an Azure‑SSIS Integration Runtime.

------------------------- ------------------------- ------------------------- -------------------------

## Exercise 2: Azure Data Factory

(Keep this section as-is, no further edits recommended)

- After creating Azure-SSIS Integration Runtime, verify it is in "Running" state before proceeding

------------------------- ------------------------- ------------------------- -------------------------

## Exercise 3: Migrate to Azure

Important: Before migrating, complete Exercise 2: Azure Data Factory to ensure Azure-SSIS IR and SSISDB availability.

### Prepare Project for Azure Compatibility

In Visual Studio, update the target server version:
- Right-click your SSIS project in Solution Explorer, select "Properties"
- Under "Configuration Properties" → "General", set "TargetServerVersion" to "SQL Server 2017"
- Click "OK" to apply settings
- Click "Build" → "Build Solution" to generate the deployment (.ispac) file

#### Expected Output
```
Build started...
Build complete -- 0 errors, 0 warnings
```

### Deploy to Azure SSISDB

In Visual Studio Solution Explorer:
- Right-click your project and select "Deploy"
- In the Integration Services Deployment Wizard, select deployment target "SSIS in Data Factory"
- Enter your Azure SQL Database server name (`{prefix}ss.database.windows.net`)
- Choose "SQL Server Authentication" and provide credentials
- When selecting deployment path, click "Browse..." and confirm the correct folder path within SSISDB catalog is selected
- Follow prompts to complete deployment

After deployment completes, verify the packages appear in SSISDB by connecting to Azure SQL with SSMS or Azure Data Studio.

### Post‑Migration Considerations

- Update connection managers or environment references to use Azure resources
- Test migrated packages using SSMS or via the Azure‑SSIS Integration Runtime
- Document any configuration changes or issues encountered during migration

------------------------- ------------------------- ------------------------- -------------------------

## Exercise 4: Test Pipeline

- Clearly verify successful completion in the "Monitor" tab, indicated by the pipeline status "Succeeded"
