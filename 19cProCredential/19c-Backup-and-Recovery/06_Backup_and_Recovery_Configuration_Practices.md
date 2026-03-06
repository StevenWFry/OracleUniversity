## Lesson 6 - Backup and Recovery Configuration Practices (in which Oracle gives you four "simple" exercises and each one has a trapdoor)

And look, this is the hands-on practice pack for backup and recovery configuration: control files, FRA sizing, redo multiplexing, and archivelog mode. Calm in theory, mildly cursed in execution, and absolutely worth doing.

By the end of this lesson, you should be able to:

- Verify and improve control file redundancy
- Resize FRA safely and validate parameter changes
- Multiplex redo members and validate status transitions
- Enable archivelog mode in the correct startup state
- Explain why these steps matter when failure is not hypothetical

---

## 1. Practice Scope (what you are hardening)

This practice set targets configuration choices that determine whether recovery is smooth or performative panic:

- Control file multiplexing
- Fast Recovery Area capacity
- Redo log member redundancy
- Archivelog mode
- Archive destination resiliency

If any of these is underconfigured, recovery shifts from "procedure" to "incident."

---

## 2. Practice 2-1: Verify Control File Multiplexing

Estimated time: ~15 minutes  
Assumption: logged in as `oracle`, environment variables set

### Step A: Start instance and inspect current control files

```sql
sqlplus / as sysdba
STARTUP;
SELECT name FROM v$controlfile;
SHOW PARAMETER control_files;
```

You should initially see two control files.

### Step B: Create PFILE, then shutdown

```sql
CREATE PFILE FROM SPFILE;
SHUTDOWN IMMEDIATE;
EXIT;
```

### Step C: Prepare filesystem and backup init file

```bash
mkdir -p /u01/app/oracle/controlfiles/ORCLCDB
cp $ORACLE_HOME/dbs/initORCLCDB.ora $ORACLE_HOME/dbs/backup_initORCLCDB.ora
cp /u01/app/oracle/oradata/ORCL/control01.ctl /u01/app/oracle/controlfiles/ORCLCDB/control03.ctl
```

### Step D: Edit PFILE and append third control file

Edit `$ORACLE_HOME/dbs/initORCLCDB.ora` and update `control_files`:

```ini
control_files='/u01/app/oracle/oradata/ORCL/control01.ctl',
'/u01/app/oracle/fast_recovery_area/ORCL/control02.ctl',
'/u01/app/oracle/controlfiles/ORCLCDB/control03.ctl'
```

### Step E: Startup test (the gotcha)

```sql
sqlplus / as sysdba
STARTUP;
SELECT name FROM v$controlfile;
```

You may still see only two control files. Why? Because SPFILE takes precedence, and you only changed PFILE.

It gets worse: many people stop here and think the database ignored them out of spite. It did not. It just used the file you did not update.

### Step F: Rebuild SPFILE from edited PFILE

```sql
SHUTDOWN IMMEDIATE;
CREATE SPFILE FROM PFILE;
STARTUP;
SHOW PARAMETER control_files;
SELECT name FROM v$controlfile;
```

Now the third control file should appear.

---

## 3. Practice 2-2: Configure FRA Size

Estimated time: ~5 minutes  
Assumption: still logged in

### Step A: Check FRA location and size

```sql
SHOW PARAMETER db_recovery_file_dest;
SHOW PARAMETER db_recovery_file_dest_size;
```

### Step B: Resize FRA to 12G and validate

```sql
ALTER SYSTEM SET db_recovery_file_dest_size = 12G SCOPE = BOTH;
SHOW PARAMETER db_recovery_file_dest_size;
```

If this is not sized correctly, backups and archived logs eventually compete for space like airline passengers for one overhead bin.

---

## 4. Practice 2-3: Verify Redo Log Multiplexing

Estimated time: ~20 minutes  
Assumption: SQL*Plus session active

### Step A: Inspect current redo members and groups

```sql
COLUMN member FORMAT A65
SELECT group#, status, member FROM v$logfile;
SELECT group#, members, archived, status FROM v$log;
```

### Step B: Add one extra member per group

```sql
ALTER DATABASE ADD LOGFILE MEMBER '/u01/app/oracle/fast_recovery_area/redo01b.log' TO GROUP 1;
ALTER DATABASE ADD LOGFILE MEMBER '/u01/app/oracle/fast_recovery_area/redo02b.log' TO GROUP 2;
ALTER DATABASE ADD LOGFILE MEMBER '/u01/app/oracle/fast_recovery_area/redo03b.log' TO GROUP 3;
```

### Step C: Validate status and force usage

```sql
SELECT group#, status, member FROM v$logfile ORDER BY 1,3;
```

New members can show `INVALID` until written.

Force log switches:

```sql
ALTER SYSTEM SWITCH LOGFILE;
ALTER SYSTEM SWITCH LOGFILE;
ALTER SYSTEM SWITCH LOGFILE;
```

Recheck:

```sql
SELECT group#, status, member FROM v$logfile ORDER BY 1,3;
SELECT group#, members, archived, status FROM v$log;
```

All members should now be `VALID`.

### Step D: Clean-up flow (as shown in demo)

Drop added members one by one (switching logs as needed):

```sql
ALTER DATABASE DROP LOGFILE MEMBER '/u01/app/oracle/fast_recovery_area/redo02b.log';
ALTER SYSTEM SWITCH LOGFILE;
ALTER DATABASE DROP LOGFILE MEMBER '/u01/app/oracle/fast_recovery_area/redo03b.log';
ALTER SYSTEM SWITCH LOGFILE;
ALTER DATABASE DROP LOGFILE MEMBER '/u01/app/oracle/fast_recovery_area/redo01b.log';
```

Then remove physical files at OS level:

```bash
rm /u01/app/oracle/fast_recovery_area/redo*.log
ls -la /u01/app/oracle/fast_recovery_area
```

Concept check from this practice:

- Yes, Oracle can continue if at least one member in the group survives.
- No, that does not mean "single-member groups are fine." That is how outage postmortems get written.

---

## 5. Practice 2-4: Configure Archivelog Mode

Estimated time: ~10 minutes  
Assumption: logged in as `oracle`

### Step A: Check current archive mode

```sql
sqlplus / as sysdba
ARCHIVE LOG LIST;
```

### Step B: Move to mount and enable archivelog

```sql
SHUTDOWN IMMEDIATE;
STARTUP MOUNT;
ALTER DATABASE ARCHIVELOG;
ARCHIVE LOG LIST;
ALTER DATABASE OPEN;
```

Final validation should show:

- database in `Archive Mode`
- automatic archival enabled
- archive destination configured (often FRA by default)

---

## 6. Why These Practices Matter (yes, this is all connected)

1. More control files = easier recovery from control file loss.
2. Correct FRA sizing = fewer backup/archive failures due to space pressure.
3. Multiplexed redo members = protection from single log-file media faults.
4. Archivelog mode = complete and point-in-time recovery options.

And that is the entire game: make recovery boring before production makes it dramatic.

---

## 7. Wrap-Up

You now have the complete practice workflow for the backup and recovery configuration section:

- verified control file multiplexing
- resized FRA
- verified and exercised redo multiplexing
- enabled and validated archivelog mode

This is exactly the sort of "unexciting" prep work that keeps real outages from becoming career folklore.
