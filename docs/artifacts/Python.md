## Python

### Already installed?

Open PowerShell as an administrator and run the following command to check the installed version of Python:
```powershell
python --version
```

Expected output (version may vary):
```
Python 3.13.1
```

### If not already installed...

Download and install the latest version from the [python.org](https://www.python.org/downloads)

<img src="https://github.com/user-attachments/assets/1f103580-e5a3-46fc-82c7-2c91e700e75c" width="800" title="Snipped February 3, 2025" />

_During installation, be sure to check the "Add ... to PATH" box_

Execute the following PowerShell command to confirm Python has been added to `PATH`:
```powershell
where.exe python
```

- If the output displays a path, Python is installed
- If an error is returned restart the PowerShell window (or machine), then re-verify
