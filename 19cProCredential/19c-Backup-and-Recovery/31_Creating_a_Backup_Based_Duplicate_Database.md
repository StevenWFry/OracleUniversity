## Lesson 31 - Creating a Backup-Based Duplicate Database (in which RMAN rebuilds a database from backups, while the source material casually sneaks in a completely different PDB duplication feature and hopes no one notices)

This lesson starts as a discussion of backup-based duplication and then, with
absolutely classic Oracle University energy, drifts into duplicating a live
`PDB` into an existing `CDB`, which is **active duplication**, not backup-based
duplication.

So we are going to separate the two before the syntax starts gaslighting
everyone.

By the end of this lesson, you should be able to:

- explain what backup-based duplication needs before it can start
- prepare the auxiliary instance for a duplicate created from backups
- understand the main file-naming methods used during duplication
- explain what `DUPLICATE` options such as `NOREDO`, `NOOPEN`,
  `OPEN RESTRICTED`, `SKIP READONLY`, and `SKIP TABLESPACE` actually do
- understand when `UNDO TABLESPACE` must be specified
- distinguish duplicating a `PDB` to a **new** `CDB` from duplicating a live
  `PDB` to an **existing** `CDB`

---

## 1. Backup-Based Duplication: What It Really Means

Backup-based duplication creates the duplicate database from:

- existing RMAN backups
- archived redo logs, if needed
- optionally a recovery catalog, depending on connection mode

It is not:

- active duplication from the live source over the network

That distinction matters because the prerequisites, the connections, and even
some valid command clauses change depending on which branch you are in.

Backup-based duplication is useful when:

- backups already exist
- the source system should not be leaned on heavily
- network bandwidth is limited
- the source database is not the thing you want to touch while experimenting

RMAN can do backup-based duplication:

- with a target connection
- without a target connection but with a recovery catalog
- without both target and catalog, using `BACKUP LOCATION`

That last one is the most isolated form.
It is also the one that most aggressively expects you to have prepared the
backup location correctly before showing up.

---

## 2. Required Connections for Backup-Based Duplication

Oracle supports several connection patterns.

### Backup-based duplication with a target connection

Connect to:

- `TARGET`
- `AUXILIARY`

Optional:

- `CATALOG`

### Backup-based duplication without a target connection

Usually connect to:

- `AUXILIARY`
- `CATALOG`

### Backup-based duplication without target and without catalog

Connect to:

- `AUXILIARY`

In this mode, RMAN must rely on:

- `BACKUP LOCATION`
- and, when needed, explicit database identification such as `SET DATABASE`
  and possibly `SET DBID`

Important detail from Oracle:

- for backup-based duplication without a target connection, if the source
  database name is not unique in the recovery catalog, then you must identify
  the source by `DBID`

So no, "just connect to auxiliary and vibe" is not a supported architecture.

---

## 3. Preparing the Auxiliary Instance

The lecture is correct about the broad shape:

- create or prepare a password file if needed
- create a minimal initialization parameter file
- start the auxiliary instance in `NOMOUNT`

But some of the implementation details need cleanup.

### Password file

For backup-based duplication, Oracle allows either:

- operating system authentication
- password file authentication

So a password file is not always mandatory for backup-based duplication.

It **is** required when:

- you are duplicating to a remote host and need password file authentication
- your connection method requires it

And if you want to reuse the source password file:

- copying and renaming it is valid on the same platform

That part of the lesson is fine.

### Oracle Net connectivity

Oracle Net connectivity is required when the duplication mode needs network
connections between source and auxiliary.

If you are doing a more isolated backup-location-driven duplication, the network
dependency may be smaller, but your RMAN client still needs to connect where the
chosen method requires.

### Initialization parameter file

The source says to edit the `PFILE` so it references the new auxiliary service
name.
That is not really the core point.

What the auxiliary initialization file actually needs is database-instance
configuration such as:

- `DB_NAME`
- `CONTROL_FILES`, if you are not using OMF and not letting RMAN handle this
  through copied or restored parameter logic
- file naming or destination parameters such as `DB_FILE_NAME_CONVERT`,
  `LOG_FILE_NAME_CONVERT`, or `DB_CREATE_FILE_DEST`
- `DB_RECOVERY_FILE_DEST` and size, if using an FRA on the duplicate side
- `ENABLE_PLUGGABLE_DATABASE=TRUE`, if the auxiliary must be a `CDB`

The service name belongs to network configuration, not to the conceptual core of
the `PFILE`.

---

## 4. Why the Auxiliary Must Start in NOMOUNT

The auxiliary instance starts in:

- `NOMOUNT`

because it does not yet have:

- a usable control file
- a mounted database to open

That part of the lesson is exactly right.

Typical startup:

```sql
STARTUP NOMOUNT PFILE='$ORACLE_HOME/dbs/initAUX.ora';
```

And yes, if you only have a `PFILE` at this point, that is normal.

The lecture then says you should immediately create an `SPFILE`.
That is not wrong, but it is not universally required as a separate manual step.

RMAN can also:

- restore or copy an `SPFILE` during duplication when you use the `SPFILE`
  clause

So the clean rule is:

- a text `PFILE` is enough to start the auxiliary in `NOMOUNT`
- an `SPFILE` is often desirable or will be produced by the duplication process
- you do not always have to hand-build it first

---

## 5. Making the Backups Accessible

This is one of the most important parts of backup-based duplication.

The auxiliary host must be able to access:

- data file backups
- control file or `SPFILE` backups, if used
- archived redo logs required to recover the duplicate to the desired point

Oracle allows several patterns:

- identical disk paths visible on source and destination
- transferred disk backups
- shared storage such as NFS
- SBT tape backups accessible to the destination host
- explicit `BACKUP LOCATION`

Important Oracle rule:

- the backups do **not** have to come from a single `BACKUP DATABASE` command

You can mix:

- full backups of individual data files
- incremental backups
- archived redo logs

But a full backup of every data file is still required somewhere in that set.
More precisely:

- a usable base backup or image copy of every required data file must exist

Incrementals and archived redo can move that base forward.
They cannot conjure a missing foundation out of professional optimism.

Also, if the source database uses an FRA in ASM, Oracle recommends making a
backup outside that FRA or otherwise making the needed backup pieces reachable
by the auxiliary environment.

---

## 6. Naming Files on the Duplicate Side

This is where the lecture gets into `SET NEWNAME`, `CONFIGURE AUXNAME`, and the
convert parameters.

Those are all real tools, but they are not equal and they are not always the
best first choice.

### Option 1: SET NEWNAME

Use `SET NEWNAME` inside a `RUN` block when you want one-time naming control for
the duplication.

Examples:

```rman
RUN
{
  SET NEWNAME FOR DATABASE TO '/u02/oradata/%U';
  DUPLICATE TARGET DATABASE TO auxdb;
}
```

Or for a specific tablespace:

```rman
SET NEWNAME FOR TABLESPACE users TO '/u02/oradata/%b';
```

This is often the cleanest approach when:

- you want explicit control
- you are doing a one-off duplication
- you want different patterns for different files

### Option 2: CONFIGURE AUXNAME

`CONFIGURE AUXNAME` is persistent and can be used as an alternative to `SET NEWNAME`.

The transcript treats it like a common default.
The better interpretation is:

- it is useful when you want persistent auxiliary naming behavior across
  duplications

Important Oracle note:

- you cannot use `CONFIGURE AUXNAME` for recovery-set files; use `SET NEWNAME`
  there

So this is not dead syntax, but it is also not the universal hammer.

### Option 3: DB_FILE_NAME_CONVERT and LOG_FILE_NAME_CONVERT

These parameters map old path patterns to new path patterns.

Example:

