# Python

This guide walks you through ensuring pip is installed (and updated) and setting up a Python virtual environment.

First, verify that Python is installed. Open your terminal and run:

```powershell
python --version
```

If Python is not installed, simply run:

```powershell
python
```

This will trigger the Microsoft Store installation (if that’s acceptable for your use case).

Next, check if pip is installed by running:

```powershell
pip --version
```

If pip is not recognized, follow these steps:

Download the installer:

```powershell
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
```

Run the installer with Python:

```powershell
python get-pip.py
```

After installation, verify pip is available:

```powershell
pip --version
```

If you see a warning that pip’s scripts are installed in a directory not on your PATH (for example, a message like:

```
WARNING: The scripts pip.exe, pip3.13.exe and pip3.exe are installed in 'C:\Users\admin\...\Scripts' which is not on PATH.
```

), update pip and add pip to your PATH:

Update pip by running:

```powershell
python -m pip install --upgrade pip
```

Add pip to your PATH using PowerShell (adjust the directory as needed):

```powershell
$PipPath = "C:\Users\admin\AppData\Local\Packages\PythonSoftwareFoundation.Python.3.13_qbz5n2kfra8p0\LocalCache\local-packages\Python313\Scripts"
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$PipPath", [EnvironmentVariableTarget]::User)
```

Then restart your PowerShell session for the changes to take effect.

To set up a Python virtual environment, create a project directory:

```powershell
mkdir "$HOME\Documents\VirtualEnvironment"
cd "$HOME\Documents\VirtualEnvironment"
```

Create the virtual environment:

```powershell
python -m venv venv
```

Activate the virtual environment:

```powershell
.\venv\Scripts\Activate
```

Your prompt should now display `(venv)`, indicating the environment is active.
