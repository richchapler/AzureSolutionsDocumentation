# Python Virtual Environment

## Already Set Up?

Open PowerShell as an administrator and execute the following command to check if a virtual environment is active:

```powershell
if ($env:VIRTUAL_ENV) { "Virtual environment is active: $env:VIRTUAL_ENV" } else { "No virtual environment detected" }
```

Expected output if no virtual environment is active:
```
No virtual environment detected
```

## If Not...

Execute the following command to create and then navigate to a project directory:

```powershell
mkdir "$HOME\Documents\VirtualEnvironment"
cd "$HOME\Documents\VirtualEnvironment"
```

Execute the following command to create a virtual environment:

```powershell
python -m venv venv
```

Execute the following command to activate the virtual environment:

```powershell
venv\Scripts\Activate
```

After running this command, note that your prompt changes to include prefix `(venv)`, confirming that the virtual environment is active.

