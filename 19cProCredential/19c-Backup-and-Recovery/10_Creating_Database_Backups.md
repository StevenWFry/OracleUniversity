## Lesson 10 - Creating Database Backups (in which RMAN gives you options, and each option has consequences)

And look, this lesson is where backup strategy becomes actual commands: full and partial backups, incremental workflows, image copies, and the famous "this seemed simple until I had to restore it" moment.

By the end of this lesson, you should be able to:

- Create whole and partial backups with RMAN
- Back up CDB root, selected PDBs, and targeted tablespaces
- Distinguish backup sets from image copies in practical terms
- Use core RMAN backup syntax and scope clauses without guessing
- Configure control file/SPFILE autobackup behavior
- Understand incrementally updated image copy workflows
- Use optional RMAN features such as compression, multisection backups, duplexing, and archival retention

---

## 1. RMAN Setup and Core Flow

Before running RMAN, source your environment and confirm SID context.

Example:

```bash
export ORACLE_SID=cdb1
rman target " / as sysbackup"
```

### `REPORT SCHEMA` before targeted backup work

Before you start backing up selected pieces of the database, it helps to know
what Oracle thinks the current physical layout actually is.

Use:

```rman
REPORT SCHEMA;
```

This gives you a practical inventory of the files and tablespaces in scope.

Why this matters:

- you can confirm container-root and PDB-level tablespace names
- you avoid backing up the wrong thing because you were "pretty sure" that was
  the tablespace name
- you spend less time improvising and more time being correct

Which is, admittedly, a stronger overall lifestyle.

Core backup command patterns:

```rman
BACKUP DATABASE;
BACKUP DATABASE PLUS ARCHIVELOG;
```

If you include `PLUS ARCHIVELOG`, you preserve more complete recovery options. If you skip it, recovery flexibility gets worse later, usually at the exact moment you need it most.

---

## 2. Whole vs Partial Backup Scope (CDB/PDB aware)

### Whole database backup (multitenant)

```rman
BACKUP DATABASE PLUS ARCHIVELOG;
```

This captures:

- CDB root
- all included PDB data files in scope
- archived redo logs (because you asked for them)
- control file and SPFILE (with proper autobackup/config behavior)

### Demo-style whole backup flow

If you want the classic "back up the whole thing and include archived redo
logs" command, this is the one:

```rman
BACKUP DATABASE PLUS ARCHIVELOG;
```

That works for:

- a non-CDB database
- a multitenant CDB environment

In the multitenant case, RMAN backs up the whole container database scope,
including the root and the PDB data files that belong to that database.

What to review after it runs:

- the backup sets created
- the starting and ending backup pieces
- the archived redo log backup pieces

This is useful because successful completion is good, but knowing **what** RMAN
actually produced is better. Blind faith is not a backup validation strategy.

### Practice-style whole backup notes

In the practice flow, the database is backed up with:

```rman
BACKUP DATABASE PLUS ARCHIVELOG;
```

And the key review points are:

- you do **not** need to shut down the database if it is in `ARCHIVELOG` mode
- an online backup is operationally usable even though it is not a
  shut-down-consistent image at one instant in time
- archived redo logs and online redo complete the recovery story
- SPFILE and control file protection are included through autobackup behavior
  when configured

This is where people get sloppy with terms, so keep the distinction straight:

- **offline / cold backup** = database closed, consistent backup
- **online / hot backup** = database open, recovered to consistency using redo

Oracle will absolutely let you back up a running database.
It just expects you to understand that "running" and "self-contained without
redo" are not the same thing.

### Recover whole database

```rman
RECOVER DATABASE;
```

---

## 3. Backup Specific PDBs or Objects

You can target just selected pluggable databases:

```rman
BACKUP PLUGGABLE DATABASE sales_pdb, hr_pdb;
BACKUP PLUGGABLE DATABASE sales_pdb PLUS ARCHIVELOG;
```

You can also target specific tablespaces using PDB-qualified syntax:

```rman
BACKUP TABLESPACE sales_pdb:tbs2;
BACKUP TABLESPACE hr_pdb:system, sales_pdb:sysaux;
```

If you reference an unqualified tablespace (for example `sysaux`), RMAN interprets it in current/root context unless explicitly qualified.

### Demo-style syntax refresher

These are the basic forms you actually type when you want RMAN to stop being a
theory exercise and start doing work:

Whole database, no archived redo logs:

```rman
BACKUP DATABASE;
```

Single pluggable database:

```rman
BACKUP PLUGGABLE DATABASE sales_pdb;
```

Single tablespace inside a PDB:

```rman
BACKUP TABLESPACE sales_pdb:users;
```

The main point here is scope:

- `DATABASE` = everything in the container database
- `PLUGGABLE DATABASE pdb_name` = one PDB
- `TABLESPACE pdb_name:tbs_name` = one named tablespace inside a specific PDB

This is where `REPORT SCHEMA` earns its keep, because the syntax is simple right
up until you confidently type the wrong tablespace name and wonder why RMAN is
looking at you like you brought a spoon to a welding job.

### Demo-style partial PDB backup flow

If you are backing up selected tablespaces instead of the whole container
database, the practical RMAN flow is:

