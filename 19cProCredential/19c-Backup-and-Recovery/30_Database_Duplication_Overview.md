## Lesson 30 - Database Duplication Overview (in which RMAN happily clones your database, provided you first solve the small matter of passwords, listeners, file names, and the auxiliary instance's identity crisis)

Database duplication is one of those Oracle features that sounds simple right up
until you do it on purpose. The basic idea is easy: create another database from
an existing one. The operational reality is that RMAN needs a source,
somewhere to put the copy, and a fairly disciplined answer to the question:

- "What exactly are we duplicating, from where, and by what method?"

By the end of this lesson, you should be able to:

- explain why duplicate databases are created
- distinguish active duplication from backup-based duplication
- understand the roles of the `TARGET`, `AUXILIARY`, and `CATALOG` connections
- explain the lecture's push-based versus pull-based language without lying to
  yourself
- prepare a basic auxiliary instance for duplication
- recognize what the setup demo is really doing before RMAN ever copies a byte

---

## 1. Why Duplicate a Database

A duplicate database is commonly used for:

- testing backup and recovery procedures
- development or patch testing
- troubleshooting against production-like data
- object-level salvage work on an isolated copy
- building another environment without hand-creating everything like a medieval
  scribe

RMAN duplication can create:

- a full duplicate database
- a subset duplicate in some cases
- a standby database if you use `FOR STANDBY`

Important correction to the source:

- a normal duplicate database gets a new `DBID`
- a standby database created with `DUPLICATE ... FOR STANDBY` does **not**

So when this lesson says "duplicate database," assume a normal non-standby
duplicate unless stated otherwise.

---

## 2. Core Terms You Need Straight

Oracle's naming here is not intuitive, because naturally it is not.

### Source or target database

This is the database you are copying **from**.

In RMAN duplication, the source database is typically the:

- `TARGET`

### Auxiliary instance

This is the instance that starts out as a very small shell and ends up becoming
the duplicate database.

At the beginning it is usually:

- started in `NOMOUNT`
- backed only by a minimal parameter file or `SPFILE`
- equipped with a password file

### Destination host

This is the server that will hold the duplicate.

It can be:

- the same host as the source
- a different host

### Recovery catalog

Optional for some duplication modes, but useful when RMAN must locate backup
metadata without depending on the source control file.

---

## 3. The Two Big Duplication Families

RMAN supports two main approaches:

- **active database duplication**
- **backup-based duplication**

### Active database duplication

RMAN copies the source database over the network directly to the auxiliary
instance.

This means:

- no pre-existing backup is required
- the source database must be `MOUNT`ed or open
- if the source is open, it must be in `ARCHIVELOG` mode

Oracle recommends active duplication in general because it requires less staging
and less backup choreography.

### Backup-based duplication

RMAN creates the duplicate from existing backups and archived redo logs.

This means:

- the duplicate is restored from backup pieces or copies
- the destination host must be able to access the required backups
- you may not need a live target connection, depending on the mode you choose

This is often the better option when:

- backups already exist
- network bandwidth is limited
- you want to avoid leaning on the source database during duplication

---

## 4. Push Versus Pull: Useful, but Only in the Active-Duplication Branch

The lecture uses "push" and "pull" terminology.
That is real Oracle language, but only for **active** duplication.

### Push-based active duplication

This is active duplication using:

- **image copies**

Here, the source side does the heavier lifting.
The target channels do most of the work, and the source pushes the copied files
toward the auxiliary side.

### Pull-based active duplication

This is active duplication using:

- **backup sets**

Here, the auxiliary side does the heavier lifting.
The auxiliary connects back to the source through Oracle Net and retrieves what
it needs.

This method is attractive because backup sets can use:

- compression
- encryption
- multisection backup processing with `SECTION SIZE`

And Oracle explicitly recommends backup sets for many active duplication cases.

Important correction:

- backup-based duplication is **not** just "pull with older files"
- it is a separate duplication family that restores from pre-existing RMAN
  backups and copies

So the clean mental model is:

- active duplication = live source over the network
- backup-based duplication = existing backups
- push versus pull = only the active branch's internal transport style

---

## 5. Which RMAN Connections Are Needed

The connection pattern depends on the duplication method.

### Active duplication

Requires:

- `TARGET`
- `AUXILIARY`

And if you connect to the source as `TARGET`:

- you must supply a user name and password
- the auxiliary connection must use the same user name and password

### Backup-based duplication with a target connection

Typical connections:

- `TARGET`
- `AUXILIARY`

### Backup-based duplication without a target connection

Possible patterns include:

