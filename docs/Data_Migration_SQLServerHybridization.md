# Migrate Data: SQL Server Hybridization (WiP)

<img src="https://user-images.githubusercontent.com/44923999/235181250-128cb7c3-8ec2-4d1b-8653-39a9d9eb5c27.png" width="1000" />

## Use Case
This solution considers the following requirements:

* "We have thousands of on-prem SQL Servers and we are not ready to migrate them to Azure"
* "...that said, we want to take advantage of [Azure Hybrid Benefit](https://learn.microsoft.com/en-us/azure/azure-sql/azure-hybrid-benefit) for the discounts
* "...not to mention all the other benefits: scalability, business continuity, security, "single pane of glass" management, governance, etc."

## Prerequisites
This solution requires the following resources:

* On-prem development server with:
  * [**SQL Server 2019**](https://info.microsoft.com/ww-landing-sql-server-2019.html) default instance "**MSSQLSERVER**" with [AdventureWorks](https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure) sample database
  * [**SQL Server 2022**](https://info.microsoft.com/ww-landing-sql-server-2022.html) instance named "**SS22**", **Enterprise** edition (configured with Azure "Pay-as-you-Go" billing)
  * [**SQL Server Management Studio**](https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms)
* Azure
  * [**SQL**](https://learn.microsoft.com/en-us/azure/azure-sql) [Managed Instance](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/instance-create-quickstart)

## Proposed Solution
This solution will address requirements in three exercises:

* Exercise 1: Database Migration
* Exercise 2: Hybrid Connectivity
* Exercise 3: Activate Arc + SQL Server
* Exercise 4: Activate Purview

-----

## Exercise 1: Database Migration
In this exercise, we will migrate a database from SQL Server 2019 to SQL Server 2022.

There are several ways to migrate a database from SQL Server 2019 to SQL Server 2022:
* Backup-Restore
* Copy Database Wizard
* Generate Scripts Wizard
* Transactional Replication
* Export/Import (aka BACPAC)

This documentation covers only the first option.

### Step 1: Backup Database
Open **SQL Server Management Studio** and connect to both SQL Server 2019 and 2022 instances.

<img src="https://user-images.githubusercontent.com/44923999/234963219-df0478e4-3402-46fb-b477-c97eb92179f7.png" width="800" title="Snipped: April 27, 2023" />

Right-click on the **AdventureWorks2019** database in the SQL Server 2019 connection, then roll-over "**Tasks**" and click "**Back Up**..." in the resulting menus.

<img src="https://user-images.githubusercontent.com/44923999/234963644-c7aa4aac-50cf-4325-beab-5ae089405261.png" width="600" title="Snipped: April 27, 2023" />

Complete the resulting "Back Up Database - AdventureWorks2019" pop-up form:

Prompt | Entry
:----- | :-----
**Database** | Confirm selection "**AdventureWorks2019**"
**Backup type** | Confirm selection "**Full**"
**Back up to** | Confirm selection "**Disk**", then click "Add", browse to "**C:\Temp**" and enter file name "**AdventureWorks2019.bak**"

Click "**OK**", allow time for processing and confirm success with the resulting "The backup of database 'AdventureWorks2019' completed successfully".

### Step 1: Restore Database

<img src="https://user-images.githubusercontent.com/44923999/234965525-4bc8a8e2-7c3b-44e2-a2bb-c778be964487.png" width="800" title="Snipped: April 27, 2023" />

Right-click on **Databases** in the SQL Server 2022 connection and then click "**Restore Database**" in the resulting menu.

<img src="https://user-images.githubusercontent.com/44923999/234973981-5e240eeb-f32a-419b-a94d-bc4529829556.png" width="600" title="Snipped: April 27, 2023" />

Complete the resulting "Restore Database - AdventureWorks2019" pop-up form:

Prompt | Entry
:----- | :-----
**Device** | Select "**Device**" and then click "..."<br>Click "**Add**", then browse to and select "C:\Temp\" and "AdventureWorks2019.bak"
**Destination** | Confirm value "**AdventureWorks2019**"
**Restore to** | Confirm selection "**The last backup taken**..."

Click on the "**Files**" tab and check "**Relocate all files to folder**".

Click "**OK**", allow time for processing and confirm success.

<img src="https://user-images.githubusercontent.com/44923999/234976336-d91154e3-72b7-4ce2-89a3-dcfbf07f5e10.png" width="800" title="Snipped: April 27, 2023" />

-----

**Congratulations... you have successfully completed this exercise**

-----

## Exercise 2: Hybrid Connectivity
In this exercise, we will establish hybrid connectivity between on-prem SQL Server 2022 and an Azure SQL Managed Instance.

There are two ways to establish hybrid connectivity:
* Replicate Database ... the on-prem SQL Server is read/write and the replicated Azure SQL Managed Instance is read-only; changes are transferred near real-time
* Failover Database ... in the event of a disaster, manually failover to Azure SQL Managed Instance; after disaster mitigation, "fail back" to SQL Server 2022

This documentation covers only "**Failover Database**" because "Replicate Database" requires a Windows Server (and I only have my laptop for demonstration).

### Step 1: Create Managed Instance Link

Open **SQL Server Management Studio** and connect to the SQL Server 2022 instance.

<img src="https://user-images.githubusercontent.com/44923999/235476271-8eff885b-bf07-483b-ae82-ddfbecdd9cbe.png" width="800" title="Snipped: May 1, 2023" />

Right-click on the **AdventureWorks2019** database, then roll-over "**Azure SQL Managed Instance link**" and click "**Failover database**" in the resulting menus.

<img src="https://user-images.githubusercontent.com/44923999/235476369-24c61dbb-0444-44e8-a030-94bf6299e329.png" width="600" title="Snipped: May 1, 2023" />

Review "**Introduction**" page, then click "**Next >**".

<img src="https://user-images.githubusercontent.com/44923999/.png" width="600" title="Snipped: April 28, 2023" />







-----

## Reference
* [Copy Databases with Backup and Restore](https://learn.microsoft.com/en-us/sql/relational-databases/databases/copy-databases-with-backup-and-restore)
* [Editions and supported features of SQL Server 2022](https://learn.microsoft.com/en-us/sql/sql-server/editions-and-components-of-sql-server-2022)
* [Tutorial: Add SQL Managed Instance to a failover group](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/failover-group-add-instance-tutorial)
