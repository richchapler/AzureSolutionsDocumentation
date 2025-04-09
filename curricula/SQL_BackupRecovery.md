# SQL: Backup and Recovery

## Use Case

The database administration team at a mid‑sized organization must ensure that critical data is protected against corruption, accidental deletion, or system failures. Key requirements:

* Data Protection: Guarding against data loss with reliable backup strategies
* Rapid Recovery: Minimizing downtime and data loss by restoring data efficiently
* Compliance & Security: Meeting regulatory and internal standards by securing backup files and validating restore procedures
* Operational Continuity: Ensuring that data can be recovered quickly during planned maintenance or unexpected outages

------------------------- -------------------------

## On‑Prem

### Fundamentals

#### Backup

##### Types

* **Full Backup**: A full backup is a complete snapshot of the entire database at a specific moment—a “photo” of your database. This backup serves as the foundational baseline for all subsequent backup activities.

* **Differential Backup**: Differential backups capture only the changes made since the last full backup. They act like incremental snapshots that record what’s different from the baseline. Over time, they provide a cumulative record of modifications while reducing the amount of data to be backed up compared to a full backup.

* **Transaction Log Backup**: For databases using a full or bulk‑logged recovery model, transaction log backups record every transaction that occurs after the last backup. This continuous log enables administrators to capture near real‑time changes and supports point‑in‑time recovery.

* **Ad Hoc (COPY_ONLY) Backups**: COPY_ONLY backups provide an independent snapshot without disturbing the regular backup sequence. They’re ideal for ad‑hoc scenarios—such as before major changes or testing—ensuring that your scheduled backup strategy remains intact.
  > Note: In Always‑On Availability Groups use COPY_ONLY when backing up a secondary replica to avoid breaking the primary replica’s log chain

* **Tail‑Log Backup**: Captures the active portion of the transaction log (the “tail”) that hasn’t yet been backed up. Use this immediately before a restore operation to ensure no committed transactions are lost.

##### Designing a Backup Plan

A robust backup plan starts with understanding the criticality of your data and how frequently it changes. Key elements include:

* **Frequency**: Decide how often to perform each type of backup. Full backups, which are larger, are scheduled less frequently, while differential and transaction log backups are run more often. The frequency should align with business needs and ensure that any potential data loss falls within your predetermined Recovery Point Objective (RPO).

* **Storage Locations**: Choose between on‑site storage for rapid recovery and off‑site or cloud‑based storage for disaster recovery. Secure, redundant storage solutions—ideally with encryption—minimize the risk of data loss due to localized failures  
  note for Azure VM backups select the appropriate Azure Storage redundancy options  
  * locally‑redundant storage (LRS) for three copies in one region  
  * zone‑redundant storage (ZRS) for copies across availability zones in one region  
  * geo‑redundant storage (GRS) to replicate to a secondary region  
  * read‑access geo‑redundant storage (RA‑GRS) to enable read access in the secondary region  

* **Automation**: Implement automated scheduling and monitoring for backup operations. Automation minimizes human error by consistently executing backup jobs and provides real‑time alerts if issues arise, ensuring backups complete successfully and are available when needed.

#### Restore

##### Restore and Recovery Procedures

Restoring a database reverses the backup process and consists of these steps:

* **Full Backup Restoration:** Restore the most recent full backup **WITH NORECOVERY** to keep the database offline and allow additional backups to be applied
 
* **Differential Backup Application:** Apply the latest differential backup **WITH NORECOVERY** to continue stacking changes without bringing the database online

* **Transaction Log Restoration:** Restore each transaction log backup **WITH NORECOVERY**, then for the final log use **WITH RECOVERY** (or **WITH STOPAT = 'timestamp', RECOVERY** for point‑in‑time) to bring the database online
 
* **Tail‑Log Restore (Optional):** Backup and restore the tail of the log **WITH NORECOVERY** immediately before the final recovery to capture any remaining transactions

> **Example: Mid‑Week Point‑in‑Time Recovery**  
> Suppose you run:  
> * Weekly full backups on Sunday  
> * Differential backups nightly at 01:00  
> * Transaction‑log backups every 5 minutes  
>  
> If the database fails on **Wednesday at 10:00**, your restore sequence is:  
> 1. **Restore Sunday’s full backup** WITH NORECOVERY  
> 2. **Restore Wednesday’s differential backup** (taken at 01:00) WITH NORECOVERY  
> 3. **Restore transaction‑log backups** from 01:00 up to 10:00 WITH STOPAT='2025-04-08T10:00:00' AND NORECOVERY  
> 4. *(Optional)* Perform a **tail‑log backup** and restore it WITH NORECOVERY to capture any final transactions  
> 5. **Bring the database online** WITH RECOVERY

