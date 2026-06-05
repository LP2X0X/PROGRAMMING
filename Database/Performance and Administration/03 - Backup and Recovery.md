---
tags: [backup, recovery, administration]
---

- Backups are your last line of defense against data loss. Hardware fails, humans make mistakes, ransomware encrypts files, and natural disasters destroy data centers. If you cannot restore your database, nothing else matters — not your indexes, not your query tuning, not your carefully normalized schema. **The database only exists if you can get it back.**
- **Prerequisite:** [[02 - Transactions and Concurrency]]. Understanding transactions and the transaction log is essential because transaction log backups are the foundation of point-in-time recovery.

---

## Why Backups Matter

- Data loss scenarios are not hypothetical. They happen:
  - **Hardware failure** — disk dies, RAID controller fails, SAN corruption.
  - **Human error** — someone runs `DELETE FROM customers` without a `WHERE` clause, or drops the wrong table in production.
  - **Software bugs** — an application bug corrupts data silently over weeks before anyone notices.
  - **Malicious attacks** — ransomware encrypts database files, disgruntled employee deletes data.
  - **Natural disasters** — fire, flood, power surge destroys the server room.

- Each scenario requires a different recovery approach. A good backup strategy handles all of them.

```ad-important
A backup you haven't tested restoring is **not** a backup. It is a file that *might* contain your data. Test your restore process regularly — at minimum, once per quarter. Automate a test restore to a staging environment if possible.
```

---

## Types of Backups

| Backup Type | What It Captures | Size | Speed | Frequency | Restore Complexity |
| --- | --- | --- | --- | --- | --- |
| **Full** | The entire database — every data page, every object | Large | Slowest | Weekly (or nightly for smaller databases) | Simplest — restore one file |
| **Differential** | Only the data pages that changed since the last **full** backup | Medium (grows over time) | Medium | Daily | Restore full + latest differential |
| **Transaction Log** | All transaction log records since the last log backup | Small | Fastest | Every 15-60 minutes | Restore full + differential + all log backups in sequence |

### Full Backup

- A complete copy of the database at a specific point in time. This is the baseline that all other backup types depend on.

```sql
-- SQL Server: full backup
BACKUP DATABASE MyDb
TO DISK = 'C:\Backups\MyDb_Full_20260605.bak'
WITH COMPRESSION, CHECKSUM, STATS = 10;
```

- `COMPRESSION` — reduces backup file size significantly (often 5-10x). Uses more CPU during backup but saves disk space and I/O.
- `CHECKSUM` — calculates a checksum over the data pages and stores it in the backup. Detects corruption during restore.
- `STATS = 10` — prints progress every 10%.

```ad-note
A full backup does **not** truncate the transaction log. This is a common misconception. Only a transaction log backup truncates the log (marks inactive portions for reuse). Taking a full backup does not affect your log backup chain.
```

### Differential Backup

- Captures only the **extents** (64KB groups of 8 pages) that have changed since the last full backup. Much smaller and faster than a full backup.

```sql
BACKUP DATABASE MyDb
TO DISK = 'C:\Backups\MyDb_Diff_20260605.bak'
WITH DIFFERENTIAL, COMPRESSION, CHECKSUM;
```

- Key behavior: a differential always measures changes from the last **full** backup, not the last differential. This means:
  - Monday's differential = changes since Sunday full
  - Tuesday's differential = changes since Sunday full (includes Monday's changes too)
  - Wednesday's differential = changes since Sunday full (includes Mon + Tue)
  - Differentials **grow over time** until the next full backup resets the baseline.

```ad-warning
Because differentials are cumulative from the last full, you only need the **most recent** differential when restoring — not all of them. This is different from transaction log backups, where you need every single one in the chain.
```

### Transaction Log Backup

- Captures all transaction log records written since the last log backup. This is the finest-grained backup type and enables **point-in-time recovery** — the ability to restore to any specific moment.

```sql
BACKUP LOG MyDb
TO DISK = 'C:\Backups\MyDb_Log_20260605_1430.trn'
WITH COMPRESSION, CHECKSUM;
```

- After a log backup completes, the inactive portion of the transaction log is marked for reuse. This is how you prevent the log file from growing indefinitely.
- **Log chain:** each log backup picks up exactly where the previous one left off. They form an unbroken sequence. If you lose one log backup in the middle, you can only restore up to the backup before the missing one.

