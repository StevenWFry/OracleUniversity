## Lesson 21 - Performing Point-in-Time Recovery (in which Oracle lets you go back in time, but only if you know exactly when you stopped making good decisions)

And look, point-in-time recovery sounds heroic until you realize it is mostly a precision exercise in choosing exactly which past version of reality you are willing to keep and which later transactions you are willing to throw into the sea.

By the end of this lesson, you should be able to:

- Distinguish database PITR, tablespace PITR, and table recovery
- Choose a safe recovery target using time, SCN, sequence, or restore point
- Understand when an auxiliary instance is required and when it is not
- Plan for dependencies and object loss during TSPITR
- Use RMAN-managed auxiliary destinations correctly
- Understand the extra rules for PDB point-in-time recovery and table recovery

---

## 1. What Point-in-Time Recovery Actually Means

Point-in-time recovery (`PITR`) means restoring and recovering data **only up to
a chosen past point**, rather than to the latest possible committed state.

Oracle supports PITR at several scopes:

- whole database
- pluggable database
- tablespace
- table

The reason you use it is usually logical damage:

- bad batch job
- bad update or delete
- wrong DDL
- application mistake

So PITR is not the tool for "the disk exploded and I want current data."
It is the tool for:

- "the system kept running, and that was the problem."

---

## 2. The Four Key Terms

### Target time

The time, SCN, sequence, or restore point you want to recover **to**.

### Recovery set

The files or tablespaces you actually intend to recover.

### Auxiliary database / instance

A temporary database environment RMAN creates or uses for operations that cannot
be done purely in-place.

### Auxiliary destination

The disk location used to hold the temporary auxiliary files when RMAN manages
the auxiliary instance for you.

These terms matter because Oracle uses them very precisely, and PITR is not the
moment to interpret "recovery set" as "whatever feels emotionally relevant."

---

## 3. One Important Correction: Not Every PITR Uses an Auxiliary Instance

The source material tends to talk as if PITR always builds a little auxiliary
database in the `FRA`.

That is not true.

### Whole database PITR (`DBPITR`)

For whole-database PITR, you typically:

- mount the target database
- set the recovery target
- restore/recover the database in place
- open with `RESETLOGS`

No auxiliary instance is required for ordinary whole-database PITR.

### Auxiliary instance is used for:

- tablespace point-in-time recovery (`TSPITR`)
- table recovery from RMAN backups
- PDB PITR in multitenant environments when shared root/undo context must be
  reconstructed

This distinction matters a lot, because otherwise Oracle's recovery architecture
starts sounding like it builds little emergency databases for every minor mood
swing.

---

## 4. Choosing the Recovery Target

You can choose the PITR target by:

- time
- SCN
- log sequence and thread
- restore point

Examples:

```rman
SET UNTIL TIME "TO_DATE('2026-03-20 14:15:00','YYYY-MM-DD HH24:MI:SS')";
SET UNTIL SCN 123456789;
SET UNTIL SEQUENCE 420 THREAD 1;
```

And in the `RECOVER` command itself, Oracle supports:

- `UNTIL TIME`
- `UNTIL SCN`
- `UNTIL SEQUENCE`
- `TO RESTORE POINT`

If you use time-based recovery, remember:

- RMAN interprets time strings according to Oracle time-format rules
- `NLS_DATE_FORMAT` and explicit `TO_DATE` formatting matter

So yes, set your environment or specify the format clearly.
Because "sometime around 4 p.m." is not valid RMAN syntax, sadly.

---

## 5. Restore Points: Named SCNs for Humans

Restore points exist because raw SCN numbers are terrible at being memorable.

Examples:

```sql
CREATE RESTORE POINT before_mods;
CREATE RESTORE POINT end_quarter1 AS OF SCN 100;
```

These give you named targets for later PITR or flashback operations.

They are useful because:

- they simplify target selection
- they reduce the chance of using the wrong SCN

But keep the concept clean:

- a restore point names a target
- it does **not** replace the restore and recovery mechanics

In other words, it is a bookmark, not a teleportation spell.

---

## 6. Whole Database PITR (`DBPITR`)

Whole-database PITR is really incomplete recovery of the entire database to a
past point.

Typical RMAN pattern:

```rman
STARTUP MOUNT;
RUN {
  SET UNTIL TIME "TO_DATE('2026-03-20 14:15:00','YYYY-MM-DD HH24:MI:SS')";
  RESTORE DATABASE;
  RECOVER DATABASE;
}
ALTER DATABASE OPEN RESETLOGS;
```

Key consequences:

- all PDBs in the CDB move back with the database if you recover the whole CDB
- everything after the target time is lost from the new incarnation
- `RESETLOGS` is required

Important correction to the source:

- opening the database read-only first to "check the time point" is not the
  standard RMAN DBPITR workflow Oracle documents
- the correct planning discipline is to identify the target properly **before**
  running the recovery

Because doing a whole DBPITR twice because your target was sloppy is a very
educational way to spend an afternoon.

---

## 7. When TSPITR Is the Better Answer

Tablespace point-in-time recovery (`TSPITR`) is what you use when:

- the damage is limited to one or more tablespaces
- you do **not** want to rewind the whole database
- you want to isolate recovery scope

Typical reasons:

- bad batch job hit one application tablespace
- user dropped or corrupted data in one region of the schema
- logical damage is confined enough that full DBPITR would be excessive

This is why TSPITR can be great:

- minimal effect on the rest of the database
- narrower blast radius

But it comes with planning requirements, because Oracle will absolutely punish
you for pretending tablespaces are socially isolated when they are not.

---

## 8. TSPITR Architecture

TSPITR uses an auxiliary instance.

Why?

Because RMAN needs a temporary environment to:

1. restore the recovery set
2. restore the required auxiliary-set files
3. recover to the target time
4. export and import the metadata needed to reintegrate the recovered
   tablespaces

Oracle's terminology:

- **recovery set**
  the tablespaces you want to recover
- **auxiliary set**
  supporting files needed to make recovery work, such as:
  `SYSTEM`, `SYSAUX`, undo, temp, control file, archived redo, and auxiliary
  online redo

Important consequence:

- the auxiliary instance is temporary
- if RMAN manages it, RMAN cleans it up afterward

So this tiny recovery side-database is real, but unlike your ordinary mistakes,
it is designed to go away when the work is done.

---

## 9. Dependency Checks Before TSPITR

Before TSPITR, make sure the tablespace set is self-contained.

Use:

```sql
EXEC DBMS_TTS.TRANSPORT_SET_CHECK('USERS,EXAMPLE', TRUE, TRUE);
SELECT * FROM TRANSPORT_SET_VIOLATIONS;
```

Important Oracle point:

- for TSPITR, use the strict/full dependency check

Why?

Because if objects in the recovery set depend on objects outside the recovery
set, or vice versa, then rolling back only part of the structure can create
inconsistency.

This is exactly the kind of thing that seems theoretical until you recover a
table and discover its index, constraint, or related object stayed behind in the
future like a divorced timeline.

---

## 10. Objects That Will Be Lost in TSPITR

Objects created after the target time in the affected tablespaces will be lost.

Oracle gives you a view for this:

```sql
SELECT owner, name, creation_time, tablespace_name
FROM   TS_PITR_OBJECTS_TO_BE_DROPPED;
```

Use that view to identify objects you may want to preserve separately before the
recovery.

The usual strategy is:

- export what would otherwise be lost
- perform TSPITR
- import the needed objects afterward

Which is much better than discovering after the recovery that the object you
needed was created twenty minutes after your target time and is now cosmically
unavailable.

---

## 11. RMAN-Managed TSPITR

This is the recommended way.

Basic form:

```rman
RECOVER TABLESPACE users, example
  UNTIL TIME "TO_DATE('2026-03-20 14:15:00','YYYY-MM-DD HH24:MI:SS')"
  AUXILIARY DESTINATION '/u01/app/oracle/oradata/aux';
```

If you omit `AUXILIARY DESTINATION` and an `FRA` exists, Oracle uses the `FRA`
for the temporary auxiliary files.

This is the source's "default FRA" idea, but it applies to **auxiliary-based**
recovery workflows such as TSPITR and table recovery, not to every restore
operation in the universe.

You can also customize:

- auxiliary channels
- `SET NEWNAME`
- parameter values for the auxiliary instance

But the more you customize, the more Oracle politely hands the complexity back
to you.

---

## 12. User-Managed Auxiliary Instance for TSPITR

