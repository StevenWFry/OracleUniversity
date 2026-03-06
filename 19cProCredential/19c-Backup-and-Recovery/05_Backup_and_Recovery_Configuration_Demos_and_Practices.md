## Lesson 5 - Backup and Recovery Configuration Demos and Practices (in which sensible prep work prevents deeply unserious outages)

And look, this lesson is basically a practical lab marathon: FRA sizing, control file multiplexing, redo member expansion, and archivelog mode enablement. Calm commands, predictable output, and just enough risk to keep everyone respectful.

By the end of this lesson, you should be able to:

- Verify and resize the Fast Recovery Area (FRA)
- Add a third control file and make the instance recognize it
- Add redo members to each redo group and validate status changes
- Enable archivelog mode correctly using mount-state workflow
- Validate each configuration change with the right dynamic views

---

## 1. Demo: Configure FRA Size (because backups need somewhere to live)

Start as `SYSDBA` and check current FRA settings:

```sql
SHOW PARAMETER db_recovery_file_dest;
```

You should see:

- `db_recovery_file_dest` (location)
- `db_recovery_file_dest_size` (capacity)

Increase FRA size (example used: `30G`):

```sql
ALTER SYSTEM SET db_recovery_file_dest_size = 30G SCOPE = BOTH;
SHOW PARAMETER db_recovery_file_dest;
```

Why `SCOPE=BOTH`:

- updates memory now
- updates SPFILE for restart persistence

If FRA fills during backup, Oracle does not politely shrug and continue. It stalls work and ruins your afternoon.

---

## 2. Demo: Add Another Control File (single points of failure are tacky)

### Step A: Identify existing control files

```sql
SELECT name FROM v$controlfile;
```

### Step B: Shutdown cleanly

```sql
SHUTDOWN IMMEDIATE;
```

Exit SQL*Plus and copy a good control file to a new location.  
Example target from your demo flow:

```bash
cp <existing_control_file> /home/oracle/control03.ctl
```

### Step C: Update PFILE control file list

Go to `$ORACLE_HOME/dbs`, edit `initORCL.ora`, and append the new control file path in `control_files`.

Example pattern:

```ini
control_files='/u01/.../control01.ctl','/u02/.../control02.ctl','/home/oracle/control03.ctl'
```

### Step D: Rebuild SPFILE from updated PFILE

```sql
sqlplus / as sysdba
CREATE SPFILE FROM PFILE;
STARTUP;
SELECT name FROM v$controlfile;
```

If startup succeeds and `v$controlfile` shows all three paths, the database has accepted your new control file layout.

Practical note:

- Back up the original PFILE before editing. Typing one bad quote into `control_files` is how you earn bonus troubleshooting.

---

## 3. Demo: Add Redo Members to Existing Groups (write once, survive disk drama)

### Step A: Inspect current redo files and group status

```sql
SELECT group#, status, member FROM v$logfile;
SELECT group#, members, archived, status FROM v$log;
```

### Step B: Add one member to each group

```sql
ALTER DATABASE ADD LOGFILE MEMBER '/u01/.../redo01b.log' TO GROUP 1;
ALTER DATABASE ADD LOGFILE MEMBER '/u01/.../redo02b.log' TO GROUP 2;
ALTER DATABASE ADD LOGFILE MEMBER '/u01/.../redo03b.log' TO GROUP 3;
```

### Step C: Re-check file status

```sql
SELECT group#, status, member FROM v$logfile;
```

New members may show `INVALID` initially. Do not panic. Oracle has not cycled into them yet.

### Step D: Force log switches so all new members get used

```sql
ALTER SYSTEM SWITCH LOGFILE;
ALTER SYSTEM SWITCH LOGFILE;
ALTER SYSTEM SWITCH LOGFILE;
```

Validate again:

```sql
SELECT group#, status, member FROM v$logfile;
SELECT group#, members, archived, status FROM v$log;
```

Now you should see active/valid member usage across the expanded groups.

It gets worse if you skip this verification and discover the typo later during media recovery.

---

## 4. Demo: Enable Archivelog Mode (the difference between recoverable and vibes)

### Step A: Check current mode

```sql
ARCHIVE LOG LIST;
```

If you see `No Archive Mode`, continue.

### Step B: Transition through mount state

```sql
SHUTDOWN IMMEDIATE;
STARTUP MOUNT;
ALTER DATABASE ARCHIVELOG;
ALTER DATABASE OPEN;
ARCHIVE LOG LIST;
```

Final validation must show database in `Archive Mode`.

Why this matters:

- noarchivelog = recover to last backup only
- archivelog = recover to last committed transaction (with proper backups/logs)

One of these is operational resilience. The other is optimistic fiction.

---

## 5. Practical Runbook Checklist (do this every time, not just in class)

1. Capture current settings and file locations before changes.
2. Change one subsystem at a time (FRA, control files, redo, archivelog).
3. Validate each change immediately with SQL queries.
4. Force redo switches after adding redo members.
5. Confirm archivelog mode post-open, not just post-command.
6. Save command history and outputs for audit/repeatability.

---

## 6. Wrap-Up (configuration is boring until it saves you)

You now have one joined-up operational workflow for the four core backup/recovery configuration tasks:

- FRA sizing
- control file multiplexing
- redo member multiplexing
- archivelog enablement

None of this feels exciting while it is working. That is exactly the point.
