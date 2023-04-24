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

Complete the "**Create a virtual machine**" form, "**SQL Server settings**" tab, including:

Prompt | Entry
:----- | :-----
**SQL connectivity** | Select "**Public (Internet)**"
**Port** | Confirm default "**1433**"
**SQL Authentication** | **Enable** and enter meaningful "**Login name**" and "**Password**" values (likely matching previously-entered Administrator credentials)

Click "**Review + create**", validate settings, and then click "**Create**".

### Configure SQL Server
Connect to the new virtual machine (using "**Download RDP file**") to complete image-specific configuration.

<img src="https://user-images.githubusercontent.com/44923999/234051398-464ce44e-0dbb-4942-8ef0-f54145692a2a.png" width="800" title="Snipped: April 24, 2023" />

#### Download Sample Database

On your local machine, search for and then download the AdventureWorks sample database (on your local machine).<br>
Copy the file to your Virtual Machine.<br>




•	Open SQL Server Management Studio, right-click on Databases, and click “Restore Database” in the resulting drop-down menu
•	On the “Restore Database” pop-up, enter AdventureWorks in the “To database” text box, “Destination for restore” interface grouping
•	On the “Restore Database” pop-up, click the “From device” radio button in the “Source for restore” interface grouping
•	Click the ellipses button and in the resulting “Specify Backup” pop-up, click the Add button
•	Browse to the downloaded BAK file; you may have to copy the BAK from the Downloads folder to another location (like c:\temp) to make it accessible
•	Back on the “Restore Database” pop-up, check the box next to added item in “Select the backup sets to restore” and then click the OK button

When you have successfully completed these steps, you will see a pop-up that says “The restore of database ‘AdventureWorks” completed successfully.