------------------------- -------------------------

##### Best Practices

Effective restoration is as critical as the backup process. Best practices include:

* **Validation**: Regularly test restore procedures using representative sample data.  
  *Example:* Quarterly restore of last week’s full backup plus the latest differential into a sandbox server to verify end‑to‑end recovery.

* **Documentation**: Maintain detailed records of backup jobs—including timestamps, storage locations, and outcomes.  
  *Example:* Log each backup’s success or failure in a central table, then review monthly for trends or gaps.

* **Maintenance**: Continuously monitor backup file integrity and manage storage resources.  
  *Example:* Automate deletion of backups older than your RPO + one day to reclaim space and enforce retention policies.

------------------------- -------------------------

#### Key Considerations

A successful backup and recovery strategy depends on:

* **Data Integrity**: Ensuring that each backup builds correctly on the previous one so that the restore process maintains a consistent database state

* **Thorough Documentation and Testing**: Keeping detailed records and regularly simulating restore procedures to verify that recovery will be seamless in the event of an actual failure

* **Automation and Monitoring**: Using automated scheduling and proactive monitoring to minimize human error and quickly detect any issues in the backup process

------------------------- -------------------------

#### Key Metrics

* **Recovery Time Objective (RTO)**: The **maximum acceptable downtime following a disruption**. It is determined by measuring the time from failure detection, through the recovery process, until full restoration. This metric helps guide investments in technology and process improvements to minimize downtime.

* **Recovery Point Objective (RPO)**: The **maximum amount of data (expressed in time) that an organization can afford to lose**. It is calculated based on the frequency of backup operations and the rate of data change. For instance, if transaction logs are backed up every 15 minutes, the RPO would be approximately 15 minutes.

------------------------- -------------------------

#### Advanced Concepts

* **Backup Optimization**  
  * **Description**: Combine compression and encryption to optimize storage and secure backup data  
  * **Benefits**:  
    * Compression reduces file size and speeds up transfers  
    * Encryption protects data at rest  
  * **Considerations**:  
    * Compression may increase CPU usage during backup  
    * Encryption requires careful key management  

* **Verification and Maintenance**  
  * **Description**: Regularly verify backup integrity and perform maintenance to keep storage healthy  
  * **Benefits**:  
    * RESTORE VERIFYONLY confirms backup readability  
    * WITH CHECKSUM detects I/O or hardware errors at backup time  
  * **Considerations**:  
    * Verification adds time to the backup window  
    * Maintenance tasks must be scheduled to avoid conflicts  

* **Disaster Recovery Planning**  
  * **Description**: Combine off‑site or cloud backups with proactive monitoring for comprehensive DR readiness  
  * **Benefits**:  
    * Off‑site or cloud storage protects against site‑wide failures  
    * Automated alerts ensure prompt response to backup issues  
  * **Considerations**:  
    * Off‑site transfers can incur additional bandwidth costs  
    * Monitoring requires proper configuration and tuning  

* **Log Shipping**  
  * **Description**: Automate transaction‑log backups on the primary, copy them to a secondary, and restore on a schedule  
  * **Benefits**:  
    * Provides a warm standby without requiring clustering or shared storage  
    * Simple to implement and monitor via SQL Server Agent jobs  
  * **Considerations**:  
    * Failover is manual—applications must reconnect to the secondary  
    * Secondary database remains in a restoring state unless using STANDBY mode  

* **Corruption Recovery**  
  * **Description**: Use DBCC CHECKDB to detect corruption and built‑in repair options to recover a damaged database  
  * **Benefits**:  
    * Built‑in mechanism to detect and correct corruption  
    * Can recover a database even when backups are not fully current  
  * **Considerations**:  
    * Repair options that allow data loss can discard corrupted data  
    * Requires single‑user mode and downtime during repair operations

------------------------- -------------------------

#### Security Considerations

* **Backup File Security**:  
  * **Encryption at Rest**: Always encrypt backups using keys stored in a secure vault (HSM or Key Vault)  
  * **Encryption in Transit**: Enforce TLS/SSL for any network transfers of backup files  
  * **Immutable Storage**: Use write‑once, read‑many (WORM) or immutable storage policies for critical backups  

* **Access Controls & Auditing**:  
  * **Role‑Based Access**: Separate backup and restore privileges among different teams  
  * **Audit Trails**: Log and monitor all file‑access operations on backup storage  
  * **Alerting**: Configure alerts for unauthorized access or deletion attempts  