```ad-important
**Never** delete or lose a transaction log backup file from the chain. If you have Full (Sunday) + Log 1 + Log 2 + Log 3 + Log 4 and you lose Log 2, you can only restore up to the end of Log 1. Logs 3 and 4 become useless because they expect Log 2's end state as their starting point.
```

---

## Recovery Models (SQL Server)

- The **recovery model** determines what kind of backups you can take and what recovery options are available. It controls how the transaction log is managed.

| Recovery Model | Log Truncation | Log Backups? | Point-in-Time Recovery? | Best For |
| --- | --- | --- | --- | --- |
| **Simple** | Automatic (at checkpoint) | Not possible | No — can only restore to the last backup | Development, testing, databases where data loss is acceptable |
| **Full** | Only after log backup | Required (or the log grows forever) | Yes — restore to any point in time | Production databases, any system where data loss is unacceptable |
| **Bulk-Logged** | Only after log backup | Required | Limited — not through bulk operations | Short periods during bulk import operations, then switch back to Full |

### Setting the Recovery Model

```sql
-- Check current recovery model:
SELECT name, recovery_model_desc
FROM sys.databases
WHERE name = 'MyDb';

-- Change recovery model:
ALTER DATABASE MyDb SET RECOVERY FULL;
```

```ad-warning
If your database is in **Full** recovery model and you are NOT taking transaction log backups, the log file will grow until the disk is full and the database stops accepting writes. This is one of the most common DBA emergencies. Either take log backups regularly or switch to Simple recovery model.
```

### Simple vs Full — When to Use Which

```
Simple Recovery Model:
├── Development and test environments
├── Data warehouses that can be fully reloaded from source
├── Databases where losing data since the last backup is acceptable
└── Your "I can rebuild this from scratch" databases

Full Recovery Model:
├── Production OLTP databases
├── Any database with user data you cannot recreate
├── Databases with regulatory or compliance requirements
├── Any system where "we lost the last 15 minutes of data" is unacceptable
└── Financial, healthcare, e-commerce — anything business-critical
```

---

## Backup Strategy — Putting It Together

- A backup strategy combines all three backup types to balance data protection, storage cost, and recovery speed.

### Example Strategy: RPO = 15 Minutes

- **RPO (Recovery Point Objective):** the maximum acceptable data loss, measured in time. An RPO of 15 minutes means you can tolerate losing at most 15 minutes of data.
- **RTO (Recovery Time Objective):** the maximum acceptable downtime. How quickly you need the database back online.

| When | What | Retention |
| --- | --- | --- |
| Sunday 2:00 AM | Full backup | 4 weeks |
| Mon-Sat 2:00 AM | Differential backup | 1 week |
| Every 15 minutes, 24/7 | Transaction log backup | 1 week |

**Worst-case data loss:** 15 minutes (the interval between log backups).

**Restore scenario** — disaster at Wednesday 3:47 PM:
1. Restore Sunday's full backup (`WITH NORECOVERY`)
2. Restore Wednesday's differential (`WITH NORECOVERY`)
3. Restore every log backup from Wednesday after the differential: 2:15 AM, 2:30 AM, ..., 3:30 PM, 3:45 PM (`WITH RECOVERY` on the last one)
4. Database is online with data up to 3:45 PM — 2 minutes of data lost.

### Smaller Databases — Simpler Strategy

| When | What |
| --- | --- |
| Nightly at 2:00 AM | Full backup |
| Every 30 minutes | Transaction log backup |

- Simpler to manage. Restore is just: full + all logs since the full.

---

## Backup Syntax — Complete Reference

### Full Backup

```sql
-- Basic full backup:
BACKUP DATABASE MyDb
TO DISK = 'C:\Backups\MyDb_Full.bak';

-- Production-grade full backup:
BACKUP DATABASE MyDb
TO DISK = 'C:\Backups\MyDb_Full_20260605.bak'
WITH
    COMPRESSION,          -- compress the backup (saves disk space)
    CHECKSUM,             -- detect corruption
    INIT,                 -- overwrite the file (don't append)
    NAME = 'MyDb Full Backup',
    DESCRIPTION = 'Weekly full backup',
    STATS = 10;           -- show progress every 10%
```

### Differential Backup

```sql
BACKUP DATABASE MyDb
TO DISK = 'C:\Backups\MyDb_Diff_20260605.bak'
WITH DIFFERENTIAL, COMPRESSION, CHECKSUM, INIT;
```