Oracle still supports building your own auxiliary instance manually.

That means:

- create password file
- create parameter file
- configure network connectivity
- start auxiliary instance in `NOMOUNT`
- connect RMAN to target and auxiliary
- run the recovery manually

This is supported.
It is also the sort of thing Oracle documents partly so that people who still do
it can continue being proud of surviving it.

The recommended option is RMAN-managed auxiliary setup unless you have a very
specific reason to take the scenic route through suffering.

---

## 13. PDB Point-in-Time Recovery

PDB PITR is a special case in multitenant.

Important Oracle facts:

- PITR for a PDB must be performed with RMAN
- the PDB must be closed
- RMAN uses an auxiliary destination for the temporary recovery environment
- if an `FRA` exists, it is the default auxiliary destination

Representative pattern:

```sql
ALTER PLUGGABLE DATABASE pdb1 CLOSE IMMEDIATE;
```

Then in RMAN:

```rman
RUN {
  SET UNTIL SCN 123456789;
  RECOVER PLUGGABLE DATABASE pdb1
    AUXILIARY DESTINATION '/u01/app/oracle/oradata/aux';
}
```

Then:

```sql
ALTER PLUGGABLE DATABASE pdb1 OPEN RESETLOGS;
```

Important correction to the source:

- the command is a `RECOVER PLUGGABLE DATABASE` workflow, not simply
  `RESTORE PLUGGABLE DATABASE` followed by ordinary complete-recovery thinking
- `RESETLOGS` for the PDB matters because the PDB gets a new incarnation

So yes, the PDB has its own little timeline drama inside the larger CDB story.

---

## 14. Table Recovery from RMAN Backups

Table recovery is for cases where:

- Flashback Table is not available or not sufficient
- the damage is limited to one or a few tables
- you do not want whole-database or tablespace PITR

Oracle uses:

- an auxiliary instance
- RMAN restore/recover of the needed data
- Data Pump export/import to move the recovered table back into the target

Prerequisites include:

- target database in `READ WRITE`
- target database in `ARCHIVELOG`
- suitable backups available

Representative RMAN pattern:

```rman
RECOVER TABLE hr.employees
  UNTIL TIME 'SYSDATE-1'
  AUXILIARY DESTINATION '/tmp/recover'
  DATAPUMP DESTINATION '/tmp/dpump';
```

Optional clauses include:

- `NOTABLEIMPORT`
- `REMAP TABLE`
- `REMAP TABLESPACE`
- `OF PLUGGABLE DATABASE pdb_name`

If you use `NOTABLEIMPORT`, RMAN stops after producing the export dump and does
not import the recovered object back automatically.

That is helpful when you want to inspect or remap things first instead of
letting Oracle helpfully drop the recovered table right back into traffic.

---

## 15. Table Recovery Limitations

The source gets the spirit right, but the specifics matter.

Practical limits include:

- target database must be `READ WRITE`
- target database must be in `ARCHIVELOG`
- recovered tables cannot be in `SYS`-owned system areas in the way ordinary
  user tables are
- if a table of the same name already exists, you may need `REMAP TABLE`
- dependencies such as indexes, constraints, and related objects need thought

If you remap tables or skip import, you may need to recreate related structures
manually afterward.

So table recovery is wonderfully precise.
It is also a bit of a paperwork festival.

---

## 16. Time-Target Selection Techniques

To find the right target time, use evidence such as:

- alert log timestamps
- application logs
- audit or business timestamps
- Flashback Query
- Flashback Version Query
- Flashback Transaction Query
- known restore points

The source is correct that this investigative step matters enormously.

The entire success of PITR depends on one very annoying truth:

- the recovery command can be perfect
- and still be wrong if your target is wrong

Which is why "close enough" is not really a database time-travel philosophy.

---

## 17. Common Problems in TSPITR and Table Recovery

### File name conflicts

If your auxiliary or recovered-file naming collides with existing files, the
operation can fail.

Avoid this with:

- unique auxiliary destination paths
- careful `SET NEWNAME`
- careful file naming conventions

### Missing dependent tablespaces

If the recovery set is not self-contained, TSPITR can fail or produce
inconsistent results.

### Lost post-target objects

Anything created after the target time in the affected scope may vanish unless
you export and preserve it first.

