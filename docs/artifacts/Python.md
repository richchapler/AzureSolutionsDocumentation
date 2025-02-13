## Python

### Already Installed?

1. Open PowerShell as an administrator and run the following command to check the installed version of Python:

   ```powershell
   python --version
   ```

   - If the output displays a version, Python is installed
   - If an error is returned or a different version is shown, proceed to install or update Python

### If not, install...

1. Download and install the latest version from the [official website](https://www.python.org/downloads)

   **Important:** Check the box to "Add Python 3.9 to PATH" during installation.

2. Execute the following PowerShell command to confirm Python has been added to `PATH`:

   ```powershell
   where.exe python
   ```

   - If the output displays a path, Python is installed
   - If an error is returned restart the PowerShell window (or machine), then verify again with `where.exe python`.
