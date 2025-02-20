# PowerShell

## Up-to-Date?

Open PowerShell as an administrator and run the following command to check the installed version:

```powershell
$PSVersionTable.PSVersion
```

Expected output (version may vary):
```
Major  Minor  Build  Revision
-----  -----  -----  --------
5      1      26100  2161
```

## If not...

If the major version is lower than 7, upgrading PowerShell is recommended.

Execute the following PowerShell command to install PowerShell:

```powershell
winget install --id Microsoft.Powershell --source winget
```

_After upgrade, restart your PowerShell window to apply changes._

Execute the following PowerShell command to verify the upgrade:

```powershell
$PSVersionTable.PSVersion
```

- If the output reflects the latest version, PowerShell is up to date
- If an older version is still displayed, restart your machine and check again

------------------------- ------------------------- ------------------------- -------------------------

## Troubleshooting

### Check if PowerShell is Installed
Run:
```powershell
where.exe pwsh
```
- If a path is returned, PowerShell is installed but not set as the default.

### Manually Launch PowerShell
Try running:
```powershell
pwsh
```
This should start PowerShell. Then, re-run:
```powershell
$PSVersionTable.PSVersion
```

### Ensure PowerShell is the Default Version
If `pwsh` works but `powershell` still returns an older version:

#### Set PowerShell as Default in Windows Terminal
1. Open **Windows Terminal** (`wt` in Run or Search).
2. Click the **dropdown arrow** next to the new tab button (`+`) and select **Settings**.
3. Under **Default profile**, select **PowerShell**.
4. Click **Save** and restart Windows Terminal.

#### Set PowerShell as Default for Command Prompt and Run Box
1. Open PowerShell (`pwsh`) and execute:
   ```powershell
   Set-ItemProperty -Path "HKCU:\Software\Microsoft\Command Processor" -Name Autorun -Value "\"C:\Program Files\PowerShell\7\pwsh.exe\""
   ```
2. Close and reopen `cmd`, `Run (Win+R)`, or any script executions to ensure they launch PowerShell.