### Wrong target selection

Still the reigning champion of user-induced PITR disappointment.

---

## 18. Practical Takeaways

Use whole DB PITR when:

- the entire database must go back

Use PDB PITR when:

- one PDB must go back without rewinding the whole CDB

Use TSPITR when:

- damage is isolated to one or more tablespaces and dependencies are manageable

Use table recovery when:

- only one or a few tables need recovery from backup

And before any of them:

1. determine the target carefully
2. validate dependencies
3. understand what will be lost
4. know where your auxiliary instance will live

Because PITR is not hard mainly because the commands are hard.
It is hard because reality is interconnected and Oracle insists that you notice.

---

## 19. Wrap-Up

Point-in-time recovery is not one thing.
It is a family of recovery methods:

- whole database PITR
- PDB PITR
- tablespace PITR
- table recovery from backups

The right choice depends on:

- scope of damage
- dependencies
- backup availability
- target-time certainty
- willingness to lose everything after that point

And once you understand that, PITR stops looking like a magic rollback button
and starts looking like what it really is: controlled selective time travel with
paperwork, prerequisites, and consequences.

---

## 20. Practice 4.1 - Incomplete Recovery After Media Failure

The practice for this lesson sets up the classic PITR trap:

- a usable backup exists
- archived redo exists for a while after that backup
- one required archived log is then made unavailable
- a data file failure occurs
- complete recovery becomes impossible

So the lab is not just teaching "how to restore a file."
It is teaching:

- how to notice that complete recovery has failed, and
- how to pivot into incomplete recovery cleanly

### What the lab does

At a high level, the practice:

1. ensures the recovery catalog is available and synchronized
2. creates a backup baseline
3. runs setup scripts that create and modify application data
4. removes or damages a data file
5. confirms that the `PDB` will not open
6. restores the missing file
7. attempts recovery
8. discovers that a required archived log is missing
9. performs incomplete recovery to an earlier sequence/SCN
10. opens the database with `RESETLOGS`
11. reopens the affected `PDB`
12. verifies application data and backup metadata afterward

That is a very realistic failure pattern, because real recovery work often
starts as one thing and then mutates into another after the first serious error
message.

---

## 21. Reading the Failure Correctly

The lab first shows a `PDB` failing to open because a data file is missing.

That part is straightforward:

- Oracle can mount the container structure
- Oracle cannot open the affected `PDB`
- the missing data file number and file name matter

The optional trace-file check is also useful.
If the trace or alert output shows `DBWR` failing to access the file, then the
database is not being subtle about the problem.

Historically, the practice also uses:

```rman
LIST FAILURE;
ADVISE FAILURE;
```

That remains useful as legacy context, but remember the `19c` correction:

- Data Recovery Advisor is deprecated
- it is not the modern center of gravity for this recovery

The operationally important part is what comes next:

- `RESTORE DATAFILE n`
- `RECOVER DATAFILE n`
- observe that recovery cannot continue because a required archived log is gone

That is the point where the lab stops being a data-file recovery exercise and
becomes a point-in-time recovery exercise.

---

## 22. Determining the Recovery Boundary

The practice then gathers evidence from:

- `ARCHIVE LOG LIST`
- `V$DATABASE`
- `V$ARCHIVED_LOG`

This is exactly the right instinct.

You are trying to answer three questions:

1. Which archived log sequence is missing?
2. What SCN range did that missing log cover?
3. To what earlier boundary can I still recover safely?

Useful queries include:

```sql
SELECT name, dbid, current_scn, log_mode, open_mode
FROM v$database;
```

```sql
SELECT sequence#, first_change#, first_time, status
FROM v$archived_log
WHERE name IS NOT NULL
ORDER BY sequence#;
```

And yes, the exact values will vary from lab to lab.

That is normal.

If the missing sequence in your environment is `37`, then use `37`.
If the file number is `20`, then use `20`.
Oracle is not grading you on loyalty to the workbook's numerology.

---

## 23. Correct RMAN Pattern for the Incomplete Recovery

The source material shows the right general idea, but the cleaner RMAN pattern
is to restore the control file if needed, mount the database, set the recovery
boundary once, and then let both restore and recover obey that same boundary.

### If you need to restore the control file first