```ini
DB_FILE_NAME_CONVERT=
  ('/u01/app/oracle/oradata/orcl/',
   '/u02/app/oracle/oradata/aux/')

LOG_FILE_NAME_CONVERT=
  ('/u01/app/oracle/oradata/orcl/',
   '/u02/app/oracle/oradata/aux/')
```

This is convenient when:

- the duplicate mostly mirrors the source layout
- only the directory root changes

### Option 4: OMF destinations

If you use Oracle Managed Files, then the simpler answer is often:

- set `DB_CREATE_FILE_DEST`
- set appropriate online redo destinations
- let Oracle generate names

Important Oracle rule:

- do not set `DB_FILE_NAME_CONVERT` if you set `DB_CREATE_FILE_DEST`

Because Oracle does not need two adults fighting over the same filename.

---

## 7. SET NEWNAME Substitution Variables

The lecture's example with `%U`, `%d`, `%I`, `%N`, and `%f` is basically about
building unique file names.

That part is worth remembering.

For example:

- `%U` gives a system-generated unique name
- `%d` is the database name
- `%I` is the `DBID`
- `%N` is the tablespace name
- `%f` is the absolute file number
- `%b` is the file name without the path

Oracle explicitly warns that when you use:

- `SET NEWNAME FOR DATABASE`
- `SET NEWNAME FOR TABLESPACE`

you should include substitution variables to avoid collisions.

So the transcript's general advice is sound:

- use generated naming patterns instead of hand-typing every file name unless
  you truly enjoy self-inflicted maintenance

---

## 8. Channels and RUN Blocks

The lecture says that if you use a `RUN` block, then you have to allocate
auxiliary channels, and if you do not use a `RUN` block, Oracle will allocate
them for you.

That is too simplistic.

The more accurate rule is:

- `DUPLICATE` requires one or more auxiliary channels
- if you do not manually allocate channels, RMAN can use configured channels or
  allocate needed channels automatically depending on the duplication mode and
  configuration
- a `RUN` block is required when you need commands such as `SET NEWNAME`

So a `RUN` block and manual channel allocation are related, but not identical.

For backup-based duplication, a typical pattern might be:

```rman
RUN
{
  ALLOCATE AUXILIARY CHANNEL a1 DEVICE TYPE DISK;
  ALLOCATE AUXILIARY CHANNEL a2 DEVICE TYPE DISK;

  DUPLICATE TARGET DATABASE TO auxdb
    PFILE='/u01/app/oracle/product/19.0.0/dbhome_1/dbs/initAUX.ora'
    UNTIL TIME "SYSDATE-1";
}
```

Allocate more channels only when:

- storage can actually sustain it
- the backup media and restore path benefit from it

Because "more channels" and "more faster" are not always synonyms.

---

## 9. What RMAN Does During Backup-Based Duplication

At a high level, RMAN performs the duplicate by:

1. using the auxiliary instance started in `NOMOUNT`
2. obtaining metadata from the target connection, recovery catalog, backup
   location, or some combination of those
3. restoring the required files for the duplicate
4. recovering the duplicate using incrementals and archived redo as needed
5. opening the duplicate database in `RESETLOGS` mode by default

Two clarifications matter here:

### It is effectively a restore-and-recover workflow

Yes, duplication from backups behaves a lot like restoring another database from
backup and then recovering it to a consistent point.

That is the right mental model.

### RMAN can automatically resume failed duplication

The transcript says a failed duplication can often resume.
That is real.

Oracle supports automatic recovery from a failed `DUPLICATE`, unless you use:

- `NORESUME`

So this behavior is not imaginary. It is just not magic either.

If your prior attempt left a mess that no longer matches your new plan, clean up
first instead of asking RMAN to infer your emotional state from half-restored
files.

---

## 10. Important DUPLICATE Options

This part of the lecture is useful, but several clauses need translation into
actual Oracle behavior.

### SKIP READONLY