* **Key Management**:  
  * **Centralized Key Vault**: Store and manage encryption keys in a secure, centralized location  
  * **Key Rotation**: Rotate encryption keys on a regular schedule to reduce risk  

* **Network Security**:  
  * **Isolated Backup Network**: Restrict backup traffic to a dedicated network segment or VPN  
  * **Firewall Rules**: Limit storage account or file‑share access to approved IP ranges  

* **Compliance & Retention**:  
  * **Retention Policies**: Align backup retention schedules with industry and regulatory requirements, and implement retention locks where supported  
  * **Integrity Verification**: Maintain and periodically verify cryptographic hashes of backup files to detect silent corruption

------------------------- -------------------------

#### Monitoring Considerations

* **Automated Monitoring**:  
  * **Alerts**: Configure tools (such as SQL Server Agent jobs) to notify administrators of backup failures  
  * **Dashboard Views**: Use monitoring dashboards and DMVs to track backup status and job histories  

* **Job Failure Notifications**:  
  * **Description**: Use Database Mail and SQL Server Agent operators to send email/SMS alerts when a backup job fails  
  * **Benefits**:  
    * Immediate awareness of backup issues  
    * Faster response to prevent gaps in protection  
  * **Considerations**:  
    * Requires configuring Database Mail and operator permissions  
    * Ensure mail profiles are secured and tested  

* **Regular Reviews**:  
  * **Health Checks**: Periodically review backup logs and assess storage capacity  
  * **Testing**: Schedule routine restore tests to confirm backup reliability and readiness

------------------------- -------------------------
------------------------- -------------------------

### Exercise

#### Prepare Environment

