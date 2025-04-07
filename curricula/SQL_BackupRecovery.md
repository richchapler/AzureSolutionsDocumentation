# SQL: Backup and Recovery

## Use Case

The database administration team at a mid‑sized organization must ensure that critical data is protected against corruption, accidental deletion, or system failures. Key requirements:

* Data Protection: Guarding against data loss with reliable backup strategies
* Rapid Recovery: Minimizing downtime and data loss by restoring data efficiently
* Compliance & Security: Meeting regulatory and internal standards by securing backup files and validating restore procedures
* Operational Continuity: Ensuring that data can be recovered quickly during planned maintenance or unexpected outages

------------------------- -------------------------
------------------------- -------------------------
------------------------- -------------------------

Below is the complete, updated version of the **On‑Prem >> Fundamentals** section with our suggested changes and consistent formatting:

---

## On‑Prem

### Fundamentals

#### Backup

##### Types

* **Full Backup:**  
  A full backup is a complete snapshot of the entire database at a specific moment—a “photo” of your database. This backup serves as the foundational baseline for all subsequent backup activities.

* **Differential Backup:**  
  Differential backups capture only the changes made since the last full backup. They act like incremental snapshots that record what’s different from the baseline. Over time, they provide a cumulative record of modifications while reducing the amount of data to be backed up compared to a full backup.

* **Transaction Log Backup:**  
  For databases using a full or bulk‑logged recovery model, transaction log backups record every transaction that occurs after the last backup. This continuous log enables administrators to capture near real‑time changes and supports point‑in‑time recovery.

* **Ad Hoc (COPY_ONLY) Backups:**  
  COPY_ONLY backups provide an independent snapshot without disturbing the regular backup sequence. They’re ideal for ad‑hoc scenarios—such as before major changes or testing—ensuring that your scheduled backup strategy remains intact.

##### Designing a Backup Plan

A robust backup plan starts with understanding the criticality of your data and how frequently it changes. Key elements include:

- **Frequency:**  
  Decide how often to perform each type of backup. Full backups, which are larger, are scheduled less frequently, while differential and transaction log backups are run more often. The frequency should align with business needs and ensure that any potential data loss falls within your predetermined Recovery Point Objective (RPO).

- **Storage Locations:**  
  Choose between on‑site storage for rapid recovery and off‑site or cloud‑based storage for disaster recovery. Secure, redundant storage solutions—ideally with encryption—minimize the risk of data loss due to localized failures.

- **Automation:**  
  Implement automated scheduling and monitoring for backup operations. Automation minimizes human error by consistently executing backup jobs and provides real‑time alerts if issues arise, ensuring backups complete successfully and are available when needed.

#### Restore

##### Restore and Recovery Procedures

Restoring a database reverses the backup process and consists of these steps:

- **Full Backup Restoration:**  
  Begin by restoring the most recent full backup to reestablish the complete baseline.

- **Differential Backup Application:**  
  If available, apply the latest differential backup to incorporate changes made since the full backup.

- **Transaction Log Restoration:**  
  Finally, restore transaction log backups sequentially to replay individual transactions. For point‑in‑time recovery, halt the restoration at the designated moment to revert to a known good state.

Regular testing of these procedures is essential to ensure rapid and reliable recovery in an actual failure scenario.

##### Best Practices

Effective restoration is as critical as the backup process. Best practices include:

- **Validation:**  
  Regularly test restore procedures using representative sample data to confirm that backups are complete and usable. These exercises help uncover potential issues before an actual recovery is needed.

- **Documentation:**  
  Maintain detailed records of backup jobs—including time stamps, storage locations, and restore outcomes—to aid troubleshooting and ensure compliance with regulatory standards.

- **Maintenance:**  
  Continuously monitor backup file integrity and manage storage resources by performing periodic checks for file corruption, rotating backup media as necessary, and updating procedures to accommodate changing data volumes.

#### Key Considerations

A successful backup and recovery strategy depends on:

- **Data Integrity:**  
  Ensuring that each backup builds correctly on the previous one so that the restore process maintains a consistent database state.

- **Thorough Documentation and Testing:**  
  Keeping detailed records and regularly simulating restore procedures to verify that recovery will be seamless in the event of an actual failure.

- **Automation and Monitoring:**  
  Using automated scheduling and proactive monitoring to minimize human error and quickly detect any issues in the backup process.

#### Key Metrics

