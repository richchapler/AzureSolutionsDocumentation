# SQL Server Report Server (SSRS)

This documentation provides a complete, step‐by‐step guide for configuring a fully functioning SSRS 2019 instance on a VM that comes with SQL Server 2019 Reporting Services and SQL Server 2019 Standard on Windows Server 2022 (using the default port 80). In addition, it explains how to configure report caching and also notes the limitations when reports always run under the individual user’s credentials.

------

## 1. Report Server Configuration Manager

**Base Image:** SQL Server 2019 Reporting Services and SQL Server 2019 Standard on Windows Server 2022

1. Launch Report Server Configuration Manager:

   - Open Report Server Configuration Manager from the Start Menu.
   - Connect to the local SSRS 2019 instance.

2. Configure the Report Server Database:

   - Go to the **Database** tab.
   - Click **Change Database** and select “Create a new report server database.”
   - Follow the wizard to create the ReportServer and ReportServerTempDB databases on your SQL Server instance.
   - Confirm that the connection is valid and that encryption keys are properly set.

3. Configure URL Bindings:

   - In the **Web Service URL** tab, ensure the virtual directory is set to **ReportServer** and the port is set to **80**.

   - In the 

     Web Portal URL

      tab, ensure the virtual directory is set to 

     Reports

      and the effective URL is

     ```
     http://<hostname>/Reports
     ```

     (replace 

     ```
     <hostname>
     ```

      with your VM’s name).

   - Click **Apply** on both tabs and note the effective URLs.

4. Restart the SSRS Service:

   - Confirm that the SSRS service is running (using the Report Server Status tab or Services.msc).

------

## 2. Set Up SSRS Security and Permissions

Your account must have both item-level (folder) and system-level permissions to administer SSRS.

### a. Assign Item-Level Permissions (Folder Security)

1. Access the SSRS Web Portal:

   - Open your browser and navigate to

     ```
     http://<hostname>/Reports
     ```

2. Configure Folder Security:

   - Click the gear icon (Site Settings) in the upper right.
   - Go to the **Security** tab.
   - Add your account (for example, `YOURDOMAIN\YourUserName`) and assign the **Content Manager** role.

### b. Assign System-Level Permissions via PowerShell

1. **Create a Proxy to the SSRS SOAP API:**

   Execute these individual commands (replace `<hostname>` with your actual server name):

   ```powershell
   $ReportServiceUri = "http://<hostname>/ReportServer/ReportService2010.asmx?wsdl"
   $ssrs = New-WebServiceProxy -Uri $ReportServiceUri -Namespace "ReportingService2010" -UseDefaultCredential
   ```

2. **Obtain the Required Types:**

   ```powershell
   $policyType = $ssrs.GetType().Assembly.GetType("ReportingService2010.Policy")
   ```

   ```powershell
   $roleType = $ssrs.GetType().Assembly.GetType("ReportingService2010.Role")
   ```

3. **Create the BUILTIN\Administrators Policy:**

   ```powershell
   $adminPolicy = [Activator]::CreateInstance($policyType)
   ```

   ```powershell
   $adminPolicy.GroupUserName = "BUILTIN\Administrators"
   ```

   ```powershell
   $adminRole = [Activator]::CreateInstance($roleType)
   ```

   ```powershell
   $adminRole.Name = "System Administrator"
   ```

   ```powershell
   $adminPolicy.Roles = @($adminRole)
   ```

4. **Create Your User’s Policy:**

   Replace `YOURDOMAIN\YourUserName` with your actual account, then execute:

   ```powershell
   $newSysPolicy = [Activator]::CreateInstance($policyType)
   ```

   ```powershell
   $newSysPolicy.GroupUserName = "YOURDOMAIN\YourUserName"
   ```

   ```powershell
   $sysRole = [Activator]::CreateInstance($roleType)
   ```

   ```powershell
   $sysRole.Name = "System Administrator"
   ```

   ```powershell
   $newSysPolicy.Roles = @($sysRole)
   ```