- `CATALOG` plus `AUXILIARY`
- `AUXILIARY` only, when `BACKUP LOCATION` contains what RMAN needs

If you remove the target connection, then the burden shifts to:

- the recovery catalog metadata
- or a clean backup location
- or both

That is the tradeoff. Fewer live connections, more dependence on prepared
backup visibility.

---

## 6. What RMAN Actually Duplicates

RMAN does **not** just mirror everything in the source host's life.

For a normal duplicate database:

- data files are copied or restored
- control files are re-created
- tempfiles are re-created
- online redo logs are re-created
- archived redo is copied or restored only if needed to recover the duplicate to
  a consistent point

`SPFILE` behavior:

- active duplication can copy it from the source when you use the `SPFILE`
  clause
- backup-based duplication can restore it from backup

Password file behavior:

- for non-standby duplicates, RMAN does **not** copy it by default
- use `PASSWORD FILE` if you want RMAN to overwrite the auxiliary password file

Things RMAN does **not** re-create:

- flashback logs
- block change tracking file
- miscellaneous files already sitting in the source FRA

So no, duplicating a database does not duplicate its entire emotional history.

---

## 7. Prerequisites That Actually Matter

For active duplication, Oracle's key prerequisites include:

- at least one target channel and at least one auxiliary channel
- source database `MOUNT`ed or open
- `ARCHIVELOG` required if the source is open
- if the source is not open, it must have been shut down consistently
- source database uses a server parameter file
- same administrative password on source and auxiliary for the duplication user
- static listener configuration
- Oracle Net connectivity where required

Oracle also documents an easily missed restriction:

- you cannot use the `UNTIL` clause in active database duplication

RMAN chooses the recovery point based on when the copied files can be made
consistent.

For backup-based duplication, the central requirement is different:

- the auxiliary host must be able to access all backups and archived redo needed
  for restore and recovery

If the destination host is different from the source host, and you are using
disk backups with target or catalog-driven duplication, those backups generally
need to be visible with the same full path names.

---

## 8. Preparing the Auxiliary Instance

The demo's first half is really an auxiliary-instance preparation exercise.
That is exactly the right place to start.

Oracle documents that the auxiliary initialization parameter file must contain at
least:

- `DB_NAME`
- `DB_DOMAIN`

And if the auxiliary is a `CDB`, then it must also include:

- `ENABLE_PLUGGABLE_DATABASE=TRUE`

In practice, you usually also define:

- `CONTROL_FILES`
- `DB_FILE_NAME_CONVERT`
- `LOG_FILE_NAME_CONVERT`
- `DB_RECOVERY_FILE_DEST`
- `DB_RECOVERY_FILE_DEST_SIZE`

Those conversion and destination parameters matter whenever:

- source and duplicate are on the same host
- directory structures differ
- you want RMAN to stop guessing and start behaving

Oracle's documented sample for an auxiliary parameter file looks roughly like
this:

```ini
DB_NAME=DBTEST
CONTROL_FILES=(/u01/app/oracle/oradata/DBTEST/control01.ctl,
               /u01/app/oracle/oradata/DBTEST/control02.ctl)
DB_FILE_NAME_CONVERT=(/u01/app/oracle/oradata/ORCL/,
                      /u01/app/oracle/oradata/DBTEST/)
LOG_FILE_NAME_CONVERT=(/u01/app/oracle/oradata/ORCL/redo,
                       /u01/app/oracle/oradata/DBTEST/redo)
DB_RECOVERY_FILE_DEST=/u01/app/oracle/dbtest_fra
DB_RECOVERY_FILE_DEST_SIZE=20G
ENABLE_PLUGGABLE_DATABASE=TRUE
```

The lecture's lab adds the same ideas with slightly different path pairs for
`ORCL` and `DBTEST`, which is exactly what you should expect on a same-host
duplicate.

---

## 9. Password File and Network Setup

The demo spends a lot of time in service names and password files because active
duplication depends on them more than people want to admit.

### Password file

For active duplication:

- password file authentication is required
- the auxiliary password file must exist
- `SYS` and `SYSBACKUP` passwords must match the source database

You can:

- copy the source password file and rename it
- or create one manually with `orapwd`

For a remote auxiliary host, the password file is mandatory.

### Oracle Net and listener

Oracle documents that you must configure:

- Oracle Net connectivity between source and auxiliary
- a static listener entry for duplication

The lab uses `netca` and edits `tnsnames.ora`.
That is fine as a workflow, but the real requirement is simpler:

- the source and auxiliary must be able to resolve and connect to each other in
  the way the chosen duplication method requires

