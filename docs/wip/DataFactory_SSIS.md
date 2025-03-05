# Data Factory >> SSIS

In this demonstration you will:
- Create an ADF instance.
- Set up a Self‑Hosted Integration Runtime (IR) that runs on your VM.
- Install and configure the IR on your on‑premises VM (hosting SSIS).
- Create a pipeline in ADF that calls an SSIS package.

This walkthrough assumes you already have an Azure subscription and a VM (with Windows and SQL Server/SSIS installed) that can communicate with Azure.

## Prerequisites

- **Azure Subscription:** To create an Azure Data Factory instance.
- **On‑Premises VM:** A virtual machine running Windows Server with SQL Server and SSIS installed.
- **Network Connectivity:** Ensure your VM can reach Azure (or has a VPN/ExpressRoute if behind a firewall).
- **Permissions:** Administrative rights on the VM to install software and configure firewall/network settings.

---

## Step 1: Prepare the On‑Premises Environment

1. **Provision the VM:**
   - Set up a Windows Server VM.
   - Install SQL Server with the Integration Services feature enabled.
2. **Network Configuration:**
   - Ensure that outbound connections to Azure are allowed.
   - Adjust firewall settings if necessary to allow communication for the Integration Runtime.

---

## Step 2: Create an Azure Data Factory Instance

1. **Log in to the Azure Portal.**
2. **Create a new Data Factory:**
   - Click **"Create a resource"** and search for **"Data Factory"**.
   - Fill in the required details (name, resource group, region, etc.) and create the instance.
3. **Wait for deployment** to complete before proceeding.

---

## Step 3: Configure a Self‑Hosted Integration Runtime in ADF

1. **Navigate to your Data Factory instance:**
   - Go to the **"Manage"** tab (usually on the left-side panel).
2. **Create a new Integration Runtime:**
   - Click **"Integration runtimes"** and then **"New"**.
   - Choose **"Self‑hosted"** as the runtime type.
   - Provide a name and description.
3. **Generate the Registration Key:**
   - A key (or multiple keys) will be provided for installation. Save this information for later use.

---

## Step 4: Install the Integration Runtime on the On‑Premises VM

1. **Download the Setup:**
   - From the ADF Integration Runtime configuration screen, download the Self‑Hosted Integration Runtime installer.
2. **Run the Installer on your VM:**
   - Launch the installer.
   - During setup, enter the registration key that you saved from ADF.
3. **Complete Installation:**
   - Follow the prompts until installation is finished.
   - Verify that the runtime service is running on the VM.

---

## Step 5: Verify Connectivity and Configuration

1. **Back in ADF:**
   - Go to the Integration Runtime’s monitoring or status page.
   - Confirm that your on‑premises IR shows as **"Online"**.
2. **Connectivity Test:**
   - Use ADF’s test connectivity feature to ensure the runtime can reach required data sources.
   - If any issues arise, review firewall and network settings on the VM.

---

## Step 6: Create and Configure the Pipeline to Execute SSIS Packages

1. **Build a Pipeline:**
   - In ADF, go to the **"Author"** tab and create a new pipeline.
2. **Add an SSIS Activity:**
   - Drag the **"Execute SSIS Package"** activity into your pipeline.
   - Configure the activity by specifying:
     - The package location (on the on‑premises SSIS server).
     - Connection details (e.g., package path, parameters, and any necessary configurations).
3. **Assign the Integration Runtime:**
   - Set the activity’s Integration Runtime to the Self‑Hosted IR you configured.
4. **Publish and Trigger:**
   - Save and publish your changes.
   - Trigger the pipeline to execute your SSIS package.

---

## Step 7: Monitor and Troubleshoot

1. **Monitor Pipeline Runs:**
   - Use the **"Monitor"** tab in ADF to track the execution of your pipeline.
   - Check for any errors or warnings.
2. **Review Logs on the VM:**
   - If the package fails, review the logs both in ADF and on your on‑premises VM (SSIS and IR logs).
3. **Troubleshooting Tips:**
   - Verify network connectivity and firewall rules.
   - Ensure the SSIS package and related configurations are correct.
   - Re-run connectivity tests from within ADF’s IR settings if needed.

---

## Conclusion

By following these steps, you demonstrate how to connect Azure Data Factory to a SQL Server Integration Services pipeline using a Self‑Hosted Integration Runtime on an on‑premises VM. This setup allows ADF to trigger and monitor SSIS package executions remotely, bridging cloud and on‑premises environments effectively.

Feel free to adjust or expand any of these steps to better fit your demonstration scenario or to include additional details such as screenshots and troubleshooting examples.
