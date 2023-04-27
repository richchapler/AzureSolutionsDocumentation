# Migrate Data: SQL Server Hybridization

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
In this exercise, we will migrate a database from an earlier version of SQL Server.

### Step 1: Lorem Ipsum
Open SQL Server Management Studio and connect to both SQL Server 2019 and 2022 instances.

<img src="https://user-images.githubusercontent.com/44923999/234953201-01818d70-78c0-41e4-9a2b-b3eb4df83c76.png" width="800" title="Snipped: April 27, 2023" />

LOREM IPSUM

There are several ways to migrate a database from SQL Server 2019 to SQL Server 2022. Some of the options include using the Copy Database Wizard, restoring a database backup, or using the Generate Scripts Wizard to publish databases1. Another option is to use Transactional Replication2. You can also use Export/Import (also known as BACPAC) or Backup-restore to SQL Server 20222.

