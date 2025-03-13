# Python

This guide walks you through verifying your Python installation using winget, listing available Python versions, installing/upgrading Python, disabling the Microsoft Store alias, and setting up a Python virtual environment.

---

## 1. Check Available Python Versions

Before installing or upgrading, you can see which major versions are available from winget:

1. Open **PowerShell** as an administrator.
2. Run:
   ```powershell
   winget search Python.Python
   ```
3. Review the results. You might see entries like:
   ```
   Name           Id                      Version   Source
   ---------------------------------------------------------
   Python 3.13    Python.Python.3.13      3.13.1    winget
   Python 3.14    Python.Python.3.14      3.14.0    winget
   ```
4. Determine the latest major version from the list (for example, if **Python 3.14** is listed, that's the most current major version).

---

## 2. Verify Python Installation

### 2.1 Check the Installed Version via Winget

1. Once you've determined the appropriate major version (for example, 3.14), run:
   ```powershell
   winget list --id Python.Python.3.14
   ```
   - If Python is installed, you’ll see output similar to:
     ```
     Failed when searching source: msstore
     Name         Id                     Version Source
     ---------------------------------------------------------
     Python 3.14  Python.Python.3.14     3.14.0   winget
     ```
   - The **Version** column shows your currently installed version.
2. To check if an upgrade is available within the same major version, run:
   ```powershell
   winget upgrade --id Python.Python.3.14
   ```
   - This command will list patch-level updates (for example, from 3.14.0 to 3.14.1).  
   - **Note:** Upgrades here update patch versions. When a new major version is released, it will typically use a new package ID (e.g., `Python.Python.3.15`).

---

## 3. Install or Upgrade Python

1. **Search for Available Python Versions** (if you haven’t done so already):
   ```powershell
   winget search Python.Python
   ```
   Identify the latest **major version** (for example, `3.14`).

2. **Install or Upgrade**  
   For example, to install or upgrade to Python 3.14, run:
   ```powershell
   winget install --id Python.Python.3.14 --source winget -e
   ```
   - Replace `3.14` with the major version you wish to install.
   - After installation, close and reopen PowerShell.

3. **Verify Installation**  
   Run:
   ```powershell
   python --version
   ```
   Confirm it shows the expected version (e.g., `Python 3.14.x`).

---

## 4. Disable the Microsoft Store Alias for Python

If running `python --version` returns a prompt to install Python from the Microsoft Store or if:
```powershell
where.exe python
```
shows a path like:
```
C:\Users\<User>\AppData\Local\Microsoft\WindowsApps\python.exe
```
then the Microsoft Store alias is overriding your installed version.

### 4.1 Finding App Execution Aliases in Windows 11

1. **Open Settings:**  
   Click **Start** > **Settings** (or press **Win + I**).

2. **Go to Apps:**  
   In the left navigation, select **Apps**.

3. **Open Advanced App Settings:**  
   On the main Apps page, click **Advanced app settings**.

4. **Select App Execution Aliases:**  
   You should see toggles for **python.exe** and **python3.exe**.

5. **Disable Python Aliases:**  
   Toggle both **python.exe** and **python3.exe** to **Off**.

6. **Restart Your Computer:**  
   Restart to ensure changes take effect.

7. **Verify:**  
   Open a new PowerShell window and run:
   ```powershell
   where.exe python
   python --version
   ```
   You should now see the path to your installed Python (e.g.,  
   `C:\Users\<User>\AppData\Local\Programs\Python\Python3.14\python.exe`) and the correct version.

### 4.2 If You Cannot Find App Execution Aliases

- **Use the Settings Search:** Type **App execution aliases** in the Settings search bar.
- **Check Your Windows Version:** Go to **Settings** > **System** > **About** to verify your build.
- **Consult Microsoft Documentation:** If the option isn’t available, your system may manage aliases via Group Policy or registry settings.

---

## 5. Set Up a Python Virtual Environment

Virtual environments help manage project-specific dependencies.

### 5.1 Check if a Virtual Environment Is Active

1. Open **PowerShell**.
2. Run:
   ```powershell
   if ($env:VIRTUAL_ENV) { "Virtual environment is active: $env:VIRTUAL_ENV" } else { "No virtual environment detected" }
   ```
   - If the output reads `"No virtual environment detected"`, then no virtual environment is active.

### 5.2 Create and Activate a Virtual Environment

1. **Create a Project Directory:**
   ```powershell
   mkdir "$HOME\Documents\VirtualEnvironment"
   cd "$HOME\Documents\VirtualEnvironment"
   ```
2. **Create the Virtual Environment:**  
   Use the following command to create a virtual environment named `venv` (which filters out any alias paths):
   ```powershell
   & (where.exe python | Select-String "WindowsApps" -NotMatch | Select-Object -First 1) -m venv venv
   ```
3. **Activate the Virtual Environment:**
   ```powershell
   .\venv\Scripts\Activate
   ```
   When activated, your prompt will show `(venv)`, indicating the virtual environment is active.