1. source the database environment
2. connect to RMAN as `SYSBACKUP`
3. run `REPORT SCHEMA`
4. copy the exact PDB-qualified tablespace names you need
5. issue `BACKUP TABLESPACE ...`

Single tablespace from one PDB:

```rman
REPORT SCHEMA;
BACKUP TABLESPACE pdb1:users;
```

Multiple tablespaces from multiple PDBs:

```rman
BACKUP TABLESPACE pdb1:users, rcat:users;
```

Mixed root-and-PDB scope in one command:

```rman
BACKUP TABLESPACE undotbs1, pdb1:sysaux;
```

That last example is useful because RMAN is perfectly happy to back up a root
tablespace and a PDB-qualified tablespace in the same command, provided you use
the names correctly.

Which brings us back, once again, to `REPORT SCHEMA`, because copy-pasting the
right names beats retyping them from memory and then discovering you have backed
up confidence instead of data.

### Practice-style partial backup notes

The practice also walks through two useful multitenant behaviors:

1. backing up a full PDB from the CDB root
2. backing up a single PDB tablespace from the CDB root

Examples:

```rman
BACKUP PLUGGABLE DATABASE orclpdb1 PLUS ARCHIVELOG;
BACKUP TABLESPACE orclpdb2:users;
```

Important operational nuance:

- when connected to the **CDB root**, RMAN can perform CDB-aware backup
  operations and use root-level RMAN configuration
- when connected directly to a **PDB**, you can back up that PDB with
  `BACKUP DATABASE`, but you cannot change RMAN configuration there the same way

That is why a command like:

```rman
CONFIGURE CONTROLFILE AUTOBACKUP ON;
```

fails when connected only to a PDB.

Oracle is basically saying, quite reasonably:

"You are in the apartment, not the building management office."

---

## 4. Backup Set vs Image Copy

### Backup set (default RMAN style)

Compressed/proprietary RMAN format, smaller footprint:

```rman
BACKUP AS BACKUPSET TABLESPACE hr_data
  FORMAT '/u01/backup/%U';
```

Note: `AS BACKUPSET` is often implicit default, but stating it can make intent obvious in scripts.

If you want to be explicit at the full-database level, that syntax works too:

```rman
BACKUP AS BACKUPSET DATABASE;
```

That does not change RMAN's default personality so much as force it to say the
quiet part out loud.

### Image copy (`AS COPY`)

Bit-for-bit file copy:

```rman
BACKUP AS COPY DATAFILE '/u01/app/oracle/oradata/ORCL/users_01_db01.dbf';
BACKUP AS COPY ARCHIVELOG LIKE 'arch%';
```

Image copies are bigger and slower to produce, but they can make restore/switch operations faster and simpler.

### Demo-style image copy syntax

If you are creating image copies interactively, the practical flow is:

1. connect to RMAN as a user with `SYSBACKUP`
2. run `REPORT SCHEMA`
3. identify the exact file or scope you want
4. issue the `BACKUP AS COPY ...` command

Single datafile as an image copy:

```rman
REPORT SCHEMA;
BACKUP AS COPY DATAFILE '/u01/app/oracle/oradata/ORCL/users01.dbf';
```

That is useful when you want a physical image copy of one specific data file,
not a whole backup set wrapped in RMAN packaging.

Archived redo logs as image copies, using a filename pattern:

```rman
BACKUP AS COPY ARCHIVELOG LIKE '%arc%';
```

Important syntax reminder:

- it is `ARCHIVELOG`, not `ARCHIVE`

And practical behavior reminder:

- if the pattern matches no archived redo log files, RMAN will not invent them
  out of courtesy
- instead, the command finishes with the operational equivalent of "nothing
  matched, so nothing happened"

This is one of those commands that is perfectly simple right up until a missing
`LOG` keyword or a bad pattern turns it into a tiny public embarrassment.

### Practice-style image copy tradeoffs

The practice compares whole database image copies to backup sets.

Key takeaway:

- image copies are easier to work with at restore time because the files are
  already in database-file form
- backup sets usually save more space because they do not waste effort storing
  all unused blocks as bluntly as image copies do

In most environments:

- backup sets win on storage efficiency
- image copies win on restore simplicity and granularity

Which means, as usual, the correct answer is not "which is better?" but "better
for what?"

---

## 5. Control File and SPFILE Protection

Recommended persistent setting:

```rman
CONFIGURE CONTROLFILE AUTOBACKUP ON;
```

With this enabled, RMAN automatically captures current control file and SPFILE metadata in backup workflows. This is exactly the kind of boring safeguard that saves you from very expensive improvisation later.

---

## 6. Full vs Incremental Refresher

### Full backup

- All used blocks from data files.

### Incremental level 0

- Base equivalent used for incremental strategy lineage.

### Incremental level 1 cumulative

- Changes since last level 0.

### Incremental level 1 differential

- Changes since last incremental backup.

Cumulative is usually larger. Differential is usually smaller and more frequent-friendly.

---

## 7. Incrementally Updated Image Copy (rolling-forward strategy)

This workflow keeps an image copy progressively current using level 1 incrementals.

Example command pair:

```rman
RECOVER COPY OF DATABASE WITH TAG 'DAILY_INC';
BACKUP INCREMENTAL LEVEL 1 FOR RECOVER OF COPY WITH TAG 'DAILY_INC' DATABASE;
```

How it behaves:

1. Day 1: no prior copy -> RMAN creates baseline image copy.
2. Day 2: no prior incremental to apply -> RMAN produces first level 1 incremental.
3. Day 3 onward: RMAN applies previous incremental to roll image copy forward, then takes next incremental.

Result: you maintain a near-current image copy plus incremental chain.

This is one of those strategies that feels magical when tested properly and catastrophic when never rehearsed.

---

## 8. Fast Incremental Backups and Block Change Tracking

If your incremental backups are reading far too much data just to figure out
what changed, then it is time to let Oracle stop wandering around the files like
someone searching for car keys in a parking lot.

That is where **block change tracking** helps.

### What it does

With block change tracking enabled, Oracle records changed block ranges in a
small tracking file.

Then, when RMAN runs an incremental backup with level greater than `0`, it can
consult that file to identify changed blocks instead of scanning every block in
every data file.

This is especially helpful when only a small percentage of data blocks changed
between backups.

Important correction to the lecture shorthand:

- the benefit is **not** "less than 20% change means it works"
- the feature is generally useful whenever the changed-block percentage is small
- the `20%` rule of thumb is more about **when to consider taking a fresh level
  0**, not the definition of block change tracking benefit

### Enabling it

Block change tracking is disabled by default.

You can enable it while the database is `OPEN` or `MOUNTED`:

```sql
ALTER DATABASE ENABLE BLOCK CHANGE TRACKING;
```

Or place the file exactly where you want:

```sql
ALTER DATABASE ENABLE BLOCK CHANGE TRACKING
  USING FILE '/u02/oradata/bct/rman_change_track.f' REUSE;
```

### Demo-style maintenance flow

If you want the practical command-line sequence for maintaining the block change
tracking file, it looks like this:

1. source the correct database environment
2. connect as an administrative user, typically `SYSDBA`
3. verify or set the default Oracle Managed Files location if you plan to let
   Oracle create the file automatically
4. enable block change tracking
5. verify the file exists
6. disable it later if you no longer need it

Check the current setting first:

```sql
SHOW PARAMETER DB_CREATE_FILE_DEST;
```

If the parameter is not set and you want Oracle Managed Files behavior, Oracle
docs show using:

```sql
ALTER SYSTEM SET DB_CREATE_FILE_DEST = '/u01/app/oracle/oradata'
  SCOPE=BOTH SID='*';
```

Then enable block change tracking:

```sql
ALTER DATABASE ENABLE BLOCK CHANGE TRACKING;
```

Or specify a file explicitly:

```sql
ALTER DATABASE ENABLE BLOCK CHANGE TRACKING
  USING FILE '/home/oracle/ctf.dbf' REUSE;
```

That second form is useful when you want the file somewhere very specific and do
not want Oracle generating the filename for you.

### Default location

Current Oracle docs say the default change tracking file is created as an Oracle
Managed File in the location specified by `DB_CREATE_FILE_DEST`.

So no, the default is **not** simply "the FRA."

If you want it somewhere else, specify the file explicitly when enabling it.

And one subtle but important point:

- `DB_CREATE_FILE_DEST` is a general Oracle Managed Files default location
- it is not a "block change tracking only" parameter

So do not change it lightly on a production system unless you actually intend to
change where Oracle creates managed files more broadly.

### The "eight backups" detail, translated into English

Oracle automatically manages the change tracking file to retain bitmaps covering
the **eight most recent backups**.

That does **not** mean you manually create a new file after eight incrementals.

It means Oracle cycles the internal bitmap history.

Why this matters:

- if your parent backup falls outside the retained bitmap history, RMAN cannot
  optimize that incremental with block change tracking
- this is one reason incremental strategy design still matters

### Practice-style BCT configuration notes

The practice version of this is refreshingly short:

1. connect as `SYSDBA`
2. make sure `DB_CREATE_FILE_DEST` points somewhere sensible
3. enable block change tracking

Representative commands:

```sql
ALTER SYSTEM SET DB_CREATE_FILE_DEST = '/u01/app/oracle/oradata/ORCLCDB'
  SCOPE=BOTH;

SHOW PARAMETER DB_CREATE_FILE_DEST;

ALTER DATABASE ENABLE BLOCK CHANGE TRACKING;
```

The strategic point is more interesting than the syntax:

- block change tracking helps incremental backups only while Oracle still has
  the bitmap history it needs
- the file keeps eight bitmaps, not an infinite scrapbook of your backup life
- so if you take a level `0` and then keep stacking incrementals forever,
  eventually the parent relationship ages out of the bitmap history

One especially annoying example:

- level `0`
- seven differential level `1` backups
- then a cumulative level `1`

At that point, the parent bitmap history you hoped to use may already have been
overwritten, so RMAN cannot fully optimize the backup with block change
tracking.

Which is Oracle's way of saying: yes, the feature is smart, but it is not a
time machine.

### Checking whether it is being used

To see whether block change tracking is enabled:

```sql
SELECT status, filename, bytes
FROM   V$BLOCK_CHANGE_TRACKING;
```