### Transaction Log Backup

```sql
BACKUP LOG MyDb
TO DISK = 'C:\Backups\MyDb_Log_20260605_1430.trn'
WITH COMPRESSION, CHECKSUM;
```

### Tail-Log Backup

- A **tail-log backup** captures any log records that haven't been backed up yet — the "tail" of the log. This is critical when the database is damaged and you need to capture the very latest data before restoring.

```sql
-- Take a tail-log backup before restoring (captures latest uncommitted log):
BACKUP LOG MyDb
TO DISK = 'C:\Backups\MyDb_TailLog.trn'
WITH NORECOVERY;  -- puts the database in RESTORING state
```

```ad-important
Always take a tail-log backup before restoring a database (unless the data files are destroyed and the log is inaccessible). Without it, you lose all data since the last log backup. The `NORECOVERY` option puts the database into a restoring state so you can proceed with the restore immediately.
```

### Backup to Multiple Files (Striped Backup)

```sql
-- Split the backup across multiple files for faster I/O:
BACKUP DATABASE MyDb
TO DISK = 'C:\Backups\MyDb_1.bak',
   DISK = 'C:\Backups\MyDb_2.bak',
   DISK = 'C:\Backups\MyDb_3.bak'
WITH COMPRESSION, CHECKSUM;
```

- Useful for very large databases. Each file is written in parallel, significantly reducing backup time.

---

## Restore Syntax — Complete Reference

### Restore a Full Backup

```sql
-- Simple restore (database comes online immediately):
RESTORE DATABASE MyDb
FROM DISK = 'C:\Backups\MyDb_Full.bak'
WITH RECOVERY;

-- RECOVERY means: bring the database online after restoring.
-- This is the default if you don't specify.
```

### Restore Full + Differential + Logs

```sql
-- Step 1: Restore the full backup (don't bring online yet)
RESTORE DATABASE MyDb
FROM DISK = 'C:\Backups\MyDb_Full_Sunday.bak'
WITH NORECOVERY;  -- keeps database in RESTORING state

-- Step 2: Restore the latest differential
RESTORE DATABASE MyDb
FROM DISK = 'C:\Backups\MyDb_Diff_Wednesday.bak'
WITH NORECOVERY;

-- Step 3: Restore each transaction log in order
RESTORE LOG MyDb FROM DISK = 'C:\Backups\MyDb_Log_1430.trn' WITH NORECOVERY;
RESTORE LOG MyDb FROM DISK = 'C:\Backups\MyDb_Log_1445.trn' WITH NORECOVERY;
RESTORE LOG MyDb FROM DISK = 'C:\Backups\MyDb_Log_1500.trn' WITH NORECOVERY;

-- Step 4: Bring the database online with the LAST restore
RESTORE LOG MyDb FROM DISK = 'C:\Backups\MyDb_Log_1515.trn' WITH RECOVERY;
-- Database is now online
```

- `NORECOVERY` — keep the database in "restoring" state so you can apply more backups.
- `RECOVERY` — bring the database online. Use this on the very last restore step.

```ad-warning
Once you use `WITH RECOVERY`, the restore sequence is finalized. You cannot apply any more backup files after that. If you accidentally bring the database online too early, you must start the entire restore from the beginning.
```

### Point-in-Time Recovery

- Restore to an exact moment — for example, the instant before someone ran a bad DELETE:

```sql
-- Step 1: Restore full backup
RESTORE DATABASE MyDb
FROM DISK = 'C:\Backups\MyDb_Full.bak'
WITH NORECOVERY;

-- Step 2: Restore differential (if you have one)
RESTORE DATABASE MyDb
FROM DISK = 'C:\Backups\MyDb_Diff.bak'
WITH NORECOVERY;

-- Step 3: Restore log backups up to the target time
RESTORE LOG MyDb
FROM DISK = 'C:\Backups\MyDb_Log_1.trn'
WITH NORECOVERY;

-- Step 4: Restore the final log with STOPAT
RESTORE LOG MyDb
FROM DISK = 'C:\Backups\MyDb_Log_2.trn'
WITH STOPAT = '2026-06-05 14:29:59', RECOVERY;
-- Restores all transactions committed before 14:30:00
-- The bad DELETE at 14:30:00 is excluded
```

