## Lesson 8 - Using Recovery Manager Demos and Practices (in which RMAN politely asks if you are sure, and then deletes your obsolete backups)

And look, this is the practical side of RMAN: connect correctly, run backups, query backup metadata, inspect persistent settings, and verify your defaults before you trust them in production.

By the end of this lesson, you should be able to:

- Connect to RMAN and a target database with `SYSBACKUP`
- Run backup workflows for database-only and database-plus-archivelog
- Use SQL statements inside RMAN where supported
- Inspect and manage persistent RMAN configuration
- Complete practices for default backup destination, date-time display, and key RMAN settings

---

## 1. Demo 1: Connect to RMAN and Run Backups

### Step 1: Confirm environment

Make sure your database environment is sourced first:

```bash
. oraenv
```

Example target in this course flow: `ORCL` or `ORCLCDB` depending on the practice.

### Step 2: Connect to RMAN with `SYSBACKUP`

```text
rman target " / as sysbackup"
```

The quoted literal format is important when connecting this way.

### Step 3: Run basic backup

```rman
BACKUP DATABASE;
```

Then inspect what RMAN created:

```rman
LIST BACKUP;
```

You can map backup pieces to data files/tablespaces from this output.

### Step 4: Backup database plus archive logs

```rman
BACKUP DATABASE PLUS ARCHIVELOG;
```

RMAN will show archive log backup sets and related pieces in output.

### Step 5: Remove obsolete backups

```rman
DELETE OBSOLETE;
```

RMAN will list candidates based on retention policy and prompt for confirmation.

Example response:

```text
Do you really want to delete the above objects (enter YES or NO)? YES
```

### Step 6: Exit

```rman
EXIT;
```

---

## 2. Demo 2: Use SQL Inside RMAN

You can execute certain SQL from RMAN, especially backup/recovery-relevant checks.

Example:

```rman
SQL "SELECT name, dbid, log_mode FROM v$database";
```

This gives you the same core database identity/mode information you would expect from SQL*Plus.

Not every SQL statement is allowed in RMAN. Keep expectations focused on operational and recovery-related queries.

Then exit RMAN:

```rman
EXIT;
```

---

## 3. Demo 3: Manage Persistent RMAN Settings

### Step 1: Show all persistent configuration

```rman
SHOW ALL;
```

Lines marked with default comments are baseline settings unless changed.

Important defaults to verify:

- control file autobackup is on
- archive log deletion policy is what you intend
- retention policy aligns to your recovery goals

### Step 2: Show a specific setting

```rman
SHOW CONTROLFILE AUTOBACKUP FORMAT;
```

This shows how control file autobackups are named and where they go.

### Step 3: Clear a setting if needed

```rman
CONFIGURE BACKUP OPTIMIZATION CLEAR;
```

`CLEAR` returns that setting to default behavior.

### Step 4: Exit

```rman
EXIT;
```

Practical reminder: persistent setting changes apply to future RMAN operations, not just one command.

---

## 4. Practice 3-1: Configure and Verify Default Backup Destination

Estimated time: ~5 minutes  
Assumption: terminal open, environment set for `ORCLCDB`

### Step 1: Check FRA destination in SQL*Plus

```sql
sqlplus / as sysdba
SHOW PARAMETER db_recovery_file_dest;
EXIT;
```

Expected in this practice flow:

- destination under `/u01/app/oracle/fast_recovery_area`
- size around `12G` from previous steps

### Step 2: Backup from RMAN using default destination

```text
rman target " / as sysbackup"
```

```rman
BACKUP DATABASE;
EXIT;
```

Observe that RMAN also performs control file and SPFILE autobackup when configured.

---

## 5. Practice 3-2: Set RMAN Date-Time Display Format

Estimated time: ~10 minutes  
Assumption: Practice 3-1 complete

Goal: include time-of-day in RMAN listing so same-day backups are distinguishable.

### Step 1: Update shell profile (`.bashrc`)

Set environment values, for example:

```bash
export NLS_LANG=American_America.AL32UTF8
export NLS_DATE_FORMAT='YYYY-MM-DD HH24:MI:SS'
```

This is ISO-8601-style ordering and far easier to reason about during restore decisions.

### Step 2: Reload shell session and reconnect

Close/reopen terminal or source profile, then:

```text
rman target " / as sysbackup"
```

### Step 3: Verify display format in RMAN output

```rman
LIST BACKUP;
```

Confirm timestamps now include full date-time information.

Optional good habit: spool long RMAN output to a file for audit and review.

---

## 6. Practice 3-3: Verify RMAN Settings and Backup Selected Tablespaces

Estimated time: ~5 minutes  
Assumption: Practice 3-2 complete

### Step 1: Connect and verify control file autobackup

```text
rman target " / as sysbackup"
```

```rman
SHOW CONTROLFILE AUTOBACKUP;
```

Expected in this flow: configured on (default).

### Step 2: Verify retention policy

```rman
SHOW RETENTION POLICY;
```

Expected in this flow: redundancy `1` (default).

Never assume defaults still exist. Someone could have changed them at 2:14 AM and forgotten to tell future-you.

### Step 3: Determine data files by schema report

```rman
REPORT SCHEMA;
```

Use this to identify users tablespace files in CDB and PDB context.

### Step 4: Backup users tablespace in CDB and PDB

```rman
BACKUP TABLESPACE users, ORCLPDB1:users;
```

Validate output shows expected tablespaces backed up, followed by control file/SPFILE autobackup as configured.

### Step 5: Exit

```rman
EXIT;
```

---

## 7. Why This Practice Set Matters

1. Connection syntax and environment setup avoid fake failures.
2. `LIST BACKUP` plus date-time precision helps pick the right restore source.
3. Persistent configuration awareness prevents policy surprises.
4. `DELETE OBSOLETE` keeps storage controlled without breaking retention intent.
5. Tablespace-targeted backup patterns are essential for precise recovery workflows.

It gets worse if you skip any of this and discover during an incident that your "known defaults" were actually myths.

---

## 8. Wrap-Up

You now have the hands-on RMAN workflow for this section:

- connect correctly
- run and inspect backups
- use SQL in RMAN for focused checks
- review and manage persistent settings
- complete the three core practices for destination, display format, and policy validation

Boring in rehearsal, priceless in outage response.
