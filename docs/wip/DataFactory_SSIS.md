# Data Factory + SSIS

In this demonstration you will:
- Create an ADF instance.
- Set up a Self‑Hosted Integration Runtime (IR) that runs on your VM.
- Install and configure the IR on your on‑premises VM (hosting SSIS).
- Create a pipeline in ADF that calls an SSIS package.

This walkthrough assumes you already have an Azure subscription and a VM (with Windows and SQL Server/SSIS installed) that can communicate with Azure.

------------------------- ------------------------- ------------------------- -------------------------

## On‑Prem

### Virtual Machine

Provision the Virtual Machine:
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
Information: 0x40043007 at Data Flow Task, SSIS.Pipeline: Pre-Execute phase is beginning.
Information: 0x402090DC at Data Flow Task, Flat File Destination [2]: The processing of file "C:\Users\rchapler\Downloads\DemoProducts.txt" has started.
Information: 0x4004300C at Data Flow Task, SSIS.Pipeline: Execute phase is beginning.
Information: 0x40043008 at Data Flow Task, SSIS.Pipeline: Post Execute phase is beginning.
Information: 0x402090DD at Data Flow Task, Flat File Destination [2]: The processing of file "C:\Users\rchapler\Downloads\DemoProducts.txt" has ended.
Information: 0x4004300B at Data Flow Task, SSIS.Pipeline: "Flat File Destination" wrote 3 rows.
Information: 0x40043009 at Data Flow Task, SSIS.Pipeline: Cleanup phase is beginning.
SSIS package "C:\Users\rchapler\source\repos\SSISDemoPipeline\SSISDemoPipeline\Package.dtsx" finished: Success.
```

------------------------- ------------------------- ------------------------- -------------------------

## Azure

### Data Factory

- Log in to the [Azure Portal](https://portal.azure.com) and create a new Data Factory
- Navigate to Data Factory Studio >> Manage >> Integration Runtimes
- Click "+ New" and on the resulting popout, ...






- Under "Integration runtimes," click "New" and choose "Self‑hosted"
- Provide a descriptive name and, optionally, a description
- Click "Create" to generate the integration runtime and its registration key
  - Copy the registration key for installation on your on‑premises VM
- Verify that the IR is listed (it may initially show as offline until the on‑premises installation is complete)

------------------------- -------------------------

### Install the Self‑Hosted Integration Runtime on Your On‑Premises VM

- From the ADF Integration Runtime configuration screen, download the Self‑Hosted Integration Runtime installer
- Run the installer on your VM
  - When prompted, enter the registration key you saved earlier
- Follow the installation prompts until completion
- Verify that the runtime service is running on your VM (it should appear as "Online" in ADF once connected)

------------------------- -------------------------

### Verify Connectivity and Test Integration

- In your Data Factory instance, navigate to the "Monitor" tab
- Check that the Self‑Hosted Integration Runtime displays as "Online"
- Use the test connectivity feature in ADF to ensure the IR can reach all required on‑premises data sources
  - If issues arise, review firewall settings and network configurations on your VM

------------------------- -------------------------

### Create and Configure the Pipeline to Execute SSIS Packages

- In the ADF "Author" tab, create a new pipeline
- Drag the "Execute SSIS Package" activity into your pipeline
  - Configure the activity with:
    - Package location: Specify the path on your on‑premises SSIS server where your package is stored
    - Connection details: Provide necessary parameters, such as the package path and runtime parameters
- Assign the Self‑Hosted Integration Runtime to the activity
- Save and publish your changes

------------------------- -------------------------

### Publish, Trigger, and Monitor the Pipeline

- Trigger the pipeline to execute your SSIS package
- In the "Monitor" tab of ADF, track the pipeline run
  - Review execution details, including any warnings or errors
- If issues occur, consult both ADF logs and on‑premises SSIS/IR logs for troubleshooting