One machine with SQL Server and the following items installed:  
  * [SQL Server Management Studio](https://learn.microsoft.com/en-us/ssms/download-sql-server-management-studio-ssms)  
  * [AdventureWorks](https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure?view=sql-server-ver16&tabs=ssms)

------------------------- -------------------------

#### Prepare Database

* **Verify Database**  
  Ensure that the AdventureWorks database is accessible and properly named {e.g., `AdventureWorks2022`}. This confirmation is vital because all subsequent backup operations will reference this database.

* **Set Recovery Model**  
  Set or confirm that the database is using the Full Recovery Model. The recovery model dictates how transactions are logged, how the transaction log is maintained, and ultimately, how much data you could lose in the event of a failure.

  * **Transaction Logging**: In the Full Recovery Model, every transaction is fully logged. This detailed logging allows you to perform transaction log backups, which are critical for point‑in‑time recovery. Without a proper recovery model, you might be unable to recover all the transactions that occurred between backups.  
  * **Data Protection**: With full logging, you ensure that every data change is recorded. This means that if a failure occurs, you can restore the database to the exact moment before the failure, minimizing data loss.  
  * **Recovery Objectives**: The recovery model directly impacts your Recovery Point Objective (RPO) — a key metric that defines how much data loss is acceptable. A Full Recovery Model supports a low RPO because you can recover the most recent transactions.

  Execute the following T‑SQL:
  ```sql
  USE master;
  GO
  ALTER DATABASE AdventureWorks2022 SET RECOVERY FULL;
  GO
  ```

------------------------- -------------------------

#### Backup Database

##### Full Backup

Performing a full backup creates a complete snapshot of the entire `AdventureWorks2022` database at a specific moment. This backup serves as the baseline for your backup chain.

  * **Baseline for Recovery**: The full backup is the foundational component in any backup strategy. When you restore your database, **you always start with the most recent full backup**. Without it, you wouldn’t have a complete reference of your database’s state.  
  * **Establishing a Recovery Point**: The full backup defines a clear recovery point. If a failure occurs, you will rely on this backup (and subsequent backups) to bring your database back to a known, consistent state.  
  * **Efficiency in Subsequent Backups**: Once the full backup is in place, subsequent differential backups can capture only the changes made since that full backup, reducing the amount of data that must be stored and later restored.

  Execute the following T‑SQL:
  ```sql
  BACKUP DATABASE AdventureWorks2022 TO DISK = 'C:\Temp\AdventureWorks_full.bak' WITH INIT;
  GO
  ```

Example Output:
```text
Processed 25376 pages for database 'AdventureWorks2022', file 'AdventureWorks2022' on file 1.
Processed 2 pages for database 'AdventureWorks2022', file 'AdventureWorks2022_log' on file 1.
BACKUP DATABASE successfully processed 25378 pages in 0.439 seconds (451.621 MB/sec).

Completion time: 2025-04-07T14:40:13.1775263-07:00
```

###### Verify Backup

After executing the full backup, check that the backup file (`AdventureWorks_full.bak`) was created in the designated folder. This verification is important to ensure that your backup process is working correctly and that you have a reliable starting point for recovery.

Execute the following T‑SQL to verify the backup file:
```sql
RESTORE VERIFYONLY FROM DISK = 'C:\Temp\AdventureWorks_full.bak';
```

------------------------- -------------------------

##### Differential Backup

###### Change Data

Execute the following T‑SQL to create a new table and insert data.
```sql
USE AdventureWorks2022;
GO

CREATE TABLE dbo.TestData ( ID INT IDENTITY(1,1) PRIMARY KEY, Description NVARCHAR(100) );
GO

INSERT INTO dbo.TestData (Description) VALUES ('Test record 1'), ('Test record 2');
GO
```

Simulating data changes illustrates the purpose of a differential backup — it captures only the modifications made since the last full backup. Students will see that a differential backup file is typically smaller and faster to create.

###### Perform Backup

Execute the following T‑SQL:
```sql
BACKUP DATABASE AdventureWorks2022 TO DISK = 'C:\Temp\AdventureWorks_diff.bak' WITH DIFFERENTIAL, INIT;
GO
```

Example Output:
```text
Processed 296 pages for database 'AdventureWorks2022', file 'AdventureWorks2022' on file 1.
Processed 2 pages for database 'AdventureWorks2022', file 'AdventureWorks2022_log' on file 1.
BACKUP DATABASE WITH DIFFERENTIAL successfully processed 298 pages in 0.049 seconds (47.433 MB/sec).

Completion time: 2025-04-08T05:58:38.1381296-07:00
```

###### Verify Backup

After executing the differential backup, check that the backup file (`AdventureWorks_diff.bak`) was created in the designated folder. Observe:

* **File Size**: Compare the size of the differential and full backup files  
* **Backup Duration**: Differential backups usually complete faster than full backups since they contain only recent changes

Execute the following T‑SQL to verify the differential backup:
```sql
RESTORE VERIFYONLY FROM DISK = 'C:\Temp\AdventureWorks_diff.bak';
```

------------------------- -------------------------

##### Transaction Log Backup

Execute the following T‑SQL:
```sql
BACKUP LOG AdventureWorks2022 TO DISK = 'C:\Temp\AdventureWorks_tlog.bak' WITH INIT;
GO
```

Example Output:
```text
Processed 18 pages for database 'AdventureWorks2022', file 'AdventureWorks2022_log' on file 1.
BACKUP LOG successfully processed 18 pages in 0.015 seconds (9.114 MB/sec).

Completion time: 2025-04-08T06:05:16.9254135-07:00
```

###### Verify Backup

After executing the transaction log backup, check that the backup file (`AdventureWorks_tlog.bak`) was created in the designated folder. Observe:

* **File Size**: The log backup file should be relatively small since it captures only the transactions that occurred since the previous log backup  
* **Backup Duration**: Transaction log backups typically complete very quickly, reflecting the limited amount of data captured

Execute the following T‑SQL to verify the log backup:
```sql
RESTORE VERIFYONLY FROM DISK = 'C:\Temp\AdventureWorks_tlog.bak';
GO
```

------------------------- -------------------------

###### Backup History

After executing your backups, query the msdb backup history to confirm when and how your backups ran:

```sql
SELECT 
  bs.database_name,
  bs.type,
  bs.backup_start_date,
  bm.physical_device_name
FROM msdb.dbo.backupset bs
JOIN msdb.dbo.backupmediafamily bm
  ON bs.media_set_id = bm.media_set_id
WHERE bs.database_name = 'AdventureWorks2022'
ORDER BY bs.backup_start_date;
GO
```

------------------------- -------------------------

#### Restore Database

Before restoring, simulate a failure by dropping the AdventureWorks2022 database. This will mimic an unexpected outage or data loss scenario.
```sql
USE master;
GO
DROP DATABASE AdventureWorks2022;
GO
```

> **Note**: Dropping the database is for demonstration purposes only. In a real-world scenario, you would perform a restore without deleting production data.

------------------------- -------------------------

##### Full Restore

Execute the following T-SQL to restore the full backup, which provides the complete baseline of the database.
```sql
RESTORE DATABASE AdventureWorks2022 FROM DISK = 'C:\Temp\AdventureWorks_full.bak' WITH RECOVERY;
GO
```

Example Output:
```text
Processed 25376 pages for database 'AdventureWorks2022', file 'AdventureWorks2022' on file 1.
Processed 2 pages for database 'AdventureWorks2022', file 'AdventureWorks2022_log' on file 1.
RESTORE DATABASE successfully processed 25378 pages in 0.270 seconds (734.302 MB/sec).

Completion time: 2025-04-08T06:20:18.1914188-07:00
```

------------------------- -------------------------

##### Differential Restore

**Scenario**:  
Imagine you take a full backup every Sunday night and then run a differential backup each subsequent night (Monday through Saturday). By Friday, the Friday differential contains all changes since Sunday’s full backup. When you need to restore on Friday, you don’t have to apply six separate log backups—just the full backup and the latest differential.

**Restore the Full Backup (Weekly Baseline)**  
```sql
RESTORE DATABASE AdventureWorks2022 FROM DISK = 'C:\Temp\AdventureWorks_full.bak' WITH NORECOVERY;
GO
```

This reestablishes your Sunday night baseline without bringing the database online, so you can apply the differential next.

**Restore the Friday Differential (Daily Catch‑Up) and Bring Online**  
```sql
RESTORE DATABASE AdventureWorks2022 FROM DISK = 'C:\Temp\AdventureWorks_diff.bak' WITH RECOVERY;
GO
```

**Why This Strategy Works**:  
* **Efficiency**: You avoid replaying multiple transaction logs—just one differential backup captures all interim changes  
* **Reduced Downtime**: Fewer restore steps mean faster recovery, helping you meet your Recovery Time Objective (RTO)  
* **Simplicity**: A weekly full plus daily differential schedule balances storage use and recovery speed, aligning with most operational needs

------------------------- -------------------------

##### Transaction Log Restore

**Scenario**:  
You run hourly transaction log backups to capture every change. Suppose at 3:45 PM someone accidentally deletes critical rows from `Sales.SalesOrderDetail`. To recover up to 3:44 PM, you’ll replay the log backup taken just before the mistake.

**Restore the Full Backup (Weekly Baseline)**  
```sql
RESTORE DATABASE AdventureWorks2022 FROM DISK = 'C:\Temp\AdventureWorks_full.bak' WITH NORECOVERY;
GO
```

This reestablishes your baseline without bringing the database online, so you can apply subsequent backups.

**Restore the Differential Backup (Daily Catch‑Up)**  
```sql
RESTORE DATABASE AdventureWorks2022 FROM DISK = 'C:\Temp\AdventureWorks_diff.bak' WITH NORECOVERY;
GO
```

Applies all changes since the full backup, getting you up to yesterday’s end‑of‑day state.

**Restore the Transaction Log Backup (Point‑in‑Time Recovery)**  
```sql
RESTORE LOG AdventureWorks2022 FROM DISK = 'C:\Temp\AdventureWorks_tlog.bak' WITH STOPAT = '2025-04-08T15:44:00', RECOVERY;
GO
```

The `STOPAT` option replays transactions up to 3:44 PM, just before the unwanted delete, and `RECOVERY` brings the database online.

**Why This Strategy Works**:  
* **Precision**: Transaction log backups record every transaction, enabling you to restore to an exact point in time and minimize data loss  
* **Efficiency**: Log backups are small and quick to apply, reducing the time required compared to larger full or differential restores  
* **Flexibility**: Combining full, differential, and log backups provides a layered approach—balancing storage use with the ability to perform both broad and granular recoveries

------------------------- -------------------------
------------------------- -------------------------

### Quiz

1. You run weekly full backups and nightly differential backups. On Wednesday morning, which sequence restores the database in two steps?  
   A) Restore full backup WITH RECOVERY; then restore differential backup WITH RECOVERY  
   B) Restore full backup WITH NORECOVERY; then restore differential backup WITH RECOVERY  
   C) Restore full backup WITH NORECOVERY; then restore differential backup WITH NORECOVERY  
   D) Restore full backup WITH RECOVERY; then restore differential backup WITH NORECOVERY  