* **Recovery Time Objective (RTO):**  
  RTO represents the maximum acceptable downtime following a disruption. It is determined by measuring the time from failure detection, through the recovery process, until full restoration. This metric helps guide investments in technology and process improvements to minimize downtime.

* **Recovery Point Objective (RPO):**  
  RPO indicates the maximum amount of data (expressed in time) that an organization can afford to lose. It is calculated based on the frequency of backup operations and the rate of data change. For instance, if transaction logs are backed up every 15 minutes, the RPO would be approximately 15 minutes.

#### Advanced Concepts

Enhance your backup strategy with these advanced measures:

**Backup Optimization:**  
- **Compression:**  
  Reduces the size of backup files, saving storage space and speeding up data transfers.
- **Encryption:**  
  Secures backup files to ensure sensitive data remains protected even if the storage media are compromised.

**Verification and Maintenance:**  
- **Integrity Checks:**  
  Regularly verify backup validity (for example, using methods like RESTORE VERIFYONLY).
- **Routine Maintenance:**  
  Schedule regular cleanups of old backups to manage storage efficiently.

**Disaster Recovery Planning:**  
- **Off‑Site and Cloud Backups:**  
  Combine local backups with geo‑redundant or cloud‑based storage to protect against site‑wide disasters.
- **Automated Monitoring:**  
  Employ tools to alert administrators to backup failures or anomalies, ensuring prompt remediation.

#### Security Considerations

* **Backup File Security:**  
  - **Encryption:** Always encrypt backup files to safeguard sensitive data.  
  - **Access Controls:** Limit access to backup storage locations to authorized personnel only.

* **Compliance Requirements:**  
  - **Audit Trails:** Maintain logs of all backup and restore operations.  
  - **Retention Policies:** Ensure backup retention schedules align with industry and regulatory standards.

#### Monitoring Considerations

* **Automated Monitoring:**  
  - **Alerts:** Configure tools (such as SQL Server Agent jobs) to notify administrators of backup failures.  
  - **Dashboard Views:** Use monitoring dashboards and DMVs to track backup status and job histories.

* **Regular Reviews:**  
  - **Health Checks:** Periodically review backup logs and assess storage capacity.  
  - **Testing:** Schedule routine restore tests to confirm backup reliability and readiness.

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

  * **Transaction Logging:** In the Full Recovery Model, every transaction is fully logged. This detailed logging allows you to perform transaction log backups, which are critical for point‑in‑time recovery. Without a proper recovery model, you might be unable to recover all the transactions that occurred between backups.
  * **Data Protection:** With full logging, you ensure that every data change is recorded. This means that if a failure occurs, you can restore the database to the exact moment before the failure, minimizing data loss.
  * **Recovery Objectives:** The recovery model directly impacts your Recovery Point Objective (RPO) — a key metric that defines how much data loss is acceptable. A Full Recovery Model supports a low RPO because you can recover the most recent transactions.

  Execute the following T-SQL:
  ```sql
  USE master;
  GO
  ALTER DATABASE AdventureWorks2022 SET RECOVERY FULL;
  GO
  ```

------------------------- -------------------------

#### Full Backup

Performing a full backup creates a complete snapshot of the entire `AdventureWorks2022` database at a specific moment. This backup serves as the baseline for your backup chain.

  * **Baseline for Recovery:** The full backup is the foundational component in any backup strategy. When you restore your database, **you always start with the most recent full backup**. Without it, you wouldn’t have a complete reference of your database’s state.
  * **Establishing a Recovery Point:** The full backup defines a clear recovery point. If a failure occurs, you will rely on this backup (and subsequent backups) to bring your database back to a known, consistent state.
  * **Efficiency in Subsequent Backups:** Once the full backup is in place, subsequent differential backups can capture only the changes made since that full backup, reducing the amount of data that must be stored and later restored.

  Execute the following T-SQL:
```sql
BACKUP DATABASE AdventureWorks2022 
TO DISK = 'C:\Temp\AdventureWorks_full.bak' 
WITH INIT;
GO
```

Example Output:
```text
Processed 25376 pages for database 'AdventureWorks2022', file 'AdventureWorks2022' on file 1.
Processed 2 pages for database 'AdventureWorks2022', file 'AdventureWorks2022_log' on file 1.
BACKUP DATABASE successfully processed 25378 pages in 0.439 seconds (451.621 MB/sec).

Completion time: 2025-04-07T14:40:13.1775263-07:00
```

------------------------- -------------------------

#### Verify Backup

