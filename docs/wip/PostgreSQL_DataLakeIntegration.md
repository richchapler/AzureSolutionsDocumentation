# PostgreSQL: Data Lake Integration

## Prepare Resources

------------------------- -------------------------

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
  - In the Azure Portal, click **Create a resource** and search for **Log Analytics workspace**.  
  - Click **Create** and fill in the required details such as Workspace name, Subscription, Resource Group, and Region.  
  - Click **Review + create**, then **Create** to instantiate your new Log Analytics workspace.

* **Note the Workspace Details**  
  - Once created, note the Workspace ID and primary key; you'll use these details when configuring diagnostic settings.

* **Access Diagnostic Settings**  
  - In the Azure Portal, navigate to your Azure Database for PostgreSQL Flexible Server instance.  
  - In the left-hand menu under **Monitoring**, click on **Diagnostic settings**.

* **Enable Diagnostics**  
  - Click **+ Add diagnostic setting**.  
  - Give the setting a name (e.g., "PostgreSQLDiagnostics").

* **Select Log Categories**  
  - Choose the log categories you want to capture (for example, **PostgreSQLLogs**, **QueryStoreRuntimeStatistics**, and **QueryStoreWaitStatistics**).  
  - These logs provide detailed insights into connection issues and overall server performance.

* **Choose a Destination**  
  - Select one or more destinations for your logs:
    - **Log Analytics Workspace**  
      - Select the workspace you just created to enable powerful query and visualization options.
    - **Storage Account** – For archival and manual review.
    - **Event Hub** – For integration with external monitoring solutions.

* **Save and Apply**  
  - Click **Save** to enable diagnostics.  
  - Once enabled, logs will start flowing to your chosen destination(s). Use these logs to troubleshoot issues such as the internal connection error reported by the azure_storage extension.

* **Review the Logs**  
  - Use the tools provided by your destination (for example, the Log Analytics query explorer) to review the logs.  
  - You can use KQL queries—such as filtering for "azure_storage" errors—to gather more details about any issues.

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
  **Important:** Use the username format `yourusername@yourserver`.

* **Open pgAdmin 4 and Create a New Server Registration**  
  In pgAdmin, right-click on **Servers** in the Browser panel and select **Create > Server...**.

* **Fill Out the Connection Information**  
  Under the **General** tab, provide a friendly name for the connection.  
  Under the **Connection** tab, enter the server’s FQDN as the host name/address, set the port to 5432, specify the maintenance database (typically `postgres` or your database name), and use the username in the format `yourusername@yourserver` along with the password you set.  
  In the **SSL** tab, set **SSL Mode** to **require** (or **verify-full** if you are using a certificate).

* **Save and Test the Connection**  
  Click **Save**. pgAdmin should now establish a connection to your Azure Database for PostgreSQL Flexible Server. If you encounter connection issues, double-check the firewall settings and ensure your IP is allowed.

------------------------- -------------------------

### Azure Data Lake Storage

