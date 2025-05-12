# Data Factory: Preserve Metadata

<!-- ------------------------- ------------------------- -->

## Use Case

- User wants to copy files from SFTP to a network drive using Data Factory
- Metadata (e.g., `Last Modified`) should be preserved on the file surfaced to the network drive
- There is no built-in solution for Data Factory to preserve metadata for this configuration... it is only supported when copying between binary/file-based stores such as Amazon S3, Azure Blob Storage, Azure Data Lake Storage Gen2, and Azure Files. It is possible to preserve metadata in those scenarios. For example, the Microsoft article “Preserve metadata and ACLs using copy activity in Azure Data Factory or Synapse Analytics” ([https://learn.microsoft.com/en-us/azure/data-factory/copy-activity-preserve-metadata](https://learn.microsoft.com/en-us/azure/data-factory/copy-activity-preserve-metadata)) states: “When you migrate data from one data lake to another including Amazon S3, Azure Blob, Azure Data Lake Storage Gen2, and Azure Files, you can choose to preserve the file metadata along with data.”

<!-- ------------------------- ------------------------- -->

## Short Answer

The short answer is that **Azure Data Factory (ADF) does not offer built‐in support to preserve file metadata like “Last Modified” when copying files from an SFTP source to a local network drive sink**. While ADF’s SFTP connector is capable of reading file properties (including the last modified timestamp) via activities such as Get Metadata, the standard Copy Activity does not provide a mechanism to write or set those metadata properties on the sink file. When files are written to a local file system (or network drive), the operating system assigns new timestamps based on when the files were created or modified on that system.

For example, the [SFTP connector documentation](https://learn.microsoft.com/en-us/azure/data-factory/connector-sftp) explains that it supports reading properties like file size and last modified time from the source; however, there is no corresponding option in the File System connector (used for local/network drives) to apply these metadata settings on the destination file. The [File System connector documentation](https://learn.microsoft.com/en-us/azure/data-factory/connector-file-system) indicates that attributes such as the file’s creation time and modification time are determined by the sink’s operating system at the time of file creation, meaning that any source metadata (like the original “Last Modified” time) won’t be carried over automatically.

If preserving the original “Last Modified” timestamp is critical for your use case, you would need to consider a custom or post-processing solution. One common approach is to use a Get Metadata activity to capture the file’s properties from the SFTP source, then execute an Azure Function (or custom activity) after the copy operation to update the file attributes on the destination file accordingly.

Would you be interested in exploring these custom approaches or additional alternatives (such as storing the metadata alongside the file data) to better manage file properties?

<!-- ------------------------- ------------------------- -->

## Proposed Solution

- Lorem

<!-- ------------------------- ------------------------- -->

## Required Resources

- **Resource Group** `<prefix>`

- **Data Factory** `<prefix>df`

- **Storage Account (source)** `<prefix>sftp`
  <br>(configured to act as an SFTP source)
  - Redundancy: **Locally-redundant storage (LRS)**
  - Hierarchical Namespace: **ENABLED**
  - SFTP: **ENABLED**

- **Storage Account (sink)** `<prefix>nd`
  <br>(configured to act as a network drive sink)
  - Redundancy: **Locally-redundant storage (LRS)**

- [**Storage Explorer**](https://azure.microsoft.com/en-us/products/storage/storage-explorer/?msockid=1aaed3c9c37c631b3702c659c25162d5)

------------------------- ------------------------- ------------------------- -------------------------

## Walk-Through

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

### Storage Account: `{prefix}sftp`

Navigate to **Storage Account** `{prefix}sftp` >> **Settings** >> **SFTP**, click **+ Add local user**, and complete the resulting **Add local user** form:

Setting | Value
:--- | :---
Username | `sftpuser`
Container | `data` (private) with Permissions: Read, Write, List, Delete

Click **Add**.

Back on the **SFTP** page, click to configure:

- **Authentication method**: On the **Edit local user** popout, **Username + Authentication** tab, check **SSH Password**, then click **Next** .. **Save**, and capture the provided password.
- **Home (landing) directory**: On the **Edit local user** popout, **Permissions** tab, enter "Home (landing) directory": `data`, then click **Save**

<!-- ------------------------- -->

#### Install Posh-SSH

Navigate to the **Cloud Shell**, configure as required, and select **Powershell**.

Execute the following command to:
- Contact the PowerShell Gallery and downloads the Posh-SSH module
- Install the module under user profile (no admin rights required)
- Overwrite any existing version and suppresses prompts about dependencies or untrusted sources

```powershell
Install-Module -Name Posh-SSH -Scope CurrentUser -Force
```

**No expected output**

<!-- ------------------------- -->

#### Import Posh-SSH

Execute the following command to:
- Load the Posh-SSH library into the current PowerShell session
- Make cmdlets (e.g., `New-SFTPSession`, `Set-SFTPItem`, `Get-SFTPChildItem`, etc.) available for use without path qualification

```powershell
Import-Module Posh-SSH
```

**No expected output**

<!-- ------------------------- -->

#### Prepare Session

```powershell
$password = ConvertTo-SecureString "<sftpPassword>" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential("<prefix>sftp.sftpuser", $password)
$session = New-SFTPSession -ComputerName "<prefix>sftp.blob.core.windows.net" -Credential $credential -Port 22
```

**Expected Output**
```plaintext
Server SSH Fingerprint
Do you want to trust the fingerprint <redacted>
[] Y  [] N  [?] Help (default is "N"): Y
```

<!-- ------------------------- -->

#### Upload Sample

Click **Manage files** >> **Upload**, select a sample file, and then click **Open**.

```powershell
Set-SFTPItem -SFTPSession $session -Path "/home/mod/sample.txt" -Destination "/" -Force
```

**No expected output**

<!-- ------------------------- -->

#### Confirm Success

Use the following PowerShell to confirm success:

```powershell
Get-SFTPChildItem -SFTPSession $session -Path "/"
```

**Expected Output**
```plaintext
FullName       : /sample.txt
LastAccessTime : 1/1/1970 12:00:00 AM
LastWriteTime  : 5/9/2025 2:43:56 PM
Length         : 11
UserId         : 0
```

Or confirm success by navigating to **Storage Account** >> **Storage browser** >> **Blob containers** >> `data` container.

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

### Storage Account: `{prefix}nd`

Navigate to **Storage Account** >> **Data storage** >> **File shares**, click **+ File share**, and complete the resulting **New file share** form:

Setting | Value
:--- | :---
Name | `data`
Access tier | `Transaction optimized`

Click **Review + create** >> **Create**.

<!-- ------------------------- ------------------------- ------------------------- ------------------------- -->

### Data Factory: What Works and What Doesn't

<!-- ------------------------- ------------------------- -->

#### Source

##### Linked Service

Navigate to **Data Factory Studio** >> **Manage** >> **Connections** >> **Linked services** and click **+ New**.

On the **New linked service** popout, search for and select **SFTP**, then complete the resulting form:

Setting | Value
:--- | :---
Name | `<prefix>sftp`
Host | `<prefix>sftp.blob.core.windows.net`
Port | `22`
SSH host key validation | `Disable SSH host key validation`
Authentication type | `Basic`
User name | `<prefix>sftp.sftpuser`
Password | <sftpPassword>

Click **Test Connection** and confirm successful connection, then click **Create**.

<!-- ------------------------- -->

##### Dataset

Navigate to **Data Factory Studio** >> **Author** and click **+** >> **Dataset**.

On the **New dataset** popout, search for and select **SFTP**, then click **Continue**.

On the **Select format** popout, select **Binary** and then click **Continue**.

Complete the **Set properties** form:

Setting | Value
:--- | :---
Name | `<prefix>sftp`
Linked service | `<prefix>sftp`

Click **OK**.

<!-- ------------------------- ------------------------- -->

#### Sink

##### Linked Service

Navigate to **Data Factory Studio** >> **Manage** >> **Connections** >> **Linked services** and click **+ New**.

On the **New linked service** popout, search for and select **File system**, then complete the resulting form:

Setting | Value
:--- | :---
Name | `<prefix>nd`
Host | `\\<prefix>nd.file.core.windows.net\data`
User name | `<prefix>nd`
Password | `<storageAccount_key1>`

Click **Test Connection** and confirm successful connection, then click **Create**.

<!-- ------------------------- -->

##### Dataset

Navigate to **Data Factory Studio** >> **Author** and click **+** >> **Dataset**.

On the **New dataset** popout, search for and select **File system**, then click **Continue**.

On the **Select format** popout, select **Binary** and then click **Continue**.

Complete the **Set properties** form:

Setting | Value
:--- | :---
Name | `<prefix>nd`
Linked service | `<prefix>nd`

Click **OK**.

<!-- ------------------------- ------------------------- -->

#### Pipeline

Navigate to **Data Factory Studio** >> **Author** and click **+** >> **Pipeline** >> **Pipeline**.

Drag-and-drop a **Copy data** activity to the canvas.

On the **Source** tab, select **Source dataset** `{prefix}sftp`.

On the **Sink** tab, select **Sink dataset** `{prefix}nd` and set **Copy behavior** `Preserve hierarchy`.

Click **Debug** and confirm successful copy of the `sample.txt` to `{prefix}nd`.


------------------------------------------------------------------------------------------

# Reference

- [Preserve metadata and ACLs using copy activity in Azure Data Factory or Synapse Analytics](https://learn.microsoft.com/en-us/azure/data-factory/copy-activity-preserve-metadata?utm_source=chatgpt.com)