Excludes data files in currently read-only tablespaces from the duplicate.

By default:

- RMAN duplicates current read-only tablespaces

### SKIP TABLESPACE

Excludes specified tablespaces from the duplicate database.

### TABLESPACE

Includes only the listed tablespaces in the duplicate.

Oracle automatically includes:

- `SYSTEM`
- `SYSAUX`
- undo tablespaces

These cannot be skipped.

### NOFILENAMECHECK

The transcript describes this as if it simply helps when directories differ.
That is not the real purpose.

`NOFILENAMECHECK` tells RMAN:

- do not check whether source and duplicate file names are already in use

It is mainly required when:

- the duplicate is on a different host
- that host has the same directory layout and file names as the source

It is dangerous on the same host.
Oracle explicitly warns not to use it there unless you enjoy corrupting the
database you were hoping to clone.

### NOOPEN

Leaves the duplicate database closed after creation.

It does **not** mean read only.
It means:

- RMAN does not open it after duplication

### OPEN RESTRICTED

This is one the lecture gets wrong.

`OPEN RESTRICTED` does **not** open the duplicate in read-only mode.

It opens the duplicate with:

- restricted session enabled

Oracle does this by issuing:

- `ALTER SYSTEM ENABLE RESTRICTED SESSION`

immediately before opening the duplicate.

So:

- restricted is not the same thing as read only

### NOREDO

This clause suppresses application of archived redo logs during duplication.

It is required in some backup-based scenarios, especially when:

- the source database was in `NOARCHIVELOG` mode when the backups were taken
- and RMAN is not connected to the target, so it cannot determine that on its own

It can also be used when:

- you intentionally do not want archived redo applied to a consistent backup

So `NOREDO` is not a generic "make duplication faster" button.
It is a deliberate choice about recovery scope.

### UNDO TABLESPACE

This must be specified when:

- you duplicate only a subset of tablespaces
- the source database is not open
- and the connection pattern means RMAN cannot infer everything it needs

Oracle documents this particularly for backup-based duplication when tablespaces
with undo segments are involved.

---

## 11. Backup-Based Duplication to a Point in Time

Unlike active duplication, backup-based duplication supports:

- `UNTIL TIME`
- `UNTIL SCN`
- `UNTIL SEQUENCE`
- `TO RESTORE POINT`

That makes backup-based duplication useful for:

- creating a test copy as of a past time
- reproducing a bug or data state
- recovering an environment to a specific point for investigation

Example shape:

```rman
RUN
{
  ALLOCATE AUXILIARY CHANNEL a1 DEVICE TYPE DISK;

  DUPLICATE TARGET DATABASE TO newdb
    PFILE='?/dbs/initNEWDB.ora'
    UNTIL TIME "SYSDATE-1"
    SKIP TABLESPACE example, history
    DB_FILE_NAME_CONVERT '/h1/oracle/dbs/trgt','/h2/oracle/oradata/newdb';
}
```

That is real backup-based duplication behavior.
It is also the clean dividing line from the next topic.

---

## 12. Duplicating PDBs: There Are Two Different Cases

This is where the lecture starts treating every PDB duplication option as if it
were one feature. It is not.

### Case A: Duplicate one or more PDBs to a new CDB

This is still part of the broader `DUPLICATE` functionality and can be:

- backup-based
- or active, depending on the command form

Examples include:

```rman
DUPLICATE DATABASE TO cdb1
  PLUGGABLE DATABASE pdb1, pdb3;
```

or:

```rman
DUPLICATE DATABASE TO cdb1
  SKIP PLUGGABLE DATABASE pdb3;
```

or:

```rman
DUPLICATE DATABASE TO cdb1
  PLUGGABLE DATABASE pdb1
  SKIP TABLESPACE pdb1:data1;
```

These patterns duplicate selected `PDB`s into a **new** `CDB`.

### Case B: Duplicate a PDB to an existing CDB

This is different.
Oracle documents that when you use:

