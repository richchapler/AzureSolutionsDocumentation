# Python

## Installed?

Open PowerShell as an administrator and run the following command to check the installed version of Python:

```powershell
python --version
```

Expected output (version may vary):

```
Python 3.13.1
```

## Not installed...

Execute the following PowerShell command to search for available Python versions:

```powershell
winget search Python.Python
```

Identify the latest available version from the results, then install it:

```powershell
winget install --id Python.Python.<major_version> --source winget -e
```

*Replace `<major_version>` with the latest available major version number (e.g., `3.13`). Do not include minor or patch versions.*

### Disable the Microsoft Store Alias for Python
If Python is still not recognized, disable the Microsoft Store alias:

1. Open **Settings** > **Apps** > **Installed Apps**
2. Search for **App Execution Aliases**
3. Locate **python.exe** and **python3.exe**, then disable them
4. Restart your computer and check Python again:
   
   ```powershell
   where.exe python
   python --version
   ```

### Ensure Python is in PATH
If Python still does not work, refresh the environment variables in the current PowerShell session:

```powershell
$env:Path = [System.Environment]::GetEnvironmentVariable("Path", [System.EnvironmentVariableTarget]::Machine)
```

Then, verify again:

```powershell
where.exe python
python --version
```

## (Optional) Set Up a Python Virtual Environment

### Already Set Up?

Open PowerShell as an administrator and execute the following command to check if a virtual environment is active:

```powershell
if ($env:VIRTUAL_ENV) { "Virtual environment is active: $env:VIRTUAL_ENV" } else { "No virtual environment detected" }
```

Expected output if no virtual environment is active:

```
No virtual environment detected
```

### If Not...

Execute the following command to create and then navigate to a project directory:

```powershell
mkdir "$HOME\Documents\VirtualEnvironment"
cd "$HOME\Documents\VirtualEnvironment"
```

Execute the following command to create a virtual environment:

```powershell
& (where.exe python | Select-String "WindowsApps" -NotMatch | Select-Object -First 1) -m venv venv
```

Execute the following command to activate the virtual environment:

```powershell
venv\Scripts\Activate
```

After running this command, note that your prompt changes to include prefix `(venv)`, confirming that the virtual environment is active.

