# Data Factory >> SSIS

In this demonstration you will:
- Create an ADF instance.
- Set up a Self‑Hosted Integration Runtime (IR) that runs on your VM.
- Install and configure the IR on your on‑premises VM (hosting SSIS).
- Create a pipeline in ADF that calls an SSIS package.

This walkthrough assumes you already have an Azure subscription and a VM (with Windows and SQL Server/SSIS installed) that can communicate with Azure.

------------------------- ------------------------- ------------------------- -------------------------

## Step 1: On‑Premises Environment

### 1.1 Virtual Machine

- Provision the Virtual Machine:
  - Set up a Windows Server virtual machine
  - Install SQL Server with the Integration Services feature enabled
  
- Network Configuration:
  - Ensure the VM can communicate with Azure by allowing outbound connections
  - Configure firewall settings to permit necessary traffic for the Integration Runtime

------------------------- -------------------------

### 1.2 SQL Server Data Tools (SSDT)

- Download and install Visual Studio 2022 Community from [Visual Studio Downloads](https://visualstudio.microsoft.com/downloads/)
  - In the Visual Studio Installer, check to include the "Data storage and processing" workload
- Launch Visual Studio 2022 and navigate to Extensions → Manage Extensions
  - Search for and download "SQL Server Integration Services Projects 2022"
  - Close Visual Studio and run "Microsoft.DataTools.IntegrationServices" from the "Downloads" folder

------------------------- -------------------------

### 1.3 SQL Server Integration Services (SSIS) Project

- Open Visual Studio with SSDT:
  - Launch Visual Studio.
  - Select "Create a new project".
  - Choose "Integration Services Project" from the list of templates.

2. Set Up Your Project:
   - Name your project (e.g., `SSISDemoPipeline`).
   - Choose a suitable location and click "Create".

#### b. Develop the SSIS Package

1. Add or Use the Default Package:
   - In Solution Explorer, open the default package (usually named `Package.dtsx`) or add a new package if needed.

2. Design the Control Flow:
   - Drag a Data Flow Task:
     - From the SSIS Toolbox, drag a Data Flow Task onto the Control Flow canvas.
   - Configure Additional Tasks (Optional):
     - Optionally, add tasks like Execute SQL Task or Script Task to perform preliminary operations (e.g., creating a staging table).

3. Build the Data Flow:
   - Switch to the Data Flow Designer:
     - Double-click the Data Flow Task to open the Data Flow designer.
   - Add a Source Component:
     - Drag a Flat File Source onto the canvas.
     - Configure it to read from a sample CSV or text file.
     - Define columns, delimiters, and data types as required.
   - Add a Transformation (Optional):
     - Drag a Derived Column or Data Conversion component to demonstrate transformation capabilities.
     - Configure it to modify or convert incoming data.
   - Add a Destination Component:
     - Drag an OLE DB Destination (or SQL Server Destination) onto the Data Flow canvas.
     - Set up a connection manager to connect to your sample SQL Server database.
     - Map the source columns to the destination table columns.
     
4. Test the Package:
   - Run the package locally within Visual Studio to ensure that data flows from the source to the destination as expected.
   - Confirm that any transformations or data mappings are working correctly.

---

### 1.4 Final Preparations for the On‑Premises Demonstration

- Validate the Environment:
  - Ensure that the SSIS package runs successfully on your VM.
  - Confirm that the VM’s firewall and network settings allow the package and Integration Runtime to communicate with Azure.
  
- Document Settings:
  - Note down connection strings, file paths, and any configuration parameters used.
  - Keep records of testing results and any error logs for troubleshooting during your demonstration.

- Backup the Package:
  - Save your SSIS project and back up the package for reuse in your Azure Data Factory pipeline demonstration.

---

By completing these expanded steps, you set up an on‑premises environment that not only connects to Azure Data Factory via the Self‑Hosted Integration Runtime but also demonstrates a basic SSIS-based pipeline that moves and transforms data. This prepares you to show a full end-to-end integration scenario from on‑premises SSIS execution to cloud-based orchestration.

---

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