```rman
SHUTDOWN IMMEDIATE;
STARTUP NOMOUNT;
RESTORE CONTROLFILE FROM AUTOBACKUP;
ALTER DATABASE MOUNT;

RUN {
  SET UNTIL SEQUENCE 37 THREAD 1;
  RESTORE DATABASE;
  RECOVER DATABASE;
}

ALTER DATABASE OPEN RESETLOGS;
```

If you prefer `SCN` instead of sequence:

```rman
RUN {
  SET UNTIL SCN 3817001;
  RESTORE DATABASE;
  RECOVER DATABASE;
}
```

This is better than issuing separate commands such as:

- `RESTORE DATABASE UNTIL SEQUENCE ...`
- `RECOVER DATABASE UNTIL SEQUENCE ...`

The reason is simple:

- one boundary
- one recovery story
- fewer opportunities to improvise yourself into a different timeline

Once the `CDB` is open again, reopen the affected `PDB`:

```sql
ALTER PLUGGABLE DATABASE orclpdb1 OPEN;
```

And then verify:

```sql
SHOW PDBS;
```

---

## 24. Why the Control File Restore Can Make Sense Here

The practice notes that restoring the control file is recommended for incomplete
recovery.

That is a defensible approach when:

- you are doing database point-in-time recovery
- you want RMAN working from metadata consistent with the older recovery point
- you are prepared to roll the whole database back, not just the one `PDB`

What matters is understanding the blast radius.

If you restore the control file and do a whole-database incomplete recovery,
then you are not merely fixing one missing `PDB` file.
You are choosing an earlier history for the database incarnation as a whole.

That is why the lab correctly ends with:

- `ALTER DATABASE OPEN RESETLOGS`
- reopening the `PDB`
- validating the data afterward

---

## 25. Post-Recovery Cleanup Matters

The lab finishes with several practical housekeeping steps:

- `LIST FAILURE` to confirm there are no remaining registered failures
- `CROSSCHECK ARCHIVELOG ALL`
- `DELETE NOPROMPT OBSOLETE`
- application-level verification queries
- cleanup scripts for the lab environment

These are good habits.

The most important post-recovery step, however, is the final backup.

After `OPEN RESETLOGS`:

- you have a new incarnation
- your recovery baseline has changed
- you should create a fresh backup as soon as practical

The source is absolutely right about that.

Old backups may still exist and may still matter historically, but your new
operational recovery plan should be anchored in the new incarnation, not in your
fond memories of the old one.

---

## 26. What This Practice Actually Teaches

Practice 4.1 is valuable because it forces you through the decision point that
matters:

- if the missing file can be restored and recovered fully, do complete recovery
- if the required archived redo is missing, stop pretending and move to PITR

It also reinforces several operational truths:

- file numbers and log sequences are environment-specific
- `PDB` symptoms can still lead to `CDB`-scope recovery decisions
- a successful `RESTORE DATAFILE` does not mean recovery will succeed
- missing redo, not missing backups, is often what turns a repair into a
  rollback

And that is the real DBA skill here.
Not typing the command.
Knowing when the category of the problem has changed.

---

## 27. Practice 4.2 - Recovering a Table from a Backup

The second practice for this lesson moves from whole-database drama to a much
more civilized form of damage control:

- one table is created
- rows are committed
- a known SCN is captured
- the table is purged
- RMAN recovers that table from backup without rewinding the rest of the schema

This is exactly the kind of problem table recovery was built for.

It is not:

- "rewind the whole database"

It is:

- "bring back this one stupidly deleted object without setting fire to the
  neighborhood"

---

## 28. What the Practice Is Setting Up

The lab starts by preparing the environment:

- setup script creates the practice user and tablespace
- `SQL*Plus` confirms the database is `READ WRITE`
- `SQL*Plus` confirms the database is in `ARCHIVELOG`
- `SHOW PARAMETER COMPATIBLE` confirms the feature is supported
- the session is switched into `orclpdb1`
- the initial practice objects are verified

That matches Oracle's documented prerequisites for table recovery from RMAN
backups.

You need:

- target database open `READ WRITE`
- target database in `ARCHIVELOG`
- compatible backup coverage for the time you want
- RMAN connected locally to the target

The lab then creates a backup chain:

- level 0 backup plus archived logs
- later level 1 backup plus archived logs