The lecture also adds `SERVER=DEDICATED` in the connect descriptor.
That can be reasonable, but it is not the magical essence of duplication.
The essential part is:

- the service resolves
- the listener knows about the auxiliary
- the connection works

---

## 10. Directories and File Naming

Before duplication, create the directories the auxiliary will use for:

- data files
- control files
- online redo logs
- tempfiles
- fast recovery area contents

The demo's `DBTEST` setup creates the future FRA path first and makes sure it is
empty.
That is sensible.
RMAN duplication goes much better when you do not hand it a destination already
polluted with leftovers from some previous bright idea.

If source and duplicate use the same file names:

- that is only safe on a different host
- or in carefully isolated storage layouts

If you duplicate to the same host, use:

- file name conversion parameters
- or OMF destinations
- or both

And if you truly must use identical names on a remote host, `NOFILENAMECHECK`
exists.
That clause is not permission to be reckless.
It is permission to be deliberate.

---

## 11. Starting the Auxiliary Instance

Because the auxiliary does not yet have a control file, Oracle allows only:

- `NOMOUNT`

So the startup pattern is:

```sql
STARTUP NOMOUNT PFILE='$ORACLE_HOME/dbs/initDBTEST.ora';
```

Or, if you already created a server-side parameter file in the default location:

```sql
STARTUP NOMOUNT;
```

Important rule:

- do **not** create a control file yourself
- do **not** try to mount it
- do **not** try to open it

The auxiliary instance is supposed to be incomplete at this stage.
That is its whole job description.

---

## 12. What the Actual RMAN Duplicate Command Will Look Like

Once the environment is ready, duplication itself becomes surprisingly compact.

### Active duplication, generic shape

```rman
CONNECT TARGET "sysbackup@orcl AS SYSBACKUP";
CONNECT AUXILIARY "sysbackup@dbtest AS SYSBACKUP";

DUPLICATE TARGET DATABASE TO DBTEST
  FROM ACTIVE DATABASE
  PASSWORD FILE
  SPFILE;
```

### Active duplication using backup sets

```rman
DUPLICATE TARGET DATABASE TO DBTEST
  FROM ACTIVE DATABASE
  USING BACKUPSET
  SECTION SIZE 1G
  PASSWORD FILE
  SPFILE;
```

That second form is the pull-based active method.
The auxiliary channels do the heavier work, and you gain the usual backup-set
advantages.

### Backup-based duplication, generic idea

```rman
DUPLICATE DATABASE TO DBTEST
  BACKUP LOCATION '/path/to/backups';
```

That pattern changes depending on whether RMAN is connected to:

- `TARGET`
- `CATALOG`
- `AUXILIARY`

But the big distinction remains the same:

- active duplication copies from the live source
- backup-based duplication restores from already existing backups

---

## 13. Multitenant Note

If you duplicate a whole `CDB` or one or more `PDB`s, then the auxiliary side
must respect multitenant rules.

Key points:

- create the auxiliary instance as a `CDB`
- connect RMAN to the root container on both source and auxiliary
- include `ENABLE_PLUGGABLE_DATABASE=TRUE` in the auxiliary parameter file

For some `PDB` duplication cases, Oracle also requires:

- local undo
- `REMOTE_RECOVERY_FILE_DEST` on the destination `CDB`
- compatible release levels on source and destination

This is where duplication stops being a generic cloning feature and starts
becoming multitenant-specific bureaucracy, which is very on brand.

---

## 14. What This Demo Really Teaches

The demo looks, at first glance, like a tour of directories and config files.
That is because it is.

And that is not wasted time.

It teaches the real preconditions for successful duplication:

- the auxiliary has a valid identity
- the file destinations exist
- name conversion is defined
- the password file is ready
- Oracle Net can resolve the connection
- the auxiliary can start in `NOMOUNT`

Only after those boxes are checked does RMAN get to do the part everyone
actually remembers.

---

## 15. Practical Takeaways

- RMAN duplication comes in two big forms: active and backup-based.
- Push-based versus pull-based language applies to **active** duplication, not to
  the whole feature.
- Active duplication using backup sets is often the better choice because it
  supports compression, encryption, and multisection processing.
- A normal duplicate database gets a new `DBID`; a standby created with `FOR STANDBY` does not.
- The auxiliary instance starts in `NOMOUNT` with a minimal parameter file or
  `SPFILE`.
- For active duplication, matching administrative passwords and a usable
  password file are not optional details.
- Static listener and Oracle Net setup are part of the duplication
  prerequisites, not decorative lab busywork.
- File name conversion matters a lot on same-host duplicates.

