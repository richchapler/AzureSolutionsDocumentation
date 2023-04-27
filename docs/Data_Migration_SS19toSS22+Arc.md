Requirements:
* On-Prem Demonstration Machine with:
  * [SQL Server 2019](https://info.microsoft.com/ww-landing-sql-server-2019.html) (default instance) with [AdventureWorks](https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure) sample database
  * [SQL Server 2022](https://info.microsoft.com/ww-landing-sql-server-2022.html) (instance named "**SS22**")
  * [SQL Server Management Studio](https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms)

Steps:
* Migrate database from SQL Server 2019 to SQL Server 2022
* Activate Azure Hybrid
* Activate Arc + SQL Server
* Activate Purview

## Exercise 1: Migrate Database

There are several ways to migrate a database from SQL Server 2019 to SQL Server 2022. Some of the options include using the Copy Database Wizard, restoring a database backup, or using the Generate Scripts Wizard to publish databases1. Another option is to use Transactional Replication2. You can also use Export/Import (also known as BACPAC) or Backup-restore to SQL Server 20222.
