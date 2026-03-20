## Lesson 20 - Performing Complete Recovery (in which RMAN does the heavy lifting, provided you gave it the backups instead of wishful thinking)

And look, complete recovery sounds wonderfully reassuring until Oracle reminds you that "complete" depends on backup availability, redo availability, database mode, object scope, and whether the thing you lost was merely inconvenient or structurally catastrophic.

By the end of this lesson, you should be able to:

- Use RMAN commands to preview and validate a restore plan
- Distinguish complete recovery in `ARCHIVELOG` mode from limited recovery in `NOARCHIVELOG`
- Recover critical and non-critical data files correctly
- Use image copies and `SWITCH` for faster recovery
- Restore files to new locations with `SET NEWNAME`
- Understand the CDB/PDB-specific differences in complete recovery workflows

---

## 1. Before You Recover, Prove You Can

Before running a real restore, you want Oracle to tell you whether the required
backups are actually available.

That is where RMAN preview and validate options help.

### `RESTORE ... PREVIEW`

Use:

```rman
RESTORE DATABASE PREVIEW;
```

What it does:

- lists the backups RMAN would use
- shows the target SCN needed for recovery after restore
- consults RMAN metadata only
- does **not** actually read the backup pieces

Important correction to the source:

- `PREVIEW` does **not** give you a trustworthy "this will take X minutes"
  stopwatch estimate
- it tells you **what** RMAN plans to use, not how long your storage will sulk

### `RESTORE ... VALIDATE HEADER`

Use:

```rman
RESTORE DATABASE VALIDATE HEADER;
```

What it does:

- lists the files needed for restore and recovery
- validates backup file headers
- checks whether files on disk or in the media manager catalog correspond to the
  RMAN repository metadata

This is a metadata/header sanity check, not a full media read of every block.

### `RESTORE ... VALIDATE`

Use:

```rman
RESTORE DATABASE VALIDATE;
```

What it does:

- tests whether RMAN can actually restore the requested object
- reads backup contents and checks for corruption

This is the stronger validation.

So the correct ladder is:

- `PREVIEW` = what would RMAN use?
- `VALIDATE HEADER` = do the headers and repository line up?
- `VALIDATE` = can RMAN really read and restore the backup?

Which is much better than discovering at recovery time that your "backup
strategy" was actually a decorative belief system.

---

## 2. A Small but Important Correction About "Recover Validate Header"

The source blends together a few ideas here.

For planning and validation, Oracle's real tools are:

- `RESTORE ... PREVIEW`
- `RESTORE ... VALIDATE HEADER`
- `RESTORE ... VALIDATE`
- data dictionary / dynamic view checks such as `V$DATAFILE_HEADER` and
  `V$RECOVER_FILE`

So if the lecture sounds like there is a standalone RMAN command called
something like "recover validate header," treat that as spoken shorthand, not as
syntax you should go type with confidence.

Useful SQL checks:

```sql
SELECT file#, status, recover, fuzzy
FROM   v$datafile_header;

SELECT file#, error, online_status, change#, time
FROM   v$recover_file;
```

---

## 3. `NOARCHIVELOG` Mode: This Is Not the Happy Path

If the database runs in `NOARCHIVELOG` mode, your recovery options are sharply
limited.

What that means:

- you cannot recover to the latest committed point after the backup
- you can recover only to the end of the last **consistent** backup chain you
  possess

If you use incrementals in `NOARCHIVELOG` mode, RMAN can still apply:

- a consistent level `0`
- followed by consistent level `1` incrementals

But this is still limited recovery.

It is not "complete recovery to the crash moment."

Important Oracle point:

- in `NOARCHIVELOG`, backups must be consistent
- that means the database must be shut down cleanly for those backups

So yes, `NOARCHIVELOG` mode is cheaper right up until you need it not to be.

---

## 4. Recovering a `NOARCHIVELOG` Database with Incrementals

If media failure destroys data files and online redo logs in a `NOARCHIVELOG`
database, Oracle documents this pattern:

```rman
STARTUP FORCE NOMOUNT;
RESTORE CONTROLFILE;
ALTER DATABASE MOUNT;
RESTORE DATABASE;
RECOVER DATABASE NOREDO;
ALTER DATABASE OPEN RESETLOGS;
```

Why `NOREDO` matters:

- the online redo logs are gone
- RMAN must not keep searching for redo that no longer exists

What the result means:

- the database is recovered only to the point of the last usable backup chain
- everything after that is lost and must be re-entered

So if someone calls this "complete recovery" in casual speech, translate it in
your head as:

- "the most complete recovery possible in a bad design choice"

---

## 5. `ARCHIVELOG` Mode: Real Complete Recovery

This is the good version.

In `ARCHIVELOG` mode, if you have:

- the required backup
- the required archived redo logs
- and, if needed, the online redo logs

then you can usually recover up to the point of the last committed transaction.

This is what most people mean by complete recovery:

- restore from backup
- apply redo until current consistency is reached
- users do not have to re-enter already committed data