```ad-note
`STOPAT` stops the restore at the specified time. Any transaction that committed *after* that time is not applied. This is incredibly powerful for surgical recovery — you can undo one bad operation without losing everything after it.
```

### Restore to a Different Database (Copy)

```sql
-- Restore as a different database name (useful for testing or data recovery):
RESTORE DATABASE MyDb_Copy
FROM DISK = 'C:\Backups\MyDb_Full.bak'
WITH
    MOVE 'MyDb' TO 'C:\Data\MyDb_Copy.mdf',
    MOVE 'MyDb_log' TO 'C:\Data\MyDb_Copy_log.ldf',
    RECOVERY;
```

- `MOVE` redirects the data and log files to a different location. Without `MOVE`, the restore would try to overwrite the original database's files.

---

## Verifying Backups

### RESTORE VERIFYONLY

```sql
-- Verify the backup file is readable and not corrupt:
RESTORE VERIFYONLY
FROM DISK = 'C:\Backups\MyDb_Full.bak'
WITH CHECKSUM;
```

- This checks the backup's internal structure and checksums but does **not** actually restore data. It's fast and catches most corruption issues.

```ad-warning
`RESTORE VERIFYONLY` is a good first check, but it does not guarantee a successful restore. The only way to be 100% certain a backup works is to actually restore it to a test server. Automate this process.
```

### Check for Backup History

```sql
-- See all backups taken for a database:
SELECT 
    bs.database_name,
    bs.type AS backup_type,    -- D=Full, I=Differential, L=Log
    bs.backup_start_date,
    bs.backup_finish_date,
    DATEDIFF(SECOND, bs.backup_start_date, bs.backup_finish_date) AS duration_seconds,
    bs.backup_size / 1024 / 1024 AS size_mb,
    bs.compressed_backup_size / 1024 / 1024 AS compressed_mb,
    bmf.physical_device_name AS backup_file
FROM msdb.dbo.backupset bs
JOIN msdb.dbo.backupmediafamily bmf ON bs.media_set_id = bmf.media_set_id
WHERE bs.database_name = 'MyDb'
ORDER BY bs.backup_start_date DESC;
```

---

## Backup Storage Best Practices

1. **Store backups on a different physical disk** than the database. If the disk fails, you don't want to lose both the database and its backups.
2. **Store copies off-site** — a separate server, network share, or cloud storage (Azure Blob Storage, AWS S3). This protects against site-level disasters.
3. **Use the 3-2-1 rule:**
   - **3** copies of your data (original + 2 backups)
   - **2** different storage media (local disk + cloud, or disk + tape)
   - **1** copy off-site
4. **Encrypt backups** — backup files contain all your data. If someone steals a backup file, they have everything:

```sql
-- Create a backup encryption certificate:
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'MasterKeyP@ss!';
CREATE CERTIFICATE BackupCert WITH SUBJECT = 'Backup Encryption Certificate';

-- Encrypted backup:
BACKUP DATABASE MyDb
TO DISK = 'C:\Backups\MyDb_Encrypted.bak'
WITH COMPRESSION,
     ENCRYPTION (ALGORITHM = AES_256, SERVER CERTIFICATE = BackupCert);
```

```ad-important
Store backups on a **different machine or disk** than the database. A backup sitting on the same drive as the database is useless when that drive fails. Remote network shares, dedicated backup servers, and cloud storage are all valid options. The key is physical separation.
```

5. **Automate everything** — use SQL Server Agent jobs, maintenance plans, or third-party tools (Ola Hallengren's scripts are widely used and free) to automate backup schedules.

---

## Common Backup Mistakes

| Mistake | Why It's Dangerous | Fix |
| --- | --- | --- |
| Never testing restores | You discover the backup is corrupt when you desperately need it | Schedule quarterly test restores |
| Backups on the same disk as the database | Disk failure = both database AND backups are gone | Store on separate disk/server/cloud |
| Full recovery model with no log backups | Transaction log grows until the disk is full | Take log backups every 15-60 minutes, or switch to Simple |
| Deleting old log backups before verifying the chain | Breaks point-in-time recovery capability | Keep log backups until you've verified the chain and taken a new full |
| No backup encryption | Stolen backup file = full data breach | Encrypt backups, secure the encryption certificate |
| Backing up system databases too infrequently | Losing `msdb` means losing all job schedules, backup history, and SSIS packages | Back up `master`, `msdb`, and `model` after any changes |

---

**Next:** [[04 - Security Basics]]
