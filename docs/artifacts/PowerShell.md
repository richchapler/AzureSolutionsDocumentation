# PowerShell

## Up-to-Date?

To verify that you have the latest version of PowerShell installed, open a PowerShell window as an administrator and run:

```powershell
winget list --id Microsoft.PowerShell
```

You might see output similar to:

```
Failed when searching source; results will not be included: msstore
Name             Id                   Version Source
-----------------------------------------------------
PowerShell 7-x64 Microsoft.PowerShell 7.5.0.0 winget
```

**How to interpret this output:**

- **Version Information:**  
  The version number shown (e.g., **7.5.0.0**) is the version currently installed on your system.  
- **Source Warning:**  
  The warning about "Failed when searching source; results will not be included: msstore" indicates that winget couldn’t query the Microsoft Store, but it still shows the version from the winget source.
- **Comparison:**  
  Compare the version number from winget with the latest release listed on the [PowerShell GitHub Releases page](https://github.com/PowerShell/PowerShell/releases).  
  - If the installed version matches the latest release, then you have the latest version installed.
  - If the version is lower, you should upgrade PowerShell.

## If not...

If the installed version is older than the latest release, execute the following command to upgrade PowerShell:

```powershell
winget install --id Microsoft.PowerShell --source winget
```

After upgrading, close the current window and open **PowerShell 7 (x64)** as administrator.

_Note: Windows PowerShell (version 5.x) remains installed for legacy compatibility._

<img src="https://github.com/richchapler/AzureSolutionsDocumentation/assets/44923999/ee8abf7a-d632-4d5c-be7c-c6e785d81c1f" width="600" title="Snipped March 13, 2025" />

------------------------- ------------------------- ------------------------- -------------------------

## Troubleshooting

### Check if PowerShell is Installed
Run:
```powershell
where.exe pwsh
```
- If a path is returned, PowerShell 7 is installed but may not be set as the default.

### Manually Launch PowerShell
Try running:
```powershell
pwsh
```
This should start PowerShell 7. Once launched, you can confirm the version by using the winget list command again.

### Ensure PowerShell is the Default Version
If `pwsh` works but running `powershell` still launches an older version:

#### Set PowerShell as Default in Windows Terminal
1. Open **Windows Terminal** (`wt` in Run or Search).
2. Click the **dropdown arrow** next to the new tab button (`+`) and select **Settings**.
3. Under **Default profile**, select **PowerShell** (which should point to the PowerShell 7 executable).
4. Click **Save** and restart Windows Terminal.

#### Set PowerShell as Default for Command Prompt and Run Box
1. Open PowerShell 7 (`pwsh`) and execute:
   ```powershell
   Set-ItemProperty -Path "HKCU:\Software\Microsoft\Command Processor" -Name Autorun -Value "\"C:\Program Files\PowerShell\7\pwsh.exe\""
   ```
2. Close and reopen `cmd`, `Run (Win+R)`, or any script executions to ensure they launch PowerShell 7.