Which is why `ARCHIVELOG` mode exists, and also why DBAs keep recommending it
instead of acting enthusiastic about your minimalist backup philosophy.

---

## 6. Recovering a Non-Critical File While the Database Stays Open

If you lose a non-critical data file, and it is **not** in:

- `SYSTEM`
- `UNDO`

then you can often recover it without taking the whole database down.

Typical pattern:

```sql
ALTER TABLESPACE users OFFLINE IMMEDIATE;
```

Then in RMAN:

```rman
RESTORE TABLESPACE users;
RECOVER TABLESPACE users;
```

Then:

```sql
ALTER TABLESPACE users ONLINE;
```

This is the civilized version of recovery:

- isolate damage
- restore just what is needed
- keep the rest of the database available

Far better than flattening the entire system because one application tablespace
had a public meltdown.

---

## 7. Recovering a Critical File

If the lost file is critical, such as a `SYSTEM` data file, then you generally
do not get to keep the database cheerfully open.

The typical sequence is:

1. if the instance is still limping along, `SHUTDOWN ABORT`
2. `STARTUP MOUNT`
3. restore the damaged file
4. recover it
5. open the database
6. reopen the PDBs if needed

Representative RMAN/SQL sequence:

```rman
SHUTDOWN ABORT;
STARTUP MOUNT;
RESTORE DATAFILE 1;
RECOVER DATAFILE 1;
ALTER DATABASE OPEN;
ALTER PLUGGABLE DATABASE ALL OPEN;
```

Important correction to the source's narrative stumble:

- if `SHUTDOWN IMMEDIATE` fails because a critical file is missing, that is not
  Oracle being moody
- it is Oracle refusing to pretend the database can become consistent while one
  of its core files is gone

At that point, `ABORT` is the correct ugly answer.

---

## 8. Image Copies and Why They Can Make Recovery Fast

Image copies are bit-for-bit copies of your data files.

That matters because they can be used for rapid switch-based recovery.

If you maintain an incrementally updated image copy, then in some failure
scenarios RMAN can switch the database to use the copy rather than spending time
rebuilding the file from backup pieces.

That is where this strategy starts feeling less like backup theory and more like
practical wizardry.

### The rolling image-copy pattern

You already saw the backup side of this in the earlier lesson:

```rman
RECOVER COPY OF DATABASE WITH TAG 'DAILY_INCREMENTAL';
BACKUP INCREMENTAL LEVEL 1 FOR RECOVER OF COPY WITH TAG 'DAILY_INCREMENTAL' DATABASE;
```

Operationally:

- day 1 creates the base image copy
- later days apply yesterday's incremental and create the next one

So the copy stays close to current without requiring a full new copy every day.

---

## 9. Fast Recovery with `SWITCH TO COPY`

If a data file is lost and you already have a usable image copy, RMAN can switch
the database to use that copy.

The idea is:

```rman
SWITCH DATAFILE 7 TO COPY;
RECOVER DATAFILE 7;
```

What that means:

- the control file now points to the image copy instead of the original missing
  file
- the data file can be brought back into service quickly

That can be an excellent emergency move.

Then, if you want to put the file back in its long-term preferred location, you
can later:

- restore or copy it back where you really want it
- switch again
- recover

So the image copy becomes your rescue apartment, not necessarily the place the
data file lives forever.

---

## 10. Restoring to a New Location with `SET NEWNAME`

Sometimes you do not want the file restored to its original location.

That is where `SET NEWNAME` helps.

Typical pattern:

```rman
RUN {
  SET NEWNAME FOR DATAFILE 7 TO '/u02/oradata/users01.dbf';
  RESTORE DATAFILE 7;
  SWITCH DATAFILE ALL;
  RECOVER DATAFILE 7;
}
```

This is useful when:

- the original disk is gone
- you are relocating files intentionally
- you need to recover to a safer or more spacious location

So yes, RMAN will restore files where you tell it to.
You just have to stop assuming the original location remains a good life choice.

---

## 11. Restore Points: Useful, But Not Complete Recovery

The source shifts into restore points here, which is related but not really a
complete recovery topic.

A restore point is a named SCN target.

Example:

```sql
CREATE RESTORE POINT before_mods;
CREATE RESTORE POINT end_quarter1 AS OF SCN 100;
```

Why it helps:

- you do not have to remember a raw SCN
- you can refer to a meaningful named target for later point-in-time work

But keep the concepts clean:

- restore points are mainly for **point-in-time recovery** or **flashback**
  workflows
- they are not part of ordinary complete recovery to current time

So yes, they matter.
No, they do not belong in the same mental bucket as "recover to the last
committed change after a media failure."

---

## 12. CDB and PDB Recovery Changes the Rules a Bit

A multitenant environment gives you more scope options, which is helpful and
also mildly rude because it multiplies the number of ways you can be wrong.

### About tempfiles in PDBs

Tempfiles are not backed up like normal data files.

