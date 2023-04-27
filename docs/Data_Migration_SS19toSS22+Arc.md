# Migrate Data: SQL Server 2022, Azure Hybrid, Arc and Purview

<img src="https://user-images.githubusercontent.com/44923999/234937691-8ce2332f-d836-4a7e-a306-6c5d14ef19f7.png" width="1000" />

## Use Case
This solution considers the following requirements:

* "We have thousands of on-prem SQL Servers and we are not ready to migrate them to Azure"
* "...that said, we want to take advantage of [Azure Hybrid Benefit](https://learn.microsoft.com/en-us/azure/azure-sql/azure-hybrid-benefit) for the discounts
* "...and of course all of the other benefits of hybridization: scalability, business continuity, security, "single pane of glass" management, governance, etc."

## Prerequisites
This solution requires the following resources:

* On-Prem Demonstration Machine with:
  * [SQL Server 2019](https://info.microsoft.com/ww-landing-sql-server-2019.html) (default instance) with [AdventureWorks](https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure) sample database
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

There are several ways to migrate a database from SQL Server 2019 to SQL Server 2022. Some of the options include using the Copy Database Wizard, restoring a database backup, or using the Generate Scripts Wizard to publish databases1. Another option is to use Transactional Replication2. You can also use Export/Import (also known as BACPAC) or Backup-restore to SQL Server 20222.

<img src="https://user-images.githubusercontent.com/44923999/234934326-6712a8cf-370f-4faa-9c79-0c8dc3f7fe08.png" width="800" title="Snipped: April 27, 2023" />