* **Create an Azure Storage Account with ADLS Gen2 Enabled**  
  * In the [Azure Portal](https://portal.azure.com), click **Create a resource** and choose **Storage account**.  
  * Fill in the required details (Subscription, Resource Group, and Storage Account Name).  
  * In the **Advanced** tab, enable **Hierarchical namespace**. This converts your storage account into an Azure Data Lake Storage Gen2 account.  
  * Review and create the storage account.

* **Create a Container for Your Data**  
  * Navigate to your new storage account and open the **Containers** section.  
  * Click **+ Container** to create a new container. Name it appropriately (e.g., `datalake`).  
  * Set the public access level according to your security needs (for example, a private container with controlled credentials).

* **Upload or Organize Your Data Files**  
  * Upload your data files (e.g., CSV, JSON, or Parquet files) into the container.  
  * Organize your files into folders as needed.

* **Access Configuration**  
  * Ensure you have the necessary credentials to allow access from PostgreSQL. Typically, this means keeping your Storage Account Key or a Shared Access Signature (SAS) available.  
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

#### Shared Access Signature

* **Log in to the Azure Portal**  
  Open the [Azure Portal](https://portal.azure.com) and navigate to your Azure Storage account that is configured with ADLS Gen2.

* **Access the Shared Access Signature (SAS) Settings**  
  In the storage account's menu, under **Settings**, click on **Shared access signature**.

* **Configure the SAS Options**  
  * **Allowed Services:** Select the services you need (typically **Blob** for ADLS).  
  * **Allowed Resource Types:** Choose the resource types you require (e.g., **Service**, **Container**, **Object**).  
  * **Allowed Permissions:** Enable the necessary permissions (for example, **Read** to list blobs and retrieve file content).  
  * **Start and Expiry Date/Time:** Specify the start time (usually the current time) and set an expiry time that fits your use case.  
  * **Allowed Protocols:** Choose **HTTPS only** for secure access.

* **Generate the SAS Token**  
  Click **Generate SAS and connection string**.  
  The portal will display your SAS token. Copy the token (or the token portion after the `?`), as you'll need it for your queries in PostgreSQL.

* **Use the SAS Token in Your Queries**  
  In your PostgreSQL queries that use the **azure_storage** extension, replace the `<your_account_key>` placeholder with your SAS token if you prefer using SAS-based authentication. For example:
  ```sql
  SELECT *
  FROM azure_storage.blob_list(
    'usbdl',               -- Your storage account name
    'data',                -- Your container name
    '<your_SAS_token>'     -- Your SAS token (without the leading '?')
  );
  ```

------------------------- -------------------------

## Using pgadmin4

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

* **List Blobs in a Container (Using Access Key)**  
  Use the provided function to list blobs in your Azure Data Lake Storage container. Replace the placeholders with your actual values (using explicit casts to ensure the proper types):
  ```sql
  SELECT *
  FROM azure_storage.blob_list(
    'usbdl'::text,              -- Your storage account name
    'data'::text,               -- Your container name
    '<your_access_key>'::text    -- Your storage account access key
  );
  ```
  This query returns a list of blobs in the specified container.

* **Retrieve Blob Content**  
  To retrieve the content of a specific blob (for example, a CSV file), run:
  ```sql
  SELECT azure_storage.blob_get(
    'usbdl'::text,                  -- Your storage account name
    'data'::text,                   -- Your container name
    'path/to/mydata.csv'::text,     -- The path to your file within the container
    '<your_SAS_token>'::text         -- Your SAS token (or storage account key)
  ) AS file_content;
  ```
  This query returns the content of the specified file, which you can then process as needed.

* **Integrate the Data**  
  Once you have the blob content, consider loading it into a temporary table or processing it directly within PostgreSQL for further analysis.

------------------------- -------------------------

## Using Managed Identity Instead of Access Keys

To avoid using access keys, you can configure your PostgreSQL server to use its system-assigned managed identity with Azure Storage.

### 1. Enable System-Assigned Managed Identity for Your PostgreSQL Server

* **Open Your PostgreSQL Server in the Azure Portal:**  
  Navigate to your Azure Database for PostgreSQL Flexible Server instance.

* **Enable Managed Identity:**  
  - In the left-hand menu, click on **Identity** under the **Settings** section.  
  - Turn on the **System assigned** managed identity and click **Save**.  
  > Enabling the managed identity will restart your server.

### 2. Grant the Managed Identity Permissions on Your Storage Account

* **Open Your Storage Account in the Azure Portal:**  
  Navigate to your storage account (e.g., `usbdl`).

* **Assign RBAC Role:**  
  - Go to the **Access Control (IAM)** section.  
  - Click **+ Add** > **Add role assignment**.
  - Select the **Storage Blob Data Reader** role (or **Contributor** if you also need write access).
  - Under **Assign access to**, choose **Managed identity**.
  - In **Select members**, search for your PostgreSQL server’s managed identity (by server name) and add it.
  - Click **Save**.  
  > It may take a few minutes for permissions to propagate.

### 3. Add the Storage Account Reference Using Managed Identity

* **Open psql or pgAdmin:**  
  Connect to your PostgreSQL database.

* **Register the Storage Account for Managed Identity Access:**  
  Run the following SQL command to add a reference using the managed identity:
  ```sql
  SELECT azure_storage.account_add(
      azure_storage.account_options_managed_identity('usbdl', 'blob')
  );
  ```
  This command registers your storage account to be accessed using the system-assigned managed identity.

* **Grant Access to Your User (if necessary):**  
  If needed, grant your user permission to use the storage account reference:
  ```sql
  SELECT azure_storage.account_user_add('usbdl', 'your_username');
  ```
  Replace `'your_username'` with your PostgreSQL user name.

### 4. Test the Managed Identity Connection

Now, list blobs using managed identity (note: you do not pass an access key or SAS token):
```sql
SELECT *
FROM azure_storage.blob_list(
  'usbdl'::text,  -- Your storage account name
  'data'::text,   -- Your container name
  ''::text        -- Empty prefix (no key needed with managed identity)
);
```

This query should now list blobs using the permissions granted via the managed identity.

------------------------- -------------------------

## Using Azure Cloud Shell with PowerShell

If you prefer running these commands from Azure Cloud Shell using PowerShell (instead of pgAdmin4), follow these steps:

### 1. Open Azure Cloud Shell in PowerShell Mode

* **Access Cloud Shell:**  
  - Log in to the [Azure Portal](https://portal.azure.com).  
  - Click the Cloud Shell icon in the top navigation bar.  
  - Choose **PowerShell** as your shell environment.

### 2. Connect to Your PostgreSQL Database Using psql

* **Run the psql Command:**  
  In Cloud Shell, execute a command similar to the following (replace placeholders with your actual details):
  ```powershell
  & psql "host=ubspg.postgres.database.azure.com port=5432 dbname=postgres user=rchapler password='your_password' sslmode=require"
  ```
  > **Note:** If your password or username includes special characters, you may need to URL encode them or use environment variables.

### 3. Execute Azure Storage Commands from psql

* **Test the azure_storage Extension:**  
  Once connected, run:
  ```sql
  SELECT azure_storage.version();
  ```
  This confirms the extension is working.

* **Use Managed Identity Commands:**  
  To register your storage account with managed identity, run:
  ```sql
  SELECT azure_storage.account_add(
      azure_storage.account_options_managed_identity('usbdl', 'blob')
  );
  ```
* **Query Blobs Using Managed Identity:**  
  To list blobs from your container:
  ```sql
  SELECT *
  FROM azure_storage.blob_list(
    'usbdl'::text,
    'data'::text,
    ''::text
  );
  ```
  This command should list blobs using the managed identity without needing an access key.

### 4. Review and Troubleshoot

* **Use psql's Interactive Commands:**  
  In the Cloud Shell psql session, you can run any of your queries interactively.  
* **Leverage Cloud Shell Logging:**  
  Since Cloud Shell runs in a managed environment, any output or error messages will be visible in the terminal, aiding in troubleshooting.