Database duplication is one of Oracle's genuinely useful features.
It is also one of the better examples of a recurring Oracle law:

- the command is short
- the preparation is not
- and the preparation is what determines whether the short command feels elegant
  or humiliating

---

## 16. Demo Part 2: Running the Active Duplication

The second half of the demo is the part everyone thinks of as "the clone."
That is fair enough.
It is also the point where all of the earlier setup either pays off or ruins
your afternoon.

### Confirm the source database state

If the source database is open during active duplication, then Oracle requires:

- `ARCHIVELOG` mode

So the check is exactly the one shown in the lesson:

```sql
SELECT name, log_mode, open_mode
FROM v$database;
```

If the source is:

- open read-write, then it must be in `ARCHIVELOG`
- mounted only, then it must have been shut down consistently before being
  brought to `MOUNT`

The lab checks that `ORCL` is:

- `ARCHIVELOG`
- `READ WRITE`

which is the correct state for active duplication from a live source.

### Confirm administrative password alignment

When RMAN connects to the source as `TARGET` and to the auxiliary as
`AUXILIARY`, Oracle requires:

- the same user name
- the same password

So if you are using `SYS` or `SYSBACKUP` for the duplication, then the source
database password and the auxiliary password file must agree.

If they do not, RMAN does not reward initiative.
It simply refuses the connection.

### TNS_ADMIN is practical, not mystical

The demo explicitly sets:

- `TNS_ADMIN`

That is useful when your `tnsnames.ora`, `listener.ora`, and related Oracle Net
files live outside the default location or when you want to be absolutely
certain RMAN uses the intended network config.

It is not a universal duplication prerequisite.
It is an environment-control measure.

So the real lesson is:

- if your Oracle Net files are not in the default path, set `TNS_ADMIN`
- if they are in the default path, Oracle usually already knows where to look

### Run RMAN from the destination side

Oracle documents that the `DUPLICATE` command is run on the destination server.

That is what the lab is doing when it switches to the environment for the future
`DBTEST` system and then starts `RMAN`.

At that point RMAN connects to:

- the source as `TARGET`
- the future duplicate as `AUXILIARY`
- optionally the recovery catalog as `CATALOG`

If a recovery catalog is in use, then RMAN typically resynchronizes metadata as
part of the process before it gets to the interesting part.

### The active duplication command

The lecture describes a duplication that:

- copies the `SPFILE`
- adjusts paths and parameter values
- creates the duplicate from the live source

That pattern is exactly what Oracle expects.

A representative command shape is:

```rman
DUPLICATE TARGET DATABASE TO DBTEST
  FROM ACTIVE DATABASE
  SPFILE
    PARAMETER_VALUE_CONVERT 'ORCL','DBTEST'
    SET db_recovery_file_dest_size='20G'
    SET db_file_name_convert='/u01/app/oracle/oradata/ORCL',
                             '/u01/app/oracle/oradata/DBTEST'
    SET log_file_name_convert='/u01/app/oracle/fast_recovery_area/ORCL',
                              '/u01/app/oracle/dbtest_fra/DBTEST';
```

Two important Oracle details apply here:

- `PARAMETER_VALUE_CONVERT` changes matching initialization parameter values
  when the source `SPFILE` is copied
- `DB_FILE_NAME_CONVERT` and `LOG_FILE_NAME_CONVERT` are exceptions and should
  be set explicitly

So the lab's use of:

- `SPFILE PARAMETER_VALUE_CONVERT`
- plus explicit `SET` clauses

is the correct pattern, not decorative syntax.

### What RMAN then does

Once the command starts, RMAN goes through the usual active-duplication
sequence:

1. validate connections and metadata
2. copy or create the server parameter file for the auxiliary
3. restart the auxiliary with the copied parameters as needed
4. create a control file for the duplicate
5. copy data files from the source
6. recover the duplicate
7. open the duplicate with `RESETLOGS`

That is why the transcript shows RMAN:

- copying files
- setting names
- switching to the copied files
- and eventually reporting that duplication is complete

The command is short.
The machinery behind it is not.

### Verify the duplicate

After RMAN finishes, the verification steps in the demo are sensible:

- connect locally to `DBTEST`
- `SHOW PDBS`
- open a `PDB` if needed
- compare object counts or query results with the source

The transcript checks the row count in:

- `HR.EMPLOYEES`

inside `PDB1` and confirms that the duplicate matches the source.

That is the right instinct.
Do not stop at "RMAN said finished."
Oracle has said many optimistic things over the years.

### Check the listener