To see whether a given incremental backup actually used change tracking:

```sql
SELECT incremental_level,
       used_change_tracking,
       blocks_read,
       blocks,
       datafile_blocks
FROM   V$BACKUP_DATAFILE
WHERE  incremental_level > 0;
```

If `USED_CHANGE_TRACKING = 'YES'`, then RMAN used the feature to accelerate that
incremental backup.

And if `BLOCKS_READ` is much smaller than `DATAFILE_BLOCKS`, then Oracle has
successfully avoided a lot of unnecessary scanning, which is the whole point.

### Renaming or moving the file

If you want to rename the block change tracking file with `ALTER DATABASE RENAME
FILE`, the database must be `MOUNTED` but not open.

That is not Oracle being dramatic.
That is Oracle refusing to rename a tracking file while also actively depending
on it.

### Disabling it

If you no longer want to use block change tracking:

```sql
ALTER DATABASE DISABLE BLOCK CHANGE TRACKING;
```

Current Oracle documentation says disabling it deletes the existing block change
tracking file from the operating system.

So yes, Oracle cleans up after itself here, which is refreshingly civilized for
a feature that otherwise spends its life keeping score on changed blocks.

---

## 9. Oracle-Suggested Rolling Backup Strategy

Oracle likes a backup pattern that combines:

- a base image copy
- daily incremental backups
- periodic roll-forward of the image copy
- archived redo log backups
- control file/SPFILE protection

Which is basically the strategy we already covered with incrementally updated
image copies:

```rman
RECOVER COPY OF DATABASE WITH TAG 'DAILY_INC';
BACKUP INCREMENTAL LEVEL 1 FOR RECOVER OF COPY WITH TAG 'DAILY_INC' DATABASE;
```

Why Oracle likes this:

- your image copy stays reasonably current
- you reduce how many incrementals must be applied during recovery
- you are not stuck with a "full backup from ages ago and a towering stack of
  regret" recovery model

Add archived logs to that strategy and you get something much closer to a sane
operational backup routine.

### Practice-style incrementally updated copy workflow

The lab version turns that strategy into an actual operating sequence.

First, create a baseline image copy and tag it so RMAN knows which copy chain
you are maintaining:

```rman
RUN {
  ALLOCATE CHANNEL ch1 DEVICE TYPE DISK
    FORMAT '/home/oracle/backup/orclcdb/%U';
  BACKUP AS COPY TAG 'BASE01' INCREMENTAL LEVEL 0 DATABASE;
}
```

Then generate some changed blocks by doing actual workload. In the practice,
that is done with a lab script that modifies the sample inventory data.

After that, create the level `1` incremental intended to roll the tagged copy
forward:

```rman
RUN {
  ALLOCATE CHANNEL ch1 DEVICE TYPE DISK
    FORMAT '/home/oracle/backup/orclcdb/%U';
  BACKUP INCREMENTAL LEVEL 1
    FOR RECOVER OF COPY
    WITH TAG 'BASE01'
    DATABASE;
}
```

Then apply the incremental to the existing image copy:

```rman
RUN {
  ALLOCATE CHANNEL ch1 DEVICE TYPE DISK
    FORMAT '/home/oracle/backup/orclcdb/%U';
  RECOVER COPY OF DATABASE WITH TAG 'BASE01';
}
```

What this achieves:

- the image copy starts life as the level `0` base
- the level `1` captures only changed blocks
- `RECOVER COPY` rolls the image copy forward so it now looks like a fresh
  current baseline again

So the level `0` image copy stops being an aging museum exhibit and becomes a
living backup artifact.

Practical cleanup from the lab:

```sql
ALTER DATABASE DISABLE BLOCK CHANGE TRACKING;
```

```rman
DELETE OBSOLETE;
CROSSCHECK DATAFILECOPY ALL;
```

That last `CROSSCHECK` is not glamorous, but it is how you stop RMAN metadata
from drifting into fiction.

---

## 10. Control File Autobackup and Control File to Trace

Autobackup is the everyday safety net:

```rman
CONFIGURE CONTROLFILE AUTOBACKUP ON;
```

That protects:

- current control file metadata
- server parameter file (`SPFILE`)

But you also have another useful option:

### Back up the control file to a trace file

```sql
ALTER DATABASE BACKUP CONTROLFILE TO TRACE;
```

Or direct it to a named SQL file:

```sql
ALTER DATABASE BACKUP CONTROLFILE TO TRACE AS '/u01/backup/create_controlfile.sql';
```

Why this matters:

- it produces SQL text containing a `CREATE CONTROLFILE` statement
- you can edit it if needed
- it is very useful after structural changes

This is the sort of thing that feels unnecessary right up until the control file
goes missing and suddenly everybody becomes very interested in your past
decision-making.

Important nuance:

- the trace file is not the same as a binary control file backup
- it is a recovery-oriented SQL script template

So keep both ideas in your head, because they solve related but not identical
problems.

### Demo-style trace backup workflow

If you want the practical lab-style flow for backing up the control file to a
trace file, it looks like this:

1. make sure the correct Oracle environment is set
2. connect with `sqlplus / as sysdba`
3. issue the trace-backup command
4. inspect the alert log to find the generated trace file
5. open the trace file and keep the section you actually need

