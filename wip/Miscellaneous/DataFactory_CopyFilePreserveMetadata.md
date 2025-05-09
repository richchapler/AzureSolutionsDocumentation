# Data Factory: Preserve Metadata

## "Preserve Metadata"

<!-- ------------------------- ------------------------- -->

### Use Case

- from SFTP to copy files to storage, preserve metadata, and surface it as a network drive

<!-- ------------------------- ------------------------- -->

### Proposed Solution

- Lorem

<!-- ------------------------- ------------------------- -->

### Required Resources

- **Resource Group** `<prefix>`

- **Data Factory** `<prefix>df`

- **Storage Account** `<prefix>sftp`
  <br>(configured to act as SFTP source)
  - Redundancy: **Locally-redundant storage (LRS)**
  - Hierarchical Namespace: **ENABLED**
  - SFTP: **ENABLED**

-------------------------

### Walk-Through

<!-- ------------------------- ------------------------- -->

#### Storage Account: Prepare SFTP

Navigate to **Storage Account** >> **Settings** >> **SFTP**, click **+ Add local user**, and complete the resulting **Add local user** form:

Setting | Value
:--- | :---
Username | `sftpuser`
Container | `data` (private) with Permissions: Read, Write, List, Delete

Click **Add**.

Back on the **SFTP** page, click to configure:

- **Authentication method**: On the **Edit local user** popout, **Username + Authentication** tab, check **SSH Password**, then click **Next** .. **Save**, and capture the provided password.
- **Home (landing) directory**: On the **Edit local user** popout, **Permissions** tab, enter "Home (landing) directory": `data`, then click **Save**

<!-- ------------------------- -->

##### Install Posh-SSH

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

##### Import Posh-SSH

Execute the following command to:
- Load the Posh-SSH library into the current PowerShell session
- Make cmdlets (e.g., `New-SFTPSession`, `Set-SFTPItem`, `Get-SFTPChildItem`, etc.) available for use without path qualification

```powershell
Import-Module Posh-SSH
```

**No expected output**

<!-- ------------------------- -->

##### Prepare Session

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

##### Upload Sample

Click **Manage files** >> **Upload**, select a sample file, and then click **Open**.

```powershell
Set-SFTPItem -SFTPSession $session -Path "/home/mod/sample.txt" -Destination "/" -Force
```

**No expected output**

<!-- ------------------------- -->

##### Confirm Success

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

<!-- ------------------------- ------------------------- -->

#### Data Factory: Create Pipeline

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

Click **Test Connection** and confirm successful connection.

<!-- ------------------------- -->
<!-- ------------------------- -->
<!-- ------------------------- -->
<!-- ------------------------- -->
<!-- ------------------------- -->
<!-- ------------------------- -->
<!-- ------------------------- -->
<!-- ------------------------- -->

RESUME HERE!

- In the Azure portal navigate to your Data Factory resource and click “Author & Monitor”

- In Data Factory Studio click “Author”

- Click the down-arrow next to “Pipelines” and select “New pipeline”

- In the Properties pane set Name to CopyFromSftp

- From the Activities pane expand “Move & Transform” and drag “Copy data” onto the canvas

- Configure the source

  - Click the copy activity, select the “Source” tab
  - Under “Source dataset” choose your SFTP dataset
  - If you need subfolders select “Recursion”

- Configure the sink

  - Select the “Sink” tab
  - Under “Sink dataset” choose your Azure Storage dataset (pointing at container data)
  - In “Settings” set Copy behavior to Preserve hierarchy
  - Under “Additional settings” enable “Preserve last modified time”

- Validate and test

  - Click “Validate” at the top to check for errors
  - Click “Debug” to run the pipeline immediately with default parameters
  - Monitor the run in the “Output” pane to confirm files copied and metadata preserved

- Publish changes

  - When testing succeeds click “Publish all” in the top toolbar to deploy your pipeline to production