The final `lsnrctl status` check is just a post-build sanity test:

- the listener should now advertise `DBTEST`
- and the expected pluggable database services

That is not part of the duplication engine itself.
It is the proof that your network plumbing and the finished clone are now
aligned.

---

## 17. Practice 2.1: Duplicating a Database

The workbook labels this practice under:

- "creating a backup based duplicate database"

but the lab actually uses:

- `FROM ACTIVE DATABASE`

So this practice is an **active duplication** lab, not a backup-based one.

That distinction is worth preserving because Oracle certainly preserves it in
the syntax rules.

### Practice flow

The lab works through four broad tasks:

1. prepare the future `DBTEST` environment
2. verify and fix source-database prerequisites
3. run active duplication with `RMAN`
4. test access to the finished clone

### Environment preparation

The practice first creates:

- FRA and data directories for `DBTEST`
- Oracle Net entries for `DBTEST`
- a password file for `DBTEST`
- a minimal `initDBTEST.ora`

The minimal `PFILE` includes:

- `DB_NAME=DBTEST`
- `REMOTE_LOGIN_PASSWORDFILE=EXCLUSIVE`

That is enough to:

- start the auxiliary in `NOMOUNT`

which is exactly what the lab does next.

### Oracle Net setup

The practice uses Oracle Net Manager to:

- add a `dbtest` service name
- update the listener's static database services
- save the network config
- reload the listener

You could do the same work manually in:

- `tnsnames.ora`
- `listener.ora`

The tool is not the point.
Having a resolvable auxiliary service and listener entry is.

### Source-database prerequisite checks

The lab then returns to `ORCLCDB` and checks:

- `COMPATIBLE`
- `NONCDB_COMPATIBLE`
- FRA size
- `ARCHIVELOG` status

The notable part is that the source initially is **not** in `ARCHIVELOG` mode,
so the practice fixes it by:

1. shutting down the source
2. starting it in `MOUNT`
3. issuing `ALTER DATABASE ARCHIVELOG`
4. reopening the database

That is a useful teaching point, because active duplication from an open source
absolutely depends on this prerequisite.

The practice also enlarges the FRA to `20G`, which is lab-specific but
practical.

### Running the duplication

The practice then switches the environment to `DBTEST` and starts `RMAN`.

The note in the workbook about service names is muddy, so here is the cleaner
version:

- if your auxiliary connection is local or OS-authenticated, the shell
  environment must point to `DBTEST`
- if your auxiliary connection uses a network service name, Oracle Net
  resolution matters more than `ORACLE_SID`

In either case, the lab's intent is obvious:

- make sure the RMAN process is operating in the auxiliary database's
  environment while connecting to the source as `TARGET`

The duplicate command uses:

- `FROM ACTIVE DATABASE`
- `SPFILE`
- `PARAMETER_VALUE_CONVERT`
- explicit `SET` clauses for recovery area sizing and file-name conversion

That is a solid same-host active duplication pattern.

### Verifying the clone

The practice finishes by:

- connecting to `DBTEST`
- checking the root container identity
- checking the duplicate `DBID` and open state
- checking the pluggable databases
- verifying data in `ORCLPDB1`

That is the right kind of validation.

The transcript does this with queries such as:

```sql
SELECT name, cdb, con_id
FROM v$database;
```

and:

```sql
SELECT dbid, name, created, open_mode
FROM v$database;
```

followed by:

```sql
SHOW PDBS
```

That is a better close-out than merely trusting the final RMAN banner.
It confirms that:

- the duplicate really is `DBTEST`
- it has its own `DBID`
- it is open
- and the expected `PDB`s came across

The count query on `HR.EMPLOYEES` is not sophisticated, but it is enough to
prove:

- the `PDB` exists
- the schema objects came across
- the duplicate is not merely alive, but recognizable

---

## 18. What This Demo and Practice Add to the Lesson

The first half of the duplication lesson explained the architecture and setup.
This second half shows the operational truth:

- active duplication is mostly preparation followed by one carefully built
  command

It also reinforces some of the more important Oracle realities:

- `ARCHIVELOG` is not optional for an open active source
- same credentials on source and auxiliary are mandatory when RMAN connects that
  way
- `PARAMETER_VALUE_CONVERT` helps with copied `SPFILE` settings, but it does
  not replace explicit file-name conversion clauses
- verifying the duplicate after RMAN finishes is part of the job, not an
  optional flourish

And perhaps most usefully, the practice accidentally teaches one more thing:

- just because the lab title says "backup based" does not mean the command you
  are running agrees with it

In Oracle work, the syntax is usually the more honest party.