Basic command:

```sql
ALTER DATABASE BACKUP CONTROLFILE TO TRACE;
```

You can verify configured control file members first:

```sql
SELECT name
FROM   V$CONTROLFILE;
```

That is useful because multiplexed control files are supposed to save your day
from becoming dramatically worse.

If one member is lost, replacement is usually much simpler than a total
control-file-loss scenario. If all control files are gone, that trace script
suddenly becomes much more interesting.

### Finding the trace file

The generated trace file location is written to the alert log in the ADR
diagnostic area.

Typical operating-system pattern:

```bash
tail -50 $ORACLE_BASE/diag/rdbms/<db_unique_name>/<instance_name>/trace/alert_<instance_name>.log
```

Then inspect the generated trace file with something readable:

```bash
more /u01/app/oracle/diag/rdbms/orcl/ORCL/trace/ORCL_ora_12345.trc
```

Or:

```bash
less /u01/app/oracle/diag/rdbms/orcl/ORCL/trace/ORCL_ora_12345.trc
```

### What is inside the trace file

The trace file contains:

- comments and generation notes
- a `CREATE CONTROLFILE ... NORESETLOGS` version
- a `CREATE CONTROLFILE ... RESETLOGS` version
- data file, redo log, and other structural definitions

The practical rule is:

- use the `NORESETLOGS` section when you have the current online redo situation
  and are doing complete recovery
- use the `RESETLOGS` section when incomplete recovery is required or the online
  redo situation no longer supports `NORESETLOGS`

In either case, you do **not** just run the entire trace file as-is. You trim
the comments and keep the section that matches the recovery scenario instead of
treating the file like a sacred scroll that must be executed whole.

### When to regenerate the trace file

If the physical structure changes, regenerate it.

That includes things like:

- adding or dropping data files
- moving or resizing files
- changing redo log structure
- changing archive-log-related structure
- other structural database changes

Because the trace script is only as useful as it is current, and stale control
file reconstruction text is one of those gifts that keeps on giving in the worst
possible way.

### Practice-style lab note

In the lab flow, they also verify archive log mode and control file multiplexing
before working through the trace file.

Representative checks:

```sql
ARCHIVE LOG LIST;
SELECT name FROM V$CONTROLFILE;
```

Those are not mandatory every time you back up control file to trace, but they
are sensible checks when you are practicing whole-database recovery workflows
and want to know exactly what shape the system is in before you start breaking
things academically.

### Verifying control file autobackup behavior

From RMAN, you can check the persistent configuration with:

```rman
SHOW ALL;
```

Then look for:

```rman
CONFIGURE CONTROLFILE AUTOBACKUP ON;
```

Important practical nuance:

- many lab or DBCA-built environments show this enabled
- do **not** assume that every database everywhere has it enabled just because a
  classroom system did

Also note:

- RMAN autobackup protects the control file contents and the `SPFILE`
- it is not backing up each multiplexed control file member as separate logical
  treasures

Multiplexing protects you at the file-member level.
RMAN autobackup protects you at the backup/recovery level.
These are related protections, not interchangeable ones.

### FRA review and retention policy practice

The practice also walks through reviewing where backups landed in the Fast
Recovery Area and how retention policy drives cleanup.

Useful commands:

```rman
SHOW RETENTION POLICY;
LIST BACKUP;
DELETE OBSOLETE;
```

Useful operational ideas:

- FRA space management depends heavily on retention policy
- obsolete files are deleted when space pressure requires it
- if no files are obsolete, `DELETE OBSOLETE` will simply tell you so and move
  on with its day
- if FRA fills up during backup, increasing
  `DB_RECOVERY_FILE_DEST_SIZE` may be required

And one practical lab observation worth keeping:

- control file/SPFILE autobackups typically end up under the FRA autobackup area
- regular backup pieces and image copies appear under the backup area structure

So if the question is "where did my files actually go?" the answer is usually
"to the FRA, according to both your retention policy and Oracle's opinions
about storage hierarchy."

### Practice-style additional file backups

The practice also covers two common "do not forget these" tasks:

1. write a readable control file recreation script to a known path
2. back up archived redo logs and delete the input logs after they are safely in
   backup

Named control-file trace example:

```sql
ALTER DATABASE BACKUP CONTROLFILE TO TRACE AS
  '/home/oracle/backup/orclcdb/control.sql';
```

That gives you an explicit SQL script file instead of making you hunt through
ADR first, which is a much nicer arrangement when you already know where you
want the artifact.

Archived redo log backup with input deletion:

```rman
RUN {
  ALLOCATE CHANNEL CH1 DEVICE TYPE DISK
    FORMAT '/home/oracle/backup/orclcdb/%U';
  BACKUP ARCHIVELOG ALL DELETE ALL INPUT;
}
```

Meaning:

- back up all archived redo logs
- once RMAN is satisfied they are protected, remove the source archived logs

That is useful for space control, but only if your backup destination is real,
reachable, and part of an actual restore plan rather than a comforting myth.

---

## 11. Cataloging Files RMAN Does Not Currently Know About

Sometimes backup-related files exist on disk, but RMAN does not currently know
about them in its repository metadata.

This can happen if:

