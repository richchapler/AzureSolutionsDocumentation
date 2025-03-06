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

Below is the updated Azure section based on the latest documentation and supported scenario. Currently, the Execute SSIS Package activity in Azure Data Factory only works with an Azure‑SSIS Integration Runtime. Self‑Hosted Integration Runtime isn’t supported for executing SSIS packages via ADF. Use the following working scenario:

------------------------- ------------------------- ------------------------- -------------------------

## Azure

### Data Factory

- Log in to the [Azure Portal](https://portal.azure.com) and create a new Data Factory  
  - Click "Create a resource" and search for "Data Factory"  
  - Fill in the required details (name, subscription, resource group, region, etc.) and click "Review + create" then "Create"  
- Once deployed, open Data Factory Studio and navigate to the "Manage" tab, then select "Integration Runtimes"

------------------------- -------------------------

### Azure‑SSIS Integration Runtime

- In Data Factory Studio, click "+ New" under "Integration runtimes"  
- On the "Integration runtime setup" popout, choose "Azure‑SSIS" as the runtime type and click "Continue"  
- Configure the runtime:  
  - Provide a descriptive name (e.g., `AzureSSISIR`)  
  - Select the appropriate Azure region  
  - Choose the compute size and scale settings that match your workload  
  - Optionally, configure advanced settings (such as integration with an Azure SQL Database for the SSISDB catalog)  
- Click "Create" to deploy the Azure‑SSIS Integration Runtime  
- Once deployed, confirm the IR is listed and its status is "Running"

------------------------- -------------------------

### Deploy and Execute SSIS Packages with Azure‑SSIS IR

- Deploy your SSIS packages to the SSIS catalog (SSISDB) hosted on an Azure SQL Database or a SQL Managed Instance  
- In Data Factory Studio, navigate to the "Author" tab and create a new pipeline  
- Drag the "Execute SSIS Package" activity onto the pipeline canvas  
- Configure the activity with the following settings:  
  - Package location: Select the SSIS package from your SSISDB catalog  
  - Connection details: Provide necessary parameters, such as credentials and any runtime parameters  
- Assign the Azure‑SSIS Integration Runtime to the activity  
- Save and publish your pipeline  
- Trigger the pipeline and monitor its execution via the "Monitor" tab