5. **Combine and Apply the System Policies:**

   ```powershell
   $policyArray = [System.Array]::CreateInstance($policyType, 2)
   ```

   ```powershell
   $policyArray.SetValue($adminPolicy, 0)
   ```

   ```powershell
   $policyArray.SetValue($newSysPolicy, 1)
   ```

   ```powershell
   $ssrs.SetSystemPolicies($policyArray)
   ```

6. **Verify the System Policies:**

   ```powershell
   $ssrs.GetSystemPolicies()
   ```

   You should see both **BUILTIN\Administrators** and your account listed with the **System Administrator** role.

------

## 3. Configure Windows and Browser Settings for Integrated Authentication

For the SSRS web portal to load correctly, your browser must automatically pass your Windows credentials.

### a. Add the SSRS URL to the Local Intranet Zone

Execute these individual PowerShell commands (replace `yourhostname` with your actual hostname):

```powershell
New-Item -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Domains\yourhostname" -Force | Out-Null
New-Item -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Domains\yourhostname\80" -Force | Out-Null
New-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMap\Domains\yourhostname\80" -Name "*" -Value 1 -PropertyType DWORD -Force | Out-Null
Write-Host "Added http://yourhostname to Local Intranet zone."
```

### b. Disable Internet Explorer Enhanced Security Configuration (IE ESC)

Open **Server Manager** on your VM, select **Local Server**, and set **IE Enhanced Security Configuration** to **Off** for Administrators.

### c. Disable the Loopback Check

Execute these individual commands (replace `yourhostname` with your actual hostname):

```powershell
New-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Lsa" -Name "DisableLoopbackCheck" -Value 1 -PropertyType DWORD -Force | Out-Null
New-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Lsa\MSV1_0" -Name "BackConnectionHostNames" -Value "yourhostname" -PropertyType MultiString -Force | Out-Null
Write-Host "Loopback check disabled for 'yourhostname'."
```

### d. Force Automatic Logon for Intranet Sites

Execute this single command:

```powershell
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings\Zones\1" -Name "1A00" -Value 0x00030000 -Type DWord
Write-Host "Automatic logon with current credentials enabled for the Local Intranet zone."
```

### e. Reboot the VM

Reboot the VM so that all registry changes (IE ESC, loopback check, and IE zone settings) take effect.

------

## 4. Test the SSRS Web Portal

1. Open a Web Browser:

   - Use Microsoft Edge (or Internet Explorer configured in IE11/Edge mode).

2. Navigate to the SSRS Web Portal:

   - In the address bar, type:

     ```
     http://yourhostname/Reports
     ```

3. Verify the Interface Loads Correctly:

   - The SSRS portal should display its full, interactive web interface with no unauthorized errors.
   - Open Developer Tools (press F12) to confirm that all assets (CSS, JavaScript) load successfully and there are no script errors.

------

## 4A. Data Source Credential Requirements for Report Caching

For a report to be cached, SSRS requires that the report’s data source credentials are stored in the report server database or that an unattended execution account is specified. **Not all reports can be cached.** Microsoft states:

> "Not all reports can be cached. If a report includes user-dependent data, prompts users for credentials, or uses Windows Authentication, it can't be cached."

When you select **"Use current Windows user credentials"**, those credentials are not stored in the report server. To enable caching:

1. Open the SSRS Web Portal:

   - Navigate to `http://yourhostname/Reports`.

2. Locate and Manage the Data Source:

   - Find the data source used by your report.
   - Click the ellipsis (…) next to the data source and select **Manage**.

3. Set Credentials:

   - Under the **Credentials** section, select **Use credentials stored securely in the report server**.
   - Enter the appropriate username and password.
   - Click **Test Connection** to ensure a successful connection.
   - Click **Apply**.

   This configuration is required for report caching. Without stored credentials, SSRS will not allow caching.

------

## 5. Create and Configure a Dummy Report to Test Report Cache Expiration

This section creates a simple report that displays the current date and time, and configures caching so that the report output is cached for 3 minutes.

### a. Create a Dummy Report with Report Builder

1. Open Report Builder:

   - Launch Report Builder on your machine.

2. Create a New Blank Report:

   - Click **New Report** and select **Blank Report**.

3. Add an Embedded Dataset:

   - In the Report Data pane, right-click **Datasets** and choose **Add Dataset**.

   - Choose **Use a dataset embedded in my report**.

   - Select or create a data source that connects to your SQL Server.

   - Enter the following query:

     ```sql
     SELECT GETDATE() AS CurrentTime
     ```

4. Design the Report Layout:

   - Insert a **Textbox** onto the report canvas.
   - Set the textbox expression to: `=Fields!CurrentTime.Value`

5. Save the Report:

   - Save it locally as “CacheTestReport.rdl.”

### b. Deploy the Dummy Report to SSRS

1. Open the SSRS Web Portal:
   - Navigate to `http://yourhostname/Reports`.
2. Upload the Report:
   - Click **Upload** and select “CacheTestReport.rdl.”
   - Deploy the report into your desired folder (e.g., the **Home** folder).

### c. Configure Data Source Credentials for the Report

1. Open the SSRS Web Portal:
   - Navigate to `http://yourhostname/Reports`.
2. Locate the Data Source:
   - Find the data source used by “CacheTestReport.”
   - Click the ellipsis (…) next to the data source and choose **Manage**.
3. Set the Credentials:
   - Under the **Credentials** section, select **Use credentials stored securely in the report server**.
   - Enter the appropriate username and password.
   - Click **Test Connection** to confirm a successful connection.
   - Click **Apply**.

### d. Configure Report Cache Expiration via the SSRS SOAP API

Execute these individual PowerShell commands (replace `yourhostname` with your actual hostname):

1. Set the Report Path:

   ```powershell
   $reportPath = "/CacheTestReport"
   ```

2. Create a Proxy to the SSRS SOAP API:

   ```powershell
   $ReportServiceUri = "http://yourhostname/ReportServer/ReportService2010.asmx?wsdl"
   ```

   ```powershell
   $ssrs = New-WebServiceProxy -Uri $ReportServiceUri -Namespace "ReportingService2010" -UseDefaultCredential
   ```

3. Obtain the TimeExpiration Type:

   ```powershell
   $expirationType = $ssrs.GetType().Assembly.GetType("ReportingService2010.TimeExpiration")
   ```

4. Create a TimeExpiration Object:

   ```powershell
   $timeExpiration = [Activator]::CreateInstance($expirationType)
   ```

5. Set the Cache Expiration to 3 Minutes:

   ```powershell
   $timeExpiration.Minutes = 3
   ```

6. Apply the Cache Options to the Report:

   ```powershell
   $ssrs.SetCacheOptions($reportPath, $true, $timeExpiration)
   ```

7. Verify the Cache Settings:

   ```powershell
   $cacheOptions = $ssrs.GetCacheOptions($reportPath)
   ```

   ```powershell
   $cacheOptions
   ```

   The output should confirm that caching is enabled with a 3‑minute expiration.

### e. Test Report Cache Expiration

1. Run the Report:
   - Open your web browser and navigate to `http://yourhostname/Reports`.
   - Run “CacheTestReport” and note the displayed current date/time.
2. Observe the Cached Value:
   - Immediately refresh the report—the displayed date/time should remain the same.
3. Wait 3 Minutes and Refresh:
   - After 3 minutes, refresh the report again.
4. Verify the Change:
   - The report should display an updated date/time value, confirming that the cache expired and a new instance of the report was generated.

------

## 6. Important Note on Caching and User Credentials

Caching in SSRS requires that the report's data source credentials be stored securely in the report server database or that an unattended execution account is specified. If you configure the report to **always run with the report user's credentials** (for example, by selecting **Use current Windows user credentials**), those credentials are not stored in the report server, and SSRS will not allow the report to be cached.

In other words, if you need a report to always run under each user's individual credentials, caching (and hence cache expiration) is not available. You must store the data source credentials securely for caching to work.

------

By following these instructions exactly, you will have:

- A fully configured SSRS 2019 instance running on port 80 on your Windows Server 2022 VM.
- Proper security and integrated authentication settings.
- A dummy report deployed with a data source that uses securely stored credentials.
- Report caching configured to cache the report output for 3 minutes for testing cache expiration.