- control file records aged out
- you re-created the control file
- you moved files around
- you restored older files that need to be made known again

RMAN can catalog:

- backup pieces
- datafile copies
- control file copies
- archived redo log files

This is the point of the `CATALOG` command family.

Typical use cases:

```rman
CATALOG START WITH '/u01/backup/';
CATALOG BACKUPPIECE '/u01/backup/db_0erl7neb_1_1.bkp';
CATALOG DATAFILECOPY '/u01/backup/users01_copy.dbf';
```

This is RMAN's version of being told, "yes, these files are real, yes, they
matter, and yes, please stop pretending you have never seen them before."

Related repository warning:

The control file does **not** retain RMAN metadata forever.

The parameter `CONTROL_FILE_RECORD_KEEP_TIME` controls the minimum age, in days,
before records are candidates for reuse.

Default:

- `7` days, not `14`

If you need longer metadata retention without losing your sanity, use a recovery
catalog.

---

## 12. Reporting on Backups

RMAN reporting is wonderfully blunt.

### Basic listing

```rman
LIST BACKUP;
LIST BACKUP SUMMARY;
LIST COPY;
```

### Repository-style reporting

```rman
REPORT OBSOLETE;
REPORT NEED BACKUP;
```

Useful interpretations:

- `REPORT OBSOLETE` tells you what is no longer needed by the retention policy
- `REPORT NEED BACKUP` tells you what still needs protection

And because Oracle loves two similar words that mean very different things:

- **obsolete** = no longer needed for recovery per retention policy
- **expired** = RMAN expected the file to exist but could not find it during
  `CROSSCHECK`

If you mix those up, cleanup decisions become much more adventurous than they
should be.

---

## 13. Dynamic Views Behind the Reporting

The RMAN `LIST` and `REPORT` commands, and the Enterprise Manager pages sitting
on top of them, are all drawing from repository metadata and related views.

Useful views include:

- `V$BACKUP_SET`
- `V$BACKUP_PIECE`
- `V$BACKUP_DATAFILE`
- `V$DATAFILE_COPY`
- `V$BACKUP_FILES`

Representative examples:

```sql
SELECT recid, set_stamp, set_count, completion_time
FROM   V$BACKUP_SET;

SELECT handle, status, bytes
FROM   V$BACKUP_PIECE;

SELECT file#, incremental_level, used_change_tracking
FROM   V$BACKUP_DATAFILE;
```

This is useful when:

- you want custom reporting
- you want to validate what RMAN is actually recording
- you want to understand what Enterprise Manager is showing without treating the
  GUI like a magical prophecy tablet

---

## 14. Optional RMAN Backup Features

This is the part of RMAN where the menu gets longer, the syntax gets more
decorated, and suddenly every option sounds like something you should obviously
turn on until licensing, CPU, or reality taps you on the shoulder.

### RMAN already skips some blocks for you

RMAN has optimization features that reduce backup size before you even start
talking about binary compression.

Oracle distinguishes between:

- **unused block compression**
- **null block compression**

In plain English:

- backup sets can skip blocks that are not currently allocated to database
  objects
- full and level `0` backup sets also omit blocks that have never contained data

This is why a backup set is usually leaner than an image copy. An image copy is
still a blunt physical copy, whereas a backup set is allowed to be smarter.

Important nuance from Oracle docs:

- unused block compression is automatic only in specific cases, including backup
  sets for full or level `0` backups
- guaranteed restore points can disable that benefit
- this is not the same thing as binary compression

So yes, Oracle is already trying not to back up obvious dead air. It just does
not call it "dead air," because Oracle manuals are many things, but rarely that
honest.

### Binary compression of backup sets

If you want RMAN to do actual binary compression, use compressed backup sets.

Persistent configuration example:

```rman
CONFIGURE DEVICE TYPE DISK BACKUP TYPE TO COMPRESSED BACKUPSET;
CONFIGURE COMPRESSION ALGORITHM 'MEDIUM';
```

One-off job example:

```rman
RUN {
  SET COMPRESSION ALGORITHM 'HIGH';
  BACKUP AS COMPRESSED BACKUPSET DATABASE PLUS ARCHIVELOG;
}
```

Important correction to the lecture shorthand:

- the syntax is `AS COMPRESSED BACKUPSET`
- not just "compressed database" waved vaguely in RMAN's direction

Compression choices:

- `BASIC` does **not** require the Advanced Compression option
- `LOW`, `MEDIUM`, and `HIGH` do require the Oracle Advanced Compression option

Practical tradeoff:

- `LOW` = lower CPU overhead, lighter compression
- `MEDIUM` = balanced choice
- `HIGH` = tighter compression, more CPU appetite
- `BASIC` = decent compression, but often with heavier CPU cost than you would
  love

So when someone says, "we'll save storage by compressing everything," the DBA
translation is, "excellent, and which CPU budget did you want to set on fire for
that?"

### Demo-style compressed backup flow

The lab version of this is straightforward:

1. connect to RMAN
2. run `SHOW ALL;`
3. check the current compression-related settings
4. change the compression algorithm if needed
5. run a compressed backup

Start by reviewing the current RMAN configuration:

```rman
SHOW ALL;
```