So if a PDB tempfile is missing, the practical response is often:

- re-create it rather than restore it

The source describes cases where opening the PDB can trigger recreation of the
tempfile. Treat that as a convenience that should still be verified, not as
something to assume blindly without checking size and placement afterward.

---

## 13. Recovering `SYSTEM` or `UNDO` in a PDB

If a PDB loses `SYSTEM` or `UNDO` files, this is not a casual online fix.

Oracle's documented pattern is:

1. connect to the root as a common user with `SYSDBA` or `SYSBACKUP`
2. shut down the CDB and start it in `MOUNT`
3. restore and recover the affected PDB data files
4. open the PDBs again

Representative structure:

```rman
SHUTDOWN IMMEDIATE;
STARTUP MOUNT;
RESTORE DATAFILE <pdb_system_file_numbers>;
RECOVER DATAFILE <pdb_system_file_numbers>;
```

Then:

```sql
ALTER PLUGGABLE DATABASE ALL OPEN READ WRITE;
```

So the source is directionally right that this is a whole-PDB-level serious
problem.
But the clean Oracle formulation is:

- you are recovering the affected data files while the CDB is mounted
- not casually "connected to the PDB and restoring everything because vibes"

---

## 14. Recovering Non-`SYSTEM` Data in a PDB

If the damage is limited to a non-`SYSTEM` tablespace or data file in a PDB, the
process is much more like ordinary tablespace/datafile recovery.

Important Oracle rule:

- to recover one or more **tablespaces** in a PDB, connect directly to that PDB
  to avoid ambiguity
- for **datafiles**, you may connect either to the root or the PDB because file
  numbers and paths are unique across the CDB

Typical flow for a non-`SYSTEM` PDB tablespace:

```sql
ALTER TABLESPACE tbs_app OFFLINE IMMEDIATE;
```

Then in RMAN:

```rman
RESTORE TABLESPACE tbs_app;
RECOVER TABLESPACE tbs_app;
```

Then:

```sql
ALTER TABLESPACE tbs_app ONLINE;
```

If connected to the root and you need to specify the PDB-qualified tablespace
name, use the `pdb_name:tablespace_name` form where required.

The core idea remains:

- small failure, small recovery

Which is very comforting compared with the alternative.

---

## 15. Manual Complete-Recovery Demo: Critical Root Data File Loss

The demo in the source is a classic ugly-but-educational case:

- you try normal SQL work
- Oracle errors on the `SYSTEM` data file
- shutdown immediate fails
- you switch to RMAN and do the real work

Representative flow:

```rman
SHUTDOWN ABORT;
STARTUP MOUNT;
RESTORE DATAFILE 1;
RECOVER DATAFILE 1;
ALTER DATABASE OPEN;
ALTER PLUGGABLE DATABASE ALL OPEN;
```

Then verify in SQL*Plus:

```sql
SHOW PDBS;
ALTER SESSION SET CONTAINER = pdb1;
SHOW CON_NAME;
```

That final verification matters.

Because a recovery is not "done" when RMAN stops talking.
It is done when the database and the required PDBs are actually usable again.

---

## 16. Practice Notes: Manual RMAN and Legacy DRA Paths

The practice sequence compares:

- manual RMAN restore/recover of a critical root data file
- a legacy DRA-assisted path for the same kind of failure
- recovery of a non-`SYSTEM` PDB application data file

The important practical lessons are:

1. for a missing `SYSTEM` data file, be prepared for `SHUTDOWN ABORT`
2. a mounted database is the normal repair state for critical file recovery
3. `RESTORE DATAFILE` and `RECOVER DATAFILE` are the core manual commands
4. after opening the CDB, reopen the PDBs explicitly if needed
5. for a missing non-`SYSTEM` PDB data file, targeted restore/recover is enough

And yes, the practice still shows DRA commands.
That belongs to the same bucket as lesson 19:

- historically useful
- still understandable
- not the center of your modern 19c identity

---

## 17. Practical Takeaways

When planning complete recovery, think in this order:

1. `PREVIEW` and validate the restore plan first
2. decide whether you are in `ARCHIVELOG` or `NOARCHIVELOG`
3. determine whether the failed file is critical
4. choose the smallest recovery scope that works
5. use image copies and `SWITCH TO COPY` if speed matters and the copies exist
6. for CDB/PDB environments, be very explicit about whether you are recovering
   root, a whole PDB, a tablespace, or a data file

Because "recovery" is not one command.
It is a sequence of decisions that Oracle will absolutely make you live with.

---

## 18. Wrap-Up

Performing complete recovery with RMAN is really about using the right workflow
for the situation:

- validate first
- restore what is missing
- recover with the redo you have
- open correctly
- verify at the SQL level

And once you add `NOARCHIVELOG`, image copies, `SET NEWNAME`, restore points,
and PDB-specific rules, you stop thinking of recovery as a single action and
start thinking of it as what it actually is: a controlled sequence of getting
Oracle back to the least embarrassing state available.