2. To detect I/O or hardware errors during the backup process, which BACKUP option should you include?  
   A) WITH CHECKSUM  
   B) WITH COMPRESSION  
   C) WITH FORMAT  
   D) WITH STATS  

3. Which backup type should you schedule every 15 minutes to minimize data loss and control log growth?  
   A) Full Backup  
   B) Differential Backup  
   C) Transaction Log Backup  
   D) COPY_ONLY Backup  

4. Which restore command syntax applies a full and differential backup together in one step?  
   A) RESTORE DATABASE ... FROM DISK='full.bak', DISK='diff.bak' WITH RECOVERY  
   B) RESTORE DATABASE ... FROM DISK='full.bak' WITH NORECOVERY; RESTORE DATABASE ... FROM DISK='diff.bak' WITH RECOVERY  
   C) RESTORE DATABASE ... FROM DISK='diff.bak' WITH RECOVERY  
   D) RESTORE DATABASE ... FROM DISK='full.bak' WITH RECOVERY; RESTORE DATABASE ... FROM DISK='diff.bak' WITH NORECOVERY  

5. Which restore option leaves the database read‑only while allowing additional log restores?  
   A) WITH NORECOVERY  
   B) WITH RECOVERY  
   C) WITH REPLACE  
   D) WITH STANDBY  