That lets you see whether compressed backup sets are already configured as the
default and which compression algorithm RMAN is currently set to use.

If you want to change the algorithm persistently, use:

```rman
CONFIGURE COMPRESSION ALGORITHM 'HIGH';
```

Then run the backup explicitly as a compressed backup set:

```rman
BACKUP AS COMPRESSED BACKUPSET DATABASE PLUS ARCHIVELOG;
```

Practical nuance:

- if you have already configured the device type to use compressed backup sets
  by default, you do not have to say `AS COMPRESSED BACKUPSET` every single
  time
- saying it anyway is still useful when you want the script to be painfully
  clear

And yes, if you choose a heavier compression setting and include both database
files and archived redo logs, the backup may take noticeably longer. This is
not RMAN being broken. This is CPU time being converted into storage savings,
which is a very Oracle sort of barter system.

### Multisection backups for large files

If one giant file is holding the entire backup job hostage, multisection backups
let multiple channels work on separate sections of that file in parallel.

Example:

```rman
BACKUP
  SECTION SIZE 25M
  DATAFILE 5
  TAG 'DF5_SECTIONED';
```

Oracle also supports sectioned validation:

```rman
RUN {
  ALLOCATE CHANNEL c1 DEVICE TYPE DISK;
  ALLOCATE CHANNEL c2 DEVICE TYPE DISK;
  VALIDATE DATAFILE 5 SECTION SIZE 25M;
}
```

Important nuances:

- `SECTION SIZE` can be used for multisection backup sets and, on compatible
  releases, multisection image copies and incrementals
- if the section size is larger than the file, RMAN will not bother creating
  sections
- if your section size would create more than `256` sections, RMAN enlarges the
  effective section size
- `SECTION SIZE` cannot be combined with `MAXPIECESIZE`

The point is simple: stop making three channels sit idle while one heroic
channel drags a giant file uphill alone.

### Tape, media managers, and proxy copy

For non-disk backup devices, RMAN works through an SBT media manager.

Proxy copy means RMAN hands control of the actual data transfer to the media
manager:

```rman
BACKUP DEVICE TYPE sbt PROXY DATABASE;
```

If you want RMAN to fail instead of silently falling back to conventional backup
behavior when proxy copy is unavailable, use:

```rman
BACKUP DEVICE TYPE sbt PROXY ONLY DATABASE;
```

Key points:

- proxy copy does **not** work with `DISK` channels
- it depends on media-manager support
- control files are never backed up as proxy copies, even if you ask

So proxy copy is less "RMAN does a different style of copy" and more "RMAN
politely steps aside while the media manager takes the wheel."

### Duplexed backup sets

If you want multiple identical copies of the same backup set, use duplexing.

Example to disk:

```rman
BACKUP DEVICE TYPE DISK COPIES 2
  DATABASE
  FORMAT '/disk1/%U', '/disk2/%U';
```

Example to tape:

```rman
BACKUP DEVICE TYPE sbt COPIES 2
  INCREMENTAL LEVEL 0 DATABASE;
```

Important rules from Oracle:

- duplexing applies to **backup sets**, not image copies
- RMAN can duplex to disk or to tape, but not both at the same time
- you cannot duplex into the Fast Recovery Area
- with tape, the number of copies should not exceed the number of available tape
  devices

This does **not** create separate logical backups with unrelated identities. It
creates one backup set with multiple identical backup-piece copies. RMAN then
uses failover behavior if the first copy is unavailable.

### Demo-style duplexed backup flow

The demo version of this is the "make me two disk copies of the same backup
set" pattern.

Representative example:

```rman
BACKUP AS BACKUPSET COPIES 2
  FORMAT '/home/oracle/%U'
  INCREMENTAL LEVEL 0
  DATABASE;
```

Important practical notes:

- `AS BACKUPSET` is still the default, so this is being explicit rather than
  revolutionary
- because duplexed backup sets cannot be written to the FRA, you must provide a
  real destination with `FORMAT`
- the output backup pieces show separate copy numbers, which is how you spot the
  duplexed pieces in the RMAN output and on disk

So after the job completes, you look at the piece handles and see the same
backup-set identity represented by multiple copy numbers instead of one lonely
artifact praying not to be dropped.

### Backing up backup sets to tape

Another pattern is to create backup sets on disk first, and then copy those
backup sets to tape later:

```rman
BACKUP DEVICE TYPE DISK AS BACKUPSET DATABASE PLUS ARCHIVELOG;
BACKUP DEVICE TYPE sbt BACKUPSET ALL;
```

That is a common "disk first, tape later" flow.

It is also much less dramatic than trying to discover, mid-crisis, that your
only copy lives on a tape workflow nobody has tested since a different
government was in office.

### Demo-style backup-of-backupset flow

The practical version looks like this:

1. create a normal backup set
2. issue a second RMAN command that backs up the existing backup sets

Example:

```rman
BACKUP AS BACKUPSET DATABASE PLUS ARCHIVELOG;
BACKUP BACKUPSET ALL;
```

That second command is not backing up the live database again. It is backing up
the backup sets that already exist in the RMAN repository.

One useful behavior from the demo:

- if RMAN sees that a backup piece already has the needed backup-of-backupset
  copy, it can skip that piece instead of producing yet another redundant copy