After executing the full backup, check that the backup file (`AdventureWorks_full.bak`) was created in the designated folder. This verification is important to ensure that your backup process is working correctly and that you have a reliable starting point for recovery.












------------------------- -------------------------
------------------------- -------------------------

## Quiz

1. What is the primary purpose of a full backup in SQL Server?  
   A) To capture only changes since the last backup  
   B) To provide a complete copy of the database at a point in time  
   C) To reduce the size of subsequent backups  
   D) To encrypt data at rest  

2. Which backup type is best suited for ad‑hoc backups without disturbing the backup sequence?  
   A) Differential Backup  
   B) Transaction Log Backup  
   C) COPY_ONLY Backup  
   D) Full Backup  

3. What does a differential backup capture?  
   A) All transactions since the last log backup  
   B) All changes made since the last full backup  
   C) Only system database changes  
   D) The entire database regardless of changes  

4. Which backup method enables point‑in‑time recovery?  
   A) Full Backup alone  
   B) Differential Backup alone  
   C) Transaction Log Backup  
   D) COPY_ONLY Backup  

5. In a restore sequence, why is the NORECOVERY option used?  
   A) To finalize the restore immediately  
   B) To allow additional backups to be applied  
   C) To encrypt the restored database  
   D) To verify the integrity of the backup file  

6. Which recovery model must be set to use transaction log backups effectively?  
   A) Simple Recovery Model  
   B) Full Recovery Model  
   C) Bulk‑Logged Recovery Model  
   D) Minimal Recovery Model  

7. What is the benefit of backup compression?  
   A) Increased backup file size  
   B) Reduced storage requirements and faster transfer  
   C) Enhanced database performance during backups  
   D) Automatic encryption of backup files  

8. Why is it important to test restore procedures regularly?  
   A) To decrease the frequency of backups  
   B) To verify backup integrity and ensure quick recovery during a failure  
   C) To avoid using differential backups  
   D) To ensure the full recovery model is enabled  

9. Which command is used to verify that a backup file is readable and valid?  
   A) RESTORE DATABASE  
   B) BACKUP VERIFY  
   C) RESTORE VERIFYONLY  
   D) CHECK BACKUP  

10. What is a critical security measure for backup files?  
    A) Leaving them unencrypted for faster access  
    B) Storing them on the same server as the primary database  
    C) Encrypting backup files and restricting access  
    D) Using only full backups and skipping differential backups  

------------------------- -------------------------
------------------------- -------------------------
------------------------- -------------------------

## Azure

While on‑premises backup strategies require manual configuration, Azure SQL Database and Managed Instance come with built‑in backup capabilities that simplify these tasks.

### Fundamentals

* Automated Backups:  
  * Azure automatically creates full, differential, and log backups without user intervention.
  * Backups are stored in geo‑redundant storage, ensuring high durability.

* Restoration Options:  
  * Point‑in‑Time Restore: Quickly restore a database to a specific moment.
  * Long‑Term Retention: Maintain backups for compliance and archival purposes.

------------------------- -------------------------

### Exercise

#### Using the Azure Portal

1. Navigate to the Azure SQL Server that hosts your database.
2. Configure Backup Retention Policies:  
   * Set the desired retention period for full, differential, and log backups.
3. Initiate a Restore:  
   * From the Azure Portal, choose a database and select “Restore” to initiate a point‑in‑time restore.
4. Validate the Restored Database:  
   * Use the query editor to ensure that the data is consistent and recent.

------------------------- -------------------------

### Quiz

1. What is the key advantage of Azure SQL Database’s built‑in backup system?  
   A) It requires manual intervention for every backup  
   B) It automates backup creation and storage, ensuring geo‑redundancy  
   C) It only supports full backups  
   D) It does not allow point‑in‑time recovery  

2. How does Azure ensure that backups are durable in the event of a regional outage?  
   A) By storing backups only locally  
   B) Through geo‑redundant storage  
   C) By disabling backup automation  
   D) Through manual backup processes  

3. Which restore option allows you to recover an Azure SQL Database to a specific moment in time?  
   A) Manual Restore  
   B) Point‑in‑Time Restore  
   C) Differential Restore  
   D) COPY_ONLY Restore  

4. Why is it important to configure backup retention policies in Azure SQL Database?  
   A) To ensure that backups are available for compliance and disaster recovery  
   B) To decrease storage costs by deleting backups immediately  
   C) To disable automatic backups  
   D) To switch the recovery model to simple