6. To receive an email when a backup job fails, which components must be configured?  
   A) Database Mail, an Operator, and an Alert  
   B) Agent proxy, Credential, and Job step  
   C) Maintenance Plan, Profile, and Schedule  
   D) Extended Event, Operator, and Response  

7. Which disaster recovery method provides a warm standby without requiring clustering?  
   A) Always On Availability Groups  
   B) Log Shipping  
   C) Database Mirroring  
   D) Snapshot Replication  

8. Which DBCC CHECKDB option should be used only as a last resort when no valid backups exist?  
   A) REPAIR_REBUILD  
   B) REPAIR_FAST  
   C) REPAIR_ALLOW_DATA_LOSS  
   D) REPAIR_MASTER  

9. Which query returns the history of all backups for a database?  
   A) SELECT * FROM sys.dm_db_backupset  
   B) SELECT * FROM msdb.dbo.backupset  
   C) EXEC sp_help_backuphistory  
   D) DBCC SHOWBACKUPHISTORY  

10. Which security practice ensures backup files cannot be modified or deleted?  
    A) Role‑based access control  
    B) Immutable storage policies (WORM)  
    C) File system encryption  
    D) Network isolation  

------------------------- -------------------------

#### Answers

1. **B** – Restore the full backup with NORECOVERY, then apply the differential with RECOVERY  
   *This two‑step approach applies only the necessary backups and brings the database online quickly.*

2. **A** – WITH CHECKSUM  
   *Including CHECKSUM in the BACKUP command catches I/O or hardware errors at backup time.*

3. **C** – Transaction Log Backup  
   *Frequent log backups minimize data loss and truncate the log, preventing uncontrolled growth.*

4. **A** – RESTORE DATABASE … FROM DISK='full.bak', DISK='diff.bak' WITH RECOVERY  
   *Specifying both backup files in one command simplifies the restore process and reduces downtime.*

5. **D** – WITH STANDBY  
   *STANDBY leaves the database in read‑only mode while still allowing further log restores.*

6. **A** – Database Mail, an Operator, and an Alert  
   *These components work together to send notifications when a backup job fails.*

7. **B** – Log Shipping  
   *Log Shipping provides a warm standby by automating log backup, copy, and restore without clustering.*

8. **C** – REPAIR_ALLOW_DATA_LOSS  
   *This DBCC CHECKDB option may discard corrupted data and should only be used if no backups are available.*

9. **B** – SELECT * FROM msdb.dbo.backupset  
   *The backupset table in msdb contains detailed metadata for all database backups.*

10. **B** – Immutable storage policies (WORM)  
    *Write‑once, read‑many storage prevents deletion or modification of backup files, even by administrators.*

------------------------- -------------------------
------------------------- -------------------------
------------------------- -------------------------

## Azure

While on-premises backup strategies require manual configuration, Azure SQL Database and Managed Instance come with built-in backup capabilities that simplify these tasks. This section highlights the differences and considerations unique to Azure-based platforms without repeating core on-prem concepts.

### Fundamentals

* **Automated Backups**  
  * Azure automatically performs:  
    * **Full backups** – weekly  
    * **Differential backups** – every 12 hours  
    * **Transaction log backups** – every 5–10 minutes  
  * All backups are stored in geo‑redundant storage (GRS) by default, providing high durability and regional failover capability  

* **Restoration Options**  
  * **Point‑in‑Time Restore (PITR)**: Restore a database to any point within the retention period  
  * **Long‑Term Retention (LTR)**: Archive weekly full backups for up to 10 years, useful for audit/compliance scenarios  
  * **Geo‑Restore**: Restore the latest backup to a different region, even if the original region is inaccessible  

* **Backup Visibility and Limitations**  
  * Users cannot directly access or download Azure SQL Database backup files  
  * Backup configuration is managed through the Azure Portal, CLI, PowerShell, or ARM templates  

#### Platform Differences

