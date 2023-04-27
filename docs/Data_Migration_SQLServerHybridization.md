# Migrate Data: SQL Server Hybridization (WiP)

<img src="https://user-images.githubusercontent.com/44923999/234937691-8ce2332f-d836-4a7e-a306-6c5d14ef19f7.png" width="1000" />

## Use Case
This solution considers the following requirements:

* "We have thousands of on-prem SQL Servers and we are not ready to migrate them to Azure"
* "...that said, we want to take advantage of [Azure Hybrid Benefit](https://learn.microsoft.com/en-us/azure/azure-sql/azure-hybrid-benefit) for the discounts
* "...not to mention all the other benefits: scalability, business continuity, security, "single pane of glass" management, governance, etc."

## Prerequisites
This solution requires the following resources:

* On-prem development server with:
  * [SQL Server 2019](https://info.microsoft.com/ww-landing-sql-server-2019.html) (default instance "**MSSQLSERVER**") with [AdventureWorks](https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure) sample database
  * [SQL Server 2022](https://info.microsoft.com/ww-landing-sql-server-2022.html) (instance named "**SS22**")
  * [SQL Server Management Studio](https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms)

## Proposed Solution
This solution will address requirements in three exercises:

* Exercise 1: Migrate Database to SQL Server 2022
* Exercise 2: Activate Azure Hybrid
* Exercise 3: Activate Arc + SQL Server
* Exercise 4: Activate Purview

-----

## Exercise 1: Migrate Database to SQL Server 2022
In this exercise, we will explore options for migrating a database from an earlier version of SQL Server.

There are several ways to migrate a database from SQL Server 2019 to SQL Server 2022; this documentation details step-by-step instructions for **some** of the options.

Option | Pros | Cons | Documented Below?
:----- | :----- | :----- | :-----
**Restore a database backup** | - Lorem | - Ipsum | No
**Copy Database Wizard** | - Lorem | - Ipsum | No
**Use the Generate Scripts Wizard to publish databases** | - Lorem | - Ipsum | No
**Transactional Replication** | - Lorem | - Ipsum | No
**Export/Import (also known as BACPAC)** | - Lorem | - Ipsum | No

### Option 1: Backup-Restore
Open **SQL Server Management Studio** and connect to both SQL Server 2019 and 2022 instances.

<img src="https://user-images.githubusercontent.com/44923999/234963219-df0478e4-3402-46fb-b477-c97eb92179f7.png" width="800" title="Snipped: April 27, 2023" />

Right-click on the **AdventureWorks2019** database in the SQL Server 2019 connection, then roll-over "**Tasks**" and click "**Back Up**..." in the resulting menus.

<img src="https://user-images.githubusercontent.com/44923999/234963644-c7aa4aac-50cf-4325-beab-5ae089405261.png" width="600" title="Snipped: April 27, 2023" />

Complete the resulting "Back Up Database - AdventureWorks2019" pop-up form:

Prompt | Entry
:----- | :-----
**Database** | Confirm selection "**AdventureWorks2019**"
**Backup type** | Confirm selection "**Full**"
**Back up to** | Confirm selection "**Disk**"<br>Click "Add", browse to "C:\Temp" and enter file name "AdventureWorks2019.bak"

Click "**Ok**", allow time for processing and confirm success with the resulting "The backup of database 'AdventureWorks2019' completed successfully".

<img src="https://user-images.githubusercontent.com/44923999/234965525-4bc8a8e2-7c3b-44e2-a2bb-c778be964487.png" width="600" title="Snipped: April 27, 2023" />

Right-click on **Databases** in the SQL Server 2022 connection and then click "**Restore Database**" in the resulting menu.




LOREM IPSUM

-----

**Congratulations... you have successfully completed this exercise**

-----

## Reference
* [Copy Databases with Backup and Restore](https://learn.microsoft.com/en-us/sql/relational-databases/databases/copy-databases-with-backup-and-restore)
* [Use the Copy Database Wizard](https://learn.microsoft.com/en-us/sql/relational-databases/databases/use-the-copy-database-wizard)