So the output may include messages showing that some backup-piece handles were
not copied again because RMAN judged the extra copy unnecessary.

Which is nice, because even Oracle occasionally resists the urge to duplicate
things forever.

### Archival backups with KEEP

Sometimes the retention policy is too short for a specific backup. RMAN lets you
mark that backup as archival with `KEEP`.

Examples:

```rman
BACKUP DATABASE KEEP FOREVER TAG 'PRE_UPGRADE_SNAPSHOT';
BACKUP DATABASE KEEP UNTIL TIME 'SYSDATE+30' TAG 'MONTH_END_HOLD';
BACKUP DATABASE KEEP UNTIL RESTORE POINT before_app_cutover TAG 'APP_CUTOVER';
```

And if you later want to remove the archival status:

```rman
CHANGE BACKUP TAG 'PRE_UPGRADE_SNAPSHOT' NOKEEP;
```

Important Oracle nuance:

- `KEEP FOREVER` requires a recovery catalog
- `KEEP` backups are exempt from the normal retention policy
- when `KEEP` is used with incrementals, the parent backup must also be a
  `KEEP` backup with the same tag

This is how you preserve a special backup for legal, testing, upgrade, or
"because we know this weekend is going to be cursed" reasons.

### Demo-style archival backup flow

The demo version uses both the target database and a recovery catalog so the
database can stay open while the archival backup is created.

Representative idea:

```text
rman target "sysbackup@target" catalog rcat_owner@rcat
```

Then create the archival backup in a non-FRA location, with a tag and optionally
with a restore point reference:

```rman
BACKUP
  FORMAT '/home/oracle/%U'
  TAG 'KEEPDB'
  KEEP FOREVER
  RESTORE POINT keepdb2
  DATABASE;
```

Important practical points:

- do **not** write `KEEP` backups into the FRA
- give them an explicit destination with `FORMAT`
- use a clear tag so later listings do not look like anonymous backup soup

During execution, RMAN marks the backup as one that will not become obsolete
under the normal retention policy.

If you want to inspect restore points later, the demo uses:

```rman
LIST RESTORE POINT ALL;
LIST RESTORE POINT keepdb2;
```

That gives you a cleaner handle on the point-in-time reference than muttering,
"it was probably from Tuesday."

### Practice-style archival backup notes

The practice intentionally shows the less glamorous no-catalog path first.

What it demonstrates:

- `KEEP FOREVER` fails without a recovery catalog
- `KEEP` backups cannot live in the FRA
- if you are not using a recovery catalog in the lab flow, you mount the
  database and write the archival backup to a normal filesystem location

Representative example from that path:

```rman
BACKUP AS COPY DATABASE
  FORMAT '/home/oracle/backup/%U'
  TAG 'KEEPDB'
  KEEP UNTIL TIME 'SYSDATE+365';
```

So the practical hierarchy is:

- recovery catalog = needed for `KEEP FOREVER`
- no recovery catalog = use a time-bounded `KEEP UNTIL ...` strategy instead
- FRA = not the place for archival backups you intend to preserve outside normal
  cleanup behavior

### Backing up the Fast Recovery Area

If you use the FRA heavily, you may want to offload its contents explicitly:

```rman
BACKUP RECOVERY AREA TO DESTINATION '/safe_backup/fra_stage';
```

Oracle documents this as backing up recovery files from the current and previous
FRA destinations. That includes items such as:

- backup sets
- control file autobackups
- archived redo logs
- datafile copies

But not flashback logs. Oracle does **not** back up flashback logs with
`BACKUP RECOVERY AREA`.

If you want RMAN to back up **all** recovery files, not just the subset needed
for restore/recovery logic, Oracle also supports:

```rman
BACKUP RECOVERY FILES TO DESTINATION '/safe_backup/fra_everything';
```

Demo-style practical note:

- if you try to back up the recovery area to disk without giving RMAN another
  destination, RMAN complains
- which is fair, because "please back up this area into itself" is not really a
  backup strategy so much as an argument with physics

So yes, the FRA is important. No, it is not a magical vault that backs itself up
simply by being emotionally significant.

---

## 15. Practical Strategy Notes

1. Always pair data backup design with archive-log availability.
2. Use tags consistently (`DAILY_INC`, etc.) for operational clarity.
3. Choose backup set vs image copy based on restore-time objectives, not habit.
4. For multitenant systems, always be explicit about scope (root vs PDB).
5. Enable block change tracking if incremental backups are part of the design.
6. Keep control file protection layered: autobackup plus trace when structural changes happen.
7. Understand repository aging if you rely only on the control file.
8. Validate restore paths regularly, not just backup completion.

---

## 16. Wrap-Up

You now have the command-level map for creating database backups with RMAN across:

- whole database scope
- partial PDB/tablespace scope
- backup set and image copy formats
- incremental roll-forward image copy workflow
- fast incremental backups with block change tracking
- control file trace backup options
- RMAN cataloging and reporting

At this point, backup creation is no longer just "run `BACKUP DATABASE` and hope
for the best." It is a full strategy question involving speed, metadata,
retention, recoverability, and your willingness to test the whole thing before
disaster turns it into a very expensive scavenger hunt.
