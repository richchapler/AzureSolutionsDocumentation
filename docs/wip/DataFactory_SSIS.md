# Data Factory + SSIS

In this demonstration you will:
- Create an ADF instance.
- Set up a Self‑Hosted Integration Runtime (IR) that runs on your VM.
- Install and configure the IR on your on‑premises VM (hosting SSIS).
- Create a pipeline in ADF that calls an SSIS package.

This walkthrough assumes you already have an Azure subscription and a VM (with Windows and SQL Server/SSIS installed) that can communicate with Azure.

------------------------- ------------------------- ------------------------- -------------------------

## Step 1: On‑Premises Environment

### 1.1 Virtual Machine

Provision the Virtual Machine:
- Set up a Windows Server virtual machine
- Install SQL Server with the Integration Services feature enabled
  
Network Configuration:
- Ensure the VM can communicate with Azure by allowing outbound connections
- Configure firewall settings to permit necessary traffic for the Integration Runtime

------------------------- -------------------------

### 1.2 Visual Studio

Download and install Visual Studio 2022 Community from [Visual Studio Downloads](https://visualstudio.microsoft.com/downloads/)
- In the Visual Studio Installer, check to include the "Data storage and processing" workload

Launch Visual Studio and navigate to Extensions → Manage Extensions
- Search for and download "SQL Server Integration Services Projects 2022"
- Close Visual Studio and run "Microsoft.DataTools.IntegrationServices" from the "Downloads" folder

------------------------- -------------------------

### 1.3 SQL Server Management Studio

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

### 1.4 SQL Server Integration Services

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

## Step 2: Create an Azure Data Factory Instance

1. Log in to the Azure Portal.
2. Create a New Data Factory:
   - Click "Create a resource" and search for "Data Factory".
   - Fill in the required details (name, resource group, region, etc.) and create the instance.
3. Wait for Deployment:
   - Wait until deployment completes before proceeding.

---

## Step 3: Configure a Self‑Hosted Integration Runtime in ADF

1. Navigate to Your Data Factory Instance:
   - Go to the "Manage" tab (usually on the left-side panel).
2. Create a New Integration Runtime:
   - Click "Integration runtimes" and then "New".
   - Choose "Self‑hosted" as the runtime type.
   - Provide a name and description.
3. Generate the Registration Key:
   - A key (or multiple keys) will be provided for installation. Save this information for later use.

---

## Step 4: Install the Integration Runtime on the On‑Premises VM

1. Download the Setup:
   - From the ADF Integration Runtime configuration screen, download the Self‑Hosted Integration Runtime installer.
2. Run the Installer on Your VM:
   - Launch the installer.
   - During setup, enter the registration key that you saved from ADF.
3. Complete Installation:
   - Follow the prompts until installation is finished.
   - Verify that the runtime service is running on the VM.

---

## Step 5: Verify Connectivity and Configuration

1. Back in ADF:
   - Go to the Integration Runtime’s monitoring or status page.
   - Confirm that your on‑premises IR shows as "Online".
2. Connectivity Test:
   - Use ADF’s test connectivity feature to ensure the runtime can reach required data sources.
   - If any issues arise, review firewall and network settings on the VM.

---

## Step 6: Create and Configure the Pipeline to Execute SSIS Packages

1. Build a Pipeline:
   - In ADF, go to the "Author" tab and create a new pipeline.
2. Add an SSIS Activity:
   - Drag the "Execute SSIS Package" activity into your pipeline.
   - Configure the activity by specifying:
     - The package location (on the on‑premises SSIS server).
     - Connection details (e.g., package path, parameters, and any necessary configurations).
3. Assign the Integration Runtime:
   - Set the activity’s Integration Runtime to the Self‑Hosted IR you configured.
4. Publish and Trigger:
   - Save and publish your changes.
   - Trigger the pipeline to execute your SSIS package.

---

## Step 7: Monitor and Troubleshoot

1. Monitor Pipeline Runs:
   - Use the "Monitor" tab in ADF to track the execution of your pipeline.
   - Check for any errors or warnings.
2. Review Logs on the VM:
   - If the package fails, review the logs both in ADF and on your on‑premises VM (SSIS and IR logs).
3. Troubleshooting Tips:
   - Verify network connectivity and firewall rules.
   - Ensure the SSIS package and related configurations are correct.
   - Re-run connectivity tests from within ADF’s IR settings if needed.

---

## Conclusion

By following these steps, you demonstrate how to connect Azure Data Factory to a SQL Server Integration Services pipeline using a Self‑Hosted Integration Runtime on an on‑premises VM. This setup allows ADF to trigger and monitor SSIS package executions remotely, effectively bridging cloud and on‑premises environments.

Feel free to adjust or expand any of these steps to better fit your demonstration scenario or include additional details such as screenshots and troubleshooting examples.

---

This documentation now standardizes the formatting and incorporates the latest SSDT installation insights without modifying your original structure.