##### Azure SQL Database
* **Backups are fully managed**: PITR is available within the configured retention window (default: 7 days, configurable up to 35)
* **Geo-redundant backups**: Enabled by default. Can be restored to any Azure region
* **LTR (Long-Term Retention)**: Enables weekly full backups to be retained for years in separate blob containers
* **No access to .bak files**: All restores must be initiated via platform interfaces

##### Azure SQL Managed Instance
* **Supports traditional backups**: You can use `BACKUP` and `RESTORE` T-SQL commands to/from Azure Blob Storage
* **Automated backups**: Follows same PITR and LTR models as Azure SQL Database
* **Full native restore compatibility**: Enables restores from existing .bak files migrated from on-prem

##### Azure Backup for SQL Server on VMs
* **Centralized VM‑level backups**: Use Azure Backup to protect SQL Server databases running on Azure IaaS VMs  
* **Point‑in‑time restores**: Restores individual databases or entire VM to any point within retention window  
* **Managed by policy**: Define backup schedules, retention, and geo‑redundancy in one place  

##### Summary

| Feature                        | Azure SQL Database           | Azure SQL Managed Instance       | SQL Server On-Premises         |
|-------------------------------|------------------------------|----------------------------------|-------------------------------|
| Manual BACKUP/RESTORE         | No                           | Yes                              | Yes                           |
| PITR Support                  | Yes (7-35 days)              | Yes                              | No (must restore from logs)   |
| LTR Support                   | Yes                          | Yes                              | Manual/archive-based          |
| Geo-Restore                   | Yes                          | Yes                              | No                            |
| Backup File Access            | No                           | Yes (via Azure Blob)             | Yes                           |
| Custom Backup Strategy        | No (platform-managed)        | Yes                              | Yes                           |
| Backup Automation             | Platform-managed             | Platform-managed or manual       | Must be configured manually   |
| Restore Granularity           | Point-in-time                | Point-in-time and custom         | Fully customizable            |

------------------------- -------------------------
------------------------- -------------------------

### Exercise

> This exercise assumes an Azure subscription with SQL Server and sample data

#### Available Backups

Open the Azure Portal and navigate to SQL Server >> **Data Management** >> **Backups**, then select the **Available Backups** tab.

This tab lists each SQL Database on the server and displays metadata about backups that Azure has automatically created and manages. These backups support **Point-in-Time Restore (PITR)** and, if configured, **Long-Term Retention (LTR)**—you do not configure or initiate the backups yourself.

The table includes the following columns:

- **Database**: Name of the database
- **Earliest PITR restore point (UTC)**: The earliest timestamp you can restore to, based on the current PITR retention window and Azure’s internal backup schedule
- **Available LTR backups**: Number of full backups available under an LTR policy. If this shows `None`, no LTR policy is currently applied or no backups have aged into the retention window yet
- **Last LTR backup time (UTC)**: Timestamp of the most recent long-term backup, if one exists
- **Action**: Link launches the restore experience where you can choose to restore from PITR or LTR, depending on availability

------------------------- -------------------------

#### Retention Policies

Open the Azure Portal and navigate to SQL Server >> **Data Management** >> **Backups**, then select the **Retention Policies** tab.

This tab allows you to configure and manage how long Azure retains backups for each SQL Database—both for **short-term Point-in-Time Restore (PITR)** and **Long-Term Retention (LTR)** scenarios. The table reflects current retention settings per database.

The table includes the following columns:

- **Database**: Name of the database (e.g., `cnbss`).
- **PITR**: The short-term restore retention window, currently set to 7 days. This determines how far back you can perform a point-in-time restore (range: 1–35 days).
- **Diff. backup frequency**: Shows how often Azure takes differential backups (12 hours). This is managed by the platform and cannot be changed.
- **Weekly / Monthly / Yearly LTR**: These indicate whether LTR policies are configured to retain full backups on a recurring basis. A value of `--` means no policy is active.
- **Pricing tier**: The selected service tier for the database, which may affect available retention options.

Above the grid, you’ll find buttons to manage policies:
- **Configure policies**: Opens a panel to set weekly, monthly, and yearly retention schedules for LTR.
- **Remove LTR settings**: Clears any active long-term retention configuration.

Use this tab to explicitly control your backup retention strategy—whether keeping backups just for recovery within a few days, or retaining weekly/monthly/yearly backups for years to meet audit, compliance, or archival requirements.

------------------------- -------------------------

#### Point-in-Time Restore (PITR)

This exercise shows how to use the Azure Portal alone to restore an Azure SQL Database to a previous state. You’ll make a change via the portal’s Query editor, then roll the database back to just before that change.

* Open the Azure Portal, navigate to your SQL Server, and select **Query editor (preview)** under Settings