- `TARGET PLUGGABLE DATABASE pdb_name`

to duplicate into an existing `CDB`:

- **only active duplication is supported**

That means this is **not** backup-based duplication.

So the lesson title and the second half of the lecture are not actually about
the same workflow.

---

## 13. Active Duplication of a PDB to an Existing CDB

Oracle added this in:

- `18c` / `18.1`

Key prerequisites include:

- source and destination `COMPATIBLE` set to `18.0.0` or higher
- source `CDB` and destination `CDB` both use local undo
- source `PDB` is in read-only or read-write mode
- destination `CDB` is open read-write
- destination `CDB` uses an `SPFILE`
- `REMOTE_RECOVERY_FILE_DEST` is set on the destination `CDB`
- RMAN connects to the root of the auxiliary `CDB`

That `REMOTE_RECOVERY_FILE_DEST` setting is for:

- foreign archived redo logs needed during the duplicate

So yes, the transcript is right that you must set it.
It just belongs to the **active PDB duplication to existing CDB** story, not to
backup-based duplicate database in general.

---

## 14. Syntax for Duplicating a Live PDB to an Existing CDB

The source PDB is named in the `TARGET PLUGGABLE DATABASE` clause.

Example with same PDB name on destination:

```rman
CONNECT TARGET "sysbackup@cdb1 AS SYSBACKUP";
CONNECT AUXILIARY "sysbackup@cdb2 AS SYSBACKUP";

DUPLICATE TARGET PLUGGABLE DATABASE pdb1
  TO cdb2
  FROM ACTIVE DATABASE;
```

Example with rename on destination:

```rman
CONNECT TARGET "sysbackup@cdb1 AS SYSBACKUP";
CONNECT AUXILIARY "sysbackup@cdb2 AS SYSBACKUP";

DUPLICATE TARGET PLUGGABLE DATABASE pdb1
  AS pdb2
  TO cdb2
  FROM ACTIVE DATABASE
  DB_FILE_NAME_CONVERT ('/cdb1/','/cdb2/');
```

Important limitation:

- this form duplicates one source `PDB` per command into an existing `CDB`

So if the lecture sounds like it is mixing "duplicate selected PDBs to a new
CDB" with "duplicate one live PDB to an existing CDB," that is because it is.

---

## 15. Practical Takeaways

- Backup-based duplication restores a duplicate from existing backups and then
  recovers it to consistency.
- For backup-based duplication, a password file is optional in some cases, not
  universal.
- The auxiliary must start in `NOMOUNT`.
- `SET NEWNAME`, `CONFIGURE AUXNAME`, conversion parameters, and OMF are all
  valid naming strategies, but they solve slightly different problems.
- `OPEN RESTRICTED` is restricted session, not read only.
- `NOOPEN` means leave the duplicate closed after creation.
- `NOFILENAMECHECK` is mainly for same names on a different host and is unsafe
  on the same host.
- `NOREDO` is for suppressing archived redo application in specific backup-based
  scenarios, not as a generic speed charm.
- Duplicating a `PDB` to an existing `CDB` is an **active duplication** feature,
  even if the lecture tries to smuggle it into the backup-based lesson.

The easiest way to stay sane is to keep these two buckets separate:

- duplicate database from backups
- duplicate live `PDB` into existing `CDB`

Both use `DUPLICATE`.
They are not the same job.
And Oracle will be thrilled to teach that distinction by error message if you
do not learn it voluntarily.

---

## 16. Practice 2.2: Duplicating a PDB into an Existing CDB

This practice finally commits to the feature the lecture had been hinting at:

- duplicate one live `PDB` from `ORCLCDB` into the existing `DBTEST` `CDB`

And because Oracle enjoys consistency about as much as naming clarity, this
practice still appears under the workbook's "backup based duplicate database"
section even though the actual command is:

- `FROM ACTIVE DATABASE`

So, once again:

