# PostgreSQL

## Prepare Resources

### Azure Database for PostgreSQL Flexible Server

* **Create a New Server in the Azure Portal**  
  Log in to the [Azure Portal](https://portal.azure.com), click **Create a resource**, and select **Azure Database for PostgreSQL Flexible Server**.

* **Configure Basic Settings (Compute + Storage)**  
  Enter the required details such as a unique **Server Name**, choose the **Region** closest to you, and set the **Admin Username and Password**. Click **Next** to continue.

* **Configure Networking**  
  In the **Networking** step, select **Public Access** if you want to connect from outside Azure (for example, using pgAdmin or Cloud Shell from your local machine).  
  For testing, set the allowed IP range to **0.0.0.0 to 255.255.255.255** to allow connections from any IP (for production, restrict this range to your trusted IPs). Click **Next** and complete the wizard.

* **Review and Create**  
  Review your configuration settings and click **Create** to deploy the server.

* **Backup Note**  
  If you’re replacing an existing server, remember to back up your data using tools like `pg_dump` before deletion.

------------------------- -------------------------

#### Monitoring and Diagnostics

* **Create a Log Analytics Workspace**  
  * In the Azure Portal, click **Create a resource** and search for **Log Analytics workspace**.  
  * Click **Create** and fill in the required details such as Workspace name, Subscription, Resource Group, and Region.  
  * Click **Review + create**, then **Create** to instantiate your new Log Analytics workspace.

* **Note the Workspace Details**  
  * Once created, note the Workspace ID and primary key; you'll use these details when configuring diagnostic settings.

* **Access Diagnostic Settings**  
  * In the Azure Portal, navigate to your Azure Database for PostgreSQL Flexible Server instance.  
  * In the left-hand menu under **Monitoring**, click on **Diagnostic settings**.

* **Enable Diagnostics**  
  * Click **+ Add diagnostic setting**.  
  * Give the setting a name (e.g., "PostgreSQLDiagnostics").

* **Select Log Categories**  
  * Choose the log categories you want to capture (for example, **PostgreSQLLogs**, **QueryStoreRuntimeStatistics**, and **QueryStoreWaitStatistics**).  
  * These logs provide detailed insights into connection issues and overall server performance.

* **Choose a Destination**  
  * Select one or more destinations for your logs:
    * **Log Analytics Workspace**  
      * Select the workspace you just created to enable powerful query and visualization options.
    * **Storage Account** – For archival and manual review.
    * **Event Hub** – For integration with external monitoring solutions.

* **Save and Apply**  
  * Click **Save** to enable diagnostics.  
  * Once enabled, logs will start flowing to your chosen destination(s). Use these logs to troubleshoot issues such as the internal connection error reported by the azure_storage extension.

* **Review the Logs**  
  * Use the tools provided by your destination (for example, the Log Analytics query explorer) to review the logs.  
  * You can use KQL queries—such as filtering for "azure_storage" errors—to gather more details about any issues.

------------------------- -------------------------

#### Enable `azure_storage` Extension

* **Log in to the Azure Portal**  
  Go to [https://portal.azure.com](https://portal.azure.com) and sign in with your credentials.

* **Navigate to Your Server**  
  Open your Azure Database for PostgreSQL Flexible Server instance from the dashboard or resource list.

* **Access Server Parameters**  
  In the left-hand menu, locate and click on **Server Parameters** under the **Settings** section.

* **Locate the Allow List Parameter**  
  Search for the parameter named `azure.extensions.allowlist`. This parameter controls which extensions are allowed.

* **Edit the Parameter**  
  Click on the parameter value to edit it.  
  Check `azure_storage` to add it to the existing list.

* **Save Your Changes**  
  Click **Save** to apply the update.

* **Restart the Server (if required)**  
  Some parameter changes might require a server restart. If prompted or if the changes don't take effect, restart your server using the Azure Portal.

------------------------- -------------------------

### pgAdmin 4

* **Download pgAdmin 4**  
  Visit the [pgAdmin website](https://www.pgadmin.org/download/) and choose the installer for your operating system (Windows, macOS, or Linux).

* **Run the Installer**  
  Follow the installation wizard prompts and choose the default options unless you need a custom installation.

* **Launch pgAdmin 4**  
  Once installed, open pgAdmin 4 and configure the initial setup (e.g., master password) if prompted.

------------------------- -------------------------

#### Register Server

* **Obtain Connection Details**  
  In the Azure portal, go to your new server’s overview and note the **fully qualified domain name (FQDN)** (e.g., `yourserver.postgres.database.azure.com`), the port (typically **5432**), and your admin username.  
  **Important**: Use the username format `yourusername@yourserver`.

* **Open pgAdmin 4 and Create a New Server Registration**  
  In pgAdmin, right-click on **Servers** in the Browser panel and select **Create > Server...**.

* **Fill Out the Connection Information**  
  Under the **General** tab, provide a friendly name for the connection.  
  Under the **Connection** tab, enter the server’s FQDN as the host name/address, set the port to 5432, specify the maintenance database (typically `postgres` or your database name), and use the username in the format `yourusername@yourserver` along with the password you set.  
  In the **SSL** tab, set **SSL Mode** to **require** (or **verify-full** if you are using a certificate).

* **Save and Test the Connection**  
  Click **Save**. pgAdmin should now establish a connection to your Azure Database for PostgreSQL Flexible Server. If you encounter connection issues, double-check the firewall settings and ensure your IP is allowed.

------------------------- -------------------------

Here's a proposed rewrite for the Azure Data Lake Storage section that aligns with managed identity-only access:

---

### Azure Data Lake Storage

* **Create an Azure Storage Account with ADLS Gen2 Enabled**  
  * In the [Azure Portal](https://portal.azure.com), click **Create a resource** and choose **Storage account**.  
  * Fill in the required details (Subscription, Resource Group, and Storage Account Name — e.g., use `{prefix}dl` as your sample storage account name).
  * In the **Advanced** tab, enable **Hierarchical namespace**. This converts your storage account into an Azure Data Lake Storage Gen2 account.  
  * Review and create the storage account.

* **Create a Container for Your Data**  
  * Navigate to your new storage account and open the **Containers** section.  
  * Click **+ Container** to create a new container. Name it appropriately (e.g., `data`).  
  * Set the public access level according to your security needs (for example, a private container with controlled credentials).

* **Upload or Organize Your Data Files**  
  * Upload your data files (e.g., CSV, JSON, or Parquet files) into the container.  
  * Organize your files into folders as needed.

* **Access Configuration**  
  * Since access will be managed through the PostgreSQL server’s system-assigned managed identity, you do not need to use Storage Account Keys or SAS tokens.  
  * Ensure that the appropriate RBAC role (e.g., Storage Blob Data Reader) is assigned to your PostgreSQL server’s managed identity.  
  * Configure any firewall or virtual network rules on the storage account to control access securely.

------------------------- -------------------------

#### Add Container and Sample Blob

* **Create a New Container**  
  * In your Azure Storage account (ADLS Gen2) via the Azure Portal, navigate to **Containers** in the left-hand menu.  
  * Click **+ Container** to add a new container.  
  * Enter a name for your container (for example, `data` or `datalake`) and select the appropriate public access level (for secure use with extensions, choose **Private**).  
  * Click **Create**.

* **Upload a Sample Blob**  
  * Once your container is created, click on the container name to open it.  
  * Click the **Upload** button at the top of the container page.  
  * In the upload pane, click **Browse** (or drag and drop) to select a file from your local system.  
  * For a sample blob, you might create a small CSV file named `mydata.csv` with a few rows of sample data. For example, create a file with the following content:
    ```
    col1,col2,col3
    apple,10,2025-04-01
    banana,20,2025-04-02
    cherry,30,2025-04-03
    ```
  * After selecting the file, click **Upload** to add the blob to your container.

* **Verify the Blob**  
  * Once the upload completes, you should see your new blob (`mydata.csv`) listed in the container.  
  * You can click on the blob to view details such as its URL, which you may use later for testing with the **azure_storage** extension in PostgreSQL.

------------------------- -------------------------

### Managed Identity

Configure your PostgreSQL server to use its system-assigned managed identity for accessing Azure Storage.

#### Enable Managed Identity

* Open your PostgreSQL server in the Azure Portal.
* Under **Settings**, select **Identity**.
* Turn on **System assigned** managed identity and click **Save**.  
  > Note: Enabling managed identity will restart your server.

#### Assign Permissions

* Open your storage account (e.g., `{prefix}dl`) in the Azure Portal.
* Navigate to **Access Control (IAM)**.
* Click **+ Add > Add role assignment**.
* Choose the **Storage Blob Data Reader** role (or **Contributor** if write access is required).
* Under **Assign access to**, select **Managed identity**.
* In **Select members**, search for your PostgreSQL server’s managed identity (by server name) and add it.
* Click **Save**.  
  > Permissions propagation may take a few minutes.

#### Register the Storage Account

* Connect to your PostgreSQL database using psql or pgAdmin.
* Run the following SQL command to register the storage account using managed identity:
  ```sql
  SELECT azure_storage.account_add(
      azure_storage.account_options_managed_identity('{prefix}dl', 'blob')
  );
  ```
  This registers your storage account for access via the managed identity.

* (Optional) Grant your PostgreSQL user access if needed:
  ```sql
  SELECT azure_storage.account_user_add('{prefix}dl', 'your_username');
  ```
  Replace `'your_username'` with your PostgreSQL user name.

#### Test the Connection

* Verify the setup by listing blobs in your container:
  ```sql
  SELECT *
  FROM azure_storage.blob_list(
    '{prefix}dl'::text,  -- Your storage account name
    'data'::text,   -- Your container name
    ''::text        -- No additional credentials required
  );
  ```
  This query should list the blobs in your container using the managed identity's permissions.

------------------------- -------------------------
------------------------- -------------------------

## Data Lake Integration

### ...using pgadmin4

* **Create the Extension**

  In your PostgreSQL database, run:
  ```sql
  CREATE EXTENSION azure_storage;
  ```

* **Confirm the Extension is Installed**

  Verify the extension by running:
  ```sql
  SELECT * FROM pg_available_extensions WHERE name = 'azure_storage';
  ```
  
  You should see output indicating that **azure_storage** is installed with its version and description.

* **Test Connectivity**

  Run the following query to ensure the extension can communicate with the storage endpoint:
  ```sql
  SELECT azure_storage.version();
  ```
  
  This should return the version information (e.g., "Azure Storage 1.5.0 gitref: m43.3"), confirming basic connectivity.

* **List Available Functions**  
  To view all functions provided by the extension, run:

  ```sql
  SELECT n.nspname as schema, p.proname
  FROM pg_proc p
  JOIN pg_namespace n ON p.pronamespace = n.oid
  WHERE n.nspname = 'azure_storage';
  ```
  
  This will list all functions available under the `azure_storage` schema (such as `blob_list`, `blob_get`, etc.).

* **List Blobs in a Container**  
  Run the following query to list all blobs in your Azure Data Lake Storage container using managed identity. Replace `'{prefix}dl'` with your storage account name and `'data'` with your container name:

  ```sql
  SELECT *
  FROM azure_storage.blob_list(
    '{prefix}dl'::text,  -- Your storage account name
    'data'::text,   -- Your container name
    ''::text        -- Managed identity requires no key
  );
  ```
  
  This query returns all blobs present in the specified container.

* **Retrieve Blob Content**  
  To fetch the content of a specific blob (for example, a CSV file), run the following query. Replace `'{prefix}dl'` with your storage account name, `'data'` with your container name, and adjust the file path as needed:
  
  ```sql
  SELECT azure_storage.blob_get(
    '{prefix}dl'::text,                  -- Your storage account name
    'data'::text,                        -- Your container name
    'path/to/mydata.csv'::text,          -- File path within the container
    ''::text                             -- No credential needed with managed identity
  ) AS file_content;
  ```

  This query retrieves the content of the specified file using managed identity.

------------------------- -------------------------

### ...using PowerShell

If you prefer running these commands from Azure Cloud Shell using PowerShell (instead of pgAdmin4), follow these steps:

* **Access Cloud Shell**:  
  * Log in to the [Azure Portal](https://portal.azure.com).  
  * Click the Cloud Shell icon in the top navigation bar.  
  * Choose **PowerShell** as your shell environment.

* **Run the psql Command**: In Cloud Shell, execute a command similar to the following (replace placeholders with your actual details):
  ```powershell
  psql "host={prefix}pg.postgres.database.azure.com port=5432 dbname=postgres user=<username> password='<password>' sslmode=require"
  ```
  > **Note**: If your password or username includes special characters, you may need to URL encode them or use environment variables.

* **Test the azure_storage Extension**:  
  Once connected, run:
  ```sql
  SELECT azure_storage.version();
  ```
  This confirms the extension is working.

* **Use Managed Identity Commands**: To register your storage account with managed identity, run:
  ```sql
  SELECT azure_storage.account_add( azure_storage.account_options_managed_identity('{prefix}dl', 'blob') );
  ```
* **Query Blobs Using Managed Identity**: To list blobs from your container, run:
  ```sql
  SELECT * FROM azure_storage.blob_list(
    '{prefix}dl'::text,
    'data'::text,
    ''::text
  );
  ```

------------------------- -------------------------

### Reference

> [Import and export data using Azure Storage in Azure Database for PostgreSQL flexible server
](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/concepts-storage-extension)]

> [Import and export data using azure_storage extension in Azure Database for PostgreSQL flexible server](https://learn.microsoft.com/en-us/azure/postgresql/flexible-server/how-to-use-pg-azure-storage)
