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

Download and install the latest version from the [python.org](https://www.python.org/downloads)

<img src="https://github.com/user-attachments/assets/1f103580-e5a3-46fc-82c7-2c91e700e75c" width="800" title="Snipped February 3, 2025" />

_During installation, be sure to check the "Add ... to PATH" box_

Execute the following PowerShell command to confirm Python has been added to `PATH`:

```powershell
where.exe python
```

- If the output displays a path, Python is installed
- If an error is returned restart the PowerShell window (or machine), then re-verify

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
& (where.exe python) -m venv venv
```

Execute the following command to activate the virtual environment:

```powershell
venv\Scripts\Activate
```

After running this command, note that your prompt changes to include prefix `(venv)`, confirming that the virtual environment is active.