- this is an **active PDB duplication** lab
- not a backup-based one

### Goal of the lab

The source `PDB` is:

- `ORCLPDB1`

The destination `CDB` is:

- `DBTEST`

The duplicate `PDB` will be created as:

- `DBTESTPDB1`

This is duplication into an **existing** `CDB`, so it follows the rules from
the earlier lesson section on `TARGET PLUGGABLE DATABASE`.

### Step 1: Validate source data

The practice first connects to:

- `ORCLPDB1`

and queries:

```sql
SELECT *
FROM hr.regions;
```

The point is not that `HR.REGIONS` is special.
The point is that you want a small known-good result set before duplication.

In the lab, that baseline is:

- four rows

If the duplicate later returns the same four rows, then at least one small but
important slice of reality survived the trip.

### Step 2: Prepare the destination CDB

On `DBTEST`, the practice sets:

```sql
ALTER SYSTEM SET remote_recovery_file_dest='$HOME/labs/DBMod_Duplicate'
SCOPE=BOTH;
```

That parameter gives Oracle a place to restore:

- foreign archived redo logs

needed during this active `PDB` duplication.

The lab then creates a filesystem directory for the new `PDB` data files, such
as:

- `/u01/app/oracle/oradata/DBTEST/dbtestpdb1`

That is straightforward same-host hygiene.
The duplicate needs somewhere to put its files, and Oracle is not going to
invent a good directory structure out of personal growth.

### Step 3: Connect RMAN to source and destination

The practice then connects RMAN to:

- `TARGET` = `ORCLCDB`
- `AUXILIARY` = `DBTEST`

This is the correct connection model for duplicating a `PDB` into an existing
`CDB`.

Conceptually:

- the source `CDB` provides the live `PDB`
- the destination `CDB` acts as the auxiliary side that receives the duplicate

### Step 4: Duplicate the PDB

The lab's duplicate command is the heart of the exercise:

```rman
DUPLICATE TARGET PLUGGABLE DATABASE orclpdb1
  AS dbtestpdb1
  TO DBTEST
  FROM ACTIVE DATABASE
  DB_FILE_NAME_CONVERT ('/u01/app/oracle/oradata/ORCLCDB/orclpdb1/',
                        '/u01/app/oracle/oradata/DBTEST/dbtestpdb1/');
```

The exact path strings depend on the lab environment, but the moving parts are
the important part:

- source `PDB` name
- new destination `PDB` name
- destination `CDB`
- active duplication
- file-name conversion so the copied files land in the right place

This is one of the cleaner uses of `DB_FILE_NAME_CONVERT` because:

- the source and destination are on the same host
- the top-level directory pattern is predictable

### Step 5: Verify the new PDB

After duplication, the practice verifies success by:

- `SHOW PDBS`
- switching the session into `DBTESTPDB1`
- querying `HR.REGIONS` again

That is exactly the right post-check.

It proves that:

- the new `PDB` exists
- it is open
- the schema objects are there
- the copied data matches the source baseline

And because the test object is tiny, it is a good fast sanity check instead of
an accidental benchmark.

### Step 6: Clean up

The practice drops `DBTESTPDB1` by script at the end.

That is a good reminder that duplication labs are not just about creation.
They are also about learning how not to leave yesterday's clones cluttering the
machine like abandoned science projects.

---

## 17. What This Practice Adds to the Lesson

Practice 2.2 reinforces several points that matter more than the toy query it
uses for validation:

- duplicating a `PDB` into an existing `CDB` is a separate workflow from
  duplicate-to-new-`CDB`
- the destination `CDB` needs `REMOTE_RECOVERY_FILE_DEST`
- `DB_FILE_NAME_CONVERT` matters immediately when the duplicate must land in a
  different directory tree
- verification should happen inside the new `PDB`, not just at the root

It also cleanly exposes one of Oracle's more dependable patterns:

- the title says one thing
- the syntax says another
- the syntax is the one you should trust