* Log in and then execute the following T-SQL:
  ```sql
  CREATE TABLE dbo.PITR_Test (Id INT PRIMARY KEY, Name NVARCHAR(100));
  INSERT INTO dbo.PITR_Test VALUES (1, 'This row should disappear after restore');
  ```

* Execute the following T-SQL to confirm change:
  ```sql
  SELECT * FROM dbo.PITR_Test;
  ```

* Wait ~5 minutes for a transaction log backup to occur
  > Note: There is no easy evidence of successful transaction log backup... you can validate in Monitoring
  
* Navigate to **Data Management** >> **Backups**, then the **Available Backups** tab

* Click **Restore** beside your database, select source "**Point-in-Time**", pick a "Restore point..." timestamp just before your insert, and enter a name for the new database {e.g., `{prefix}ss_PITR`}
  
* Click "Review + create", then "Create", and wait for provisioning to complete

Once the restored database is online, open **Query editor (preview)** for it and execute the following T-SQL:
```sql
SELECT * FROM dbo.PITR_Test;
```

You should see `Failed to execute query. Error: Invalid object name 'dbo.PITR_Test'`, confirming the database has been returned to its pre‑change state.

------------------------- -------------------------
------------------------- -------------------------

## Quiz

1. What is the key advantage of Azure SQL Database’s built‑in backup system?  
   A) It requires manual intervention for every backup  
   B) It automates backup creation and storage, ensuring geo‑redundancy  
   C) It only supports full backups  
   D) It does not allow point‑in‑time recovery  

2. How does Azure ensure that backups remain available if the primary region fails?  
   A) By storing backups only in the same region  
   B) Through zone‑redundant storage  
   C) Through geo‑redundant storage  
   D) By disabling backup automation  

3. Which restore option allows you to recover an Azure SQL Database to a specific moment in time?  
   A) Differential Restore  
   B) Point‑in‑Time Restore  
   C) Manual Restore  
   D) COPY_ONLY Restore  

4. Which portal blade do you use to configure long‑term retention for weekly, monthly, and yearly backups?  
   A) Long‑term retention  
   B) Restore  
   C) Backups  
   D) Retention Policies  

5. What is the maximum retention period you can configure with point‑in‑time restore?  
   A) 7 days  
   B) 35 days  
   C) 90 days  
   D) 180 days  

6. What is the maximum retention period supported by long‑term retention policies?  
   A) 35 days  
   B) 1 year  
   C) 5 years  
   D) 10 years  

7. By default, Azure SQL Database automated backups use which type of storage redundancy?  
   A) Locally‑redundant storage (LRS)  
   B) Zone‑redundant storage (ZRS)  
   C) Geo‑redundant storage (GRS)  
   D) Read‑access geo‑redundant storage (RA‑GRS)  

8. How often does Azure SQL Database automatically perform differential backups?  
   A) Every 5 minutes  
   B) Every hour  
   C) Every 12 hours  
   D) Every day  

9. What is the default retention period for automated backups when you provision a new Azure SQL Database?  
   A) 7 days  
   B) 14 days  
   C) 30 days  
   D) 35 days  

10. Which restore option would you use to recover a database in a different Azure region after a regional outage?  
    A) Point‑in‑Time Restore  
    B) Long‑Term Restore  
    C) Differential Restore  
    D) Geo‑Restore  

------------------------- -------------------------

#### Answers

1. **B** – It automates backup creation and storage, ensuring geo‑redundancy  
   *Azure handles full, differential, and log backups automatically and stores them geo‑redundantly.*

2. **C** – Through geo‑redundant storage  
   *Backups are stored in geo‑redundant storage to survive regional failures.*

3. **B** – Point‑in‑Time Restore  
   *This option lets you restore the database to any moment within the retention window.*

4. **A** – Long‑term retention  
   *The Long‑term retention blade configures weekly, monthly, and yearly backup retention.*

5. **B** – 35 days  
   *Point‑in‑time restore supports a maximum of 35 days of retention.*

6. **D** – 10 years  
   *Long‑term retention policies can keep backups for up to 10 years.*

7. **D** – Read‑access geo‑redundant storage (RA‑GRS)  
   *Automated backups use RA‑GRS to ensure both redundancy and read access to backup blobs.*

8. **C** – Every 12 hours  
   *Azure takes differential backups twice a day to capture changes since the last full backup.*

9. **A** – 7 days  
   *By default, new databases have a 7‑day backup retention period.*

10. **D** – Geo‑Restore  
    *Geo‑Restore uses geo‑replicated backups to recover a database in a different region.*