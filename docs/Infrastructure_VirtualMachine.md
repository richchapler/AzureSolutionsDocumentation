# Virtual Machine

<img src="https://user-images.githubusercontent.com/44923999/234043896-bdf9af61-4fa4-4c12-abfb-afa3ca9224dd.png" width="1000" />

[Microsoft Learn: Virtual Machines in Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/overview) and related documentation should serve as your primary source of information.

## Create with Azure Portal (SQL Server 2012 image)

<img src="https://user-images.githubusercontent.com/44923999/234047278-739c4423-d5cf-4460-9ac7-d1714661c15c.png" width="800" title="Snipped: April 24, 2023" />

Complete the "**Create a virtual machine**" form, "**Basics**" tab, including:

Prompt | Entry
:----- | :-----
**Availability options** | Select "**No infrastructure redundancy required**"
...**inbound ports** | Select "**Allow Selected Ports**" and "**RDP (3389)**"

Complete the “Create virtual machine” form, “SQL Server settings” tab, including:
SQL Connectivity	Select “Public (Internet)”
Port	Confirm default, 1433
SQL Authentication
Login Name
Password	Click to Enable and then confirm default (which will match your previously entered Administrator values)

Click the “Review + create” button, validate settings, and then click the Create button.
Configure SQL Server
Connect to the new virtual machine (using RDP) to complete image-specific configuration.
SQL Server Browser
Enable SQL Server Browser to allow connection to the SQL Server:
•	Open Services, search for and double-click to open “SQL Server Browser”
•	On the General tab, change “Startup Type” to Automatic, and then click the Apply button
•	Click the Start button
Configure IE ESC
Disable Internet Explorer Enhanced Security Configuration to allow downloading to the demonstration VM:
•	Open Server Manager and click the “Configure IE ESC” link in the “Server Summary” > “Security Information” interface grouping
•	Alternatively, navigate to “Local Server” and you will find “IE Enhanced Security Configuration” in the Properties grouping
•	Complete the “Internet Explorer Enhanced Security Configuration” pop-up, click Off under Administrators and Users, and then click the OK button
Sample Database
Browse to https://docs.microsoft.com/en-us/sql/samples/adventureworks-install-configure and complete the following steps:
•	Download the appropriate version of the AdventureWorks sample database {e.g., “AdventureWorks2008R2-oltp.bak”}
•	Open SQL Server Management Studio, right-click on Databases, and click “Restore Database” in the resulting drop-down menu
•	On the “Restore Database” pop-up, enter AdventureWorks in the “To database” text box, “Destination for restore” interface grouping
•	On the “Restore Database” pop-up, click the “From device” radio button in the “Source for restore” interface grouping
•	Click the ellipses button and in the resulting “Specify Backup” pop-up, click the Add button
•	Browse to the downloaded BAK file; you may have to copy the BAK from the Downloads folder to another location (like c:\temp) to make it accessible
•	Back on the “Restore Database” pop-up, check the box next to added item in “Select the backup sets to restore” and then click the OK button

When you have successfully completed these steps, you will see a pop-up that says “The restore of database ‘AdventureWorks” completed successfully.
 
## Create with Cloud Shell (Bash)
Navigate to Cloud Shell and select Bash from the “Select Environment” drop-down menu.

 

Update and execute the following command to set variable rg {i.e., the name of your Resource Group}:
rg="<UseCase>rg"

Update and execute the following command to set variable vm {i.e., the desired name for the new virtual machine}:
vm="<UseCase>vm"

Execute the following command to create your virtual machine:
az vm create --resource-group $rg --name $vm --image UbuntuLTS --assign-identity --admin-username rchapler --generate-ssh-keys

Successful execution will create the following resources:
•	Virtual Machine
•	Disk
•	Network Security Group
•	Public IP Address
•	Network Interface
•	Virtual Network

Update and execute the following command to connect to the server using SSH:
ssh -i ~/.ssh/id_rsa rchapler@{public ip address}

 

You will know you are successful if your prompt has changed to your equivalent of rchapler@<UseCase>vm.