That gives RMAN the history it needs to reconstruct the table to the earlier
SCN.

---

## 29. Capture the SCN Before You Break the Table

This is the key discipline in the practice:

1. create and populate the test table
2. commit
3. capture the SCN immediately after that known-good state

That SCN becomes the recovery target.

The transcript flips between names like:

- `bar.testtable`
- `bar.test_table`
- `bar.test`

Do not imitate that chaos.

Use the actual table name created by the lab script in your environment.
From the way the verification commands are described, the intended object is
most likely:

```sql
BAR.TEST_TABLE
```

Then the lab deliberately removes it:

```sql
DROP TABLE bar.test_table PURGE;
```

Purging matters because now you are not using the recycle bin as a safety net.
You are forcing an actual recovery exercise.

---

## 30. Correct RMAN Pattern for the Table Recovery

Oracle's documented `19c` syntax for PDB table recovery is:

```rman
RECOVER TABLE schema.table_name
  OF PLUGGABLE DATABASE pdb_name
  UNTIL SCN scn_value
  AUXILIARY DESTINATION '/path/to/aux';
```

Applied to the practice, the command should conceptually look like:

```rman
RECOVER TABLE bar.test_table
  OF PLUGGABLE DATABASE orclpdb1
  UNTIL SCN 3817001
  AUXILIARY DESTINATION '/u01/app/oracle/backup/test';
```

Use your real SCN, not the example one above.

Important details:

- `AUXILIARY DESTINATION` is required
- RMAN creates and later removes the temporary auxiliary environment
- RMAN creates a Data Pump export internally during the process
- by default, RMAN imports the recovered table back into the target database

That last point matters because the lab does **not** need `NOTABLEIMPORT`.

It wants the table back in place automatically.

Because the original table was purged, there is normally no name conflict.
So this practice also does **not** need `REMAP TABLE` unless your environment
already contains an object with the same name.

Practical note:

- the auxiliary destination should exist and be writable
- it is wise to keep it empty or dedicated for the recovery operation

The lab's `HOST ls ...` check is basically a crude but perfectly serviceable way
of making sure you are not about to recover into a directory full of unrelated
junk and then act surprised when things become interpretive.

---

## 31. What RMAN Is Doing Behind the Curtain

When you run the `RECOVER TABLE` command, RMAN is doing much more than restoring
one table-shaped file, because tables are not stored as tiny self-contained
souvenirs.

RMAN:

1. identifies the backups required for the specified SCN
2. creates an auxiliary database environment
3. restores the supporting files needed for that environment
4. recovers the auxiliary copy to the requested point in time
5. exports the recovered table using Data Pump
6. imports the table back into the target database
7. removes the auxiliary environment when finished

So yes, table recovery is targeted.
It is not small.

It is Oracle doing a surprisingly elaborate amount of backstage labor so that
you can type one command and pretend this was always going to be simple.

---

## 32. Post-Recovery Verification and Cleanup

After RMAN finishes, the practice returns to `SQL*Plus` and verifies that the
rows are back:

```sql
SELECT * FROM bar.test_table;
```

That verification is the entire point.

Do not stop at "RMAN completed successfully."
Successful object recovery means:

- the table exists
- the rows are present
- the contents match the intended pre-drop state

The lab then cleans up with:

- obsolete-backup deletion in RMAN
- cleanup script for temporary practice artifacts

If you are mirroring the lab exactly, the RMAN cleanup command should be the
explicit form:

```rman
DELETE NOPROMPT OBSOLETE;
```

Not merely:

```rman
DELETE NOPROMPT;
```

because vague deletion commands are how people eventually create exciting new
recovery lessons.

---

## 33. What This Practice Actually Teaches

Practice 4.2 is useful because it shows the sweet spot for RMAN table recovery.

Use it when:

- one table is gone
- you know the approximate target SCN or time
- you do not want to rewind the tablespace or database
- Flashback Table is unavailable, insufficient, or not appropriate

It also teaches three good habits:

- capture the recovery target before destructive testing
- keep your table names and SCNs straight instead of trusting the workbook's
  improv comedy
- verify restored data in SQL, not just in RMAN output

And that is why table recovery is such a good feature.
It lets you surgically repair one object instead of punishing the entire
database for one bad decision.
