## Lesson 14 - Managing Target Database Records in the Recovery Catalog (in which RMAN politely asks you to keep the catalog in sync with reality)

And look, creating the recovery catalog is only the beginning. A beautiful empty catalog is still an empty catalog. The real work starts when you register target databases, keep the metadata synchronized, and avoid treating the catalog like a magical box that updates itself out of sheer professional pride.

By the end of this lesson, you should be able to:

- Register a target database in the recovery catalog
- Explain automatic versus manual recovery catalog resynchronization
- Distinguish full resync from partial resync
- Understand when snapshot control files are involved
- Unregister a target database when it is no longer needed in the catalog
- Protect the recovery catalog database itself with a sensible backup strategy

---

## 1. Why Registration Matters

Before the recovery catalog can track a target database, that target has to be
registered in the catalog.

Registration means RMAN takes metadata from the target control file and records
it in the recovery catalog schema.

That gives the catalog knowledge of:

- the target database identity
- the physical schema as seen through the control file
- backup metadata and related RMAN records

So when someone says "use the catalog," what they really mean is:

- create the catalog
- connect to the target
- register the target
- keep the catalog synchronized

Miss one of those steps and you do not have a catalog strategy.
You have a very confident misunderstanding.

---

## 2. Connect to Both Target and Catalog

To register a target database, RMAN must be connected to:

- the target database
- the recovery catalog

Representative example:

```text
rman target "/ as sysbackup" catalog rcatowner@rcatpdb
```

Or, with explicit credentials:

```text
rman target sysbackup@orclcdb catalog rcatowner@rcatpdb
```

Once both connections exist, RMAN knows:

- which target database you mean
- which catalog should record it

That is the moment Oracle finally has enough context to stop guessing and start
doing useful work.

---

## 3. Register the Database

The command itself is blessedly simple:

```rman
REGISTER DATABASE;
```

When RMAN registers the target:

- the database is added to the recovery catalog
- RMAN performs a full resynchronization
- the catalog is populated with metadata from the target control file

That full resync is not a side effect.
It is the whole point.

Because registering a database without immediately pulling in the control-file
metadata would be a bit like opening a library and forgetting the books.

---

## 4. What Automatic Resynchronization Does

When you use RMAN with both:

- the target database
- and the recovery catalog

Oracle normally resynchronizes the catalog automatically as needed during RMAN
operations.

That means if you are regularly connected to both during:

- backup
- restore
- delete
- maintenance operations

then RMAN is usually keeping the catalog reasonably current on its own.

This is why Oracle documentation says you should not need to run
`RESYNC CATALOG` constantly.

In other words, manual resync is a tool, not a lifestyle.

---

## 5. Full Resync vs Partial Resync

Resynchronization can be:

- **partial**
- **full**

### Partial resync

A partial resynchronization updates changed metadata such as:

- new backup records
- new archived redo log records
- new datafile copy records

But it does **not** refresh the physical schema metadata for the database.

So partial resync is basically Oracle saying:
"the routine backup facts changed, not the structure of civilization."

### Full resync

A full resynchronization updates:

- backup metadata
- archived log metadata
- physical schema metadata

That includes structural changes such as:

- adding or dropping data files
- schema-related physical changes
- changes to RMAN persistent configuration
- new incarnations

This is the heavyweight version.

And yes, this is the one where Oracle gets more serious about consistency.

---

## 6. Snapshot Control File and Why Oracle Uses It

During a full resynchronization, RMAN creates a snapshot control file.

Why?

Because Oracle wants a read-consistent view of the control file while the real
database continues doing its ordinary business instead of freezing in place for
your administrative convenience.

So the flow is roughly:

1. create snapshot control file
2. read metadata from the snapshot
3. update the recovery catalog
4. complete the full resync

This is a much more civilized approach than hammering directly on the live
control file while users and background processes are trying to get work done.

---

## 7. Manual Resynchronization

If you do RMAN work without connecting to the recovery catalog every time, then
the catalog can drift behind the target control file.

That is when you manually resynchronize:

```rman
RESYNC CATALOG;
```

Prerequisites:

- connected to the target database
- connected to the recovery catalog

When you do this, RMAN determines what kind of resynchronization is needed and
updates the catalog accordingly.

This is especially useful when:

- backups were taken in `NOCATALOG` mode
- the catalog was unavailable during backup work
- you want to bring the catalog current later

So manual resync is Oracle's version of saying:
"Fine, if you insisted on working offline from the catalog, go reconcile your
paperwork now."

---

## 8. Cataloging Extra Backup Files

The lesson also hints at another useful administrative move:

- cataloging backup files that are not currently known through active control
  file metadata

That matters when:

- files are older
- control file records aged out
- copies exist on disk that RMAN should know about again

Typical examples:

```rman
CATALOG START WITH '/u01/backup/';
CATALOG BACKUPPIECE '/u01/backup/db_piece_01.bkp';
CATALOG DATAFILECOPY '/u01/backup/users01_copy.dbf';
```

This does not recreate the backup.
It tells RMAN:

"These files are real, they matter, and you need to stop acting like you have
never met them."

---

## 9. Unregistering a Target Database

If you no longer want a target database tracked in the catalog, you can remove
it:

```rman
UNREGISTER DATABASE;
```

Prerequisites:

- connected to the target database
- connected to the catalog

This removes the target's metadata from the recovery catalog.

And yes, Oracle is quite clear here:

- unregister first if you want to remove catalog participation cleanly

Because "I thought we just stopped using it" is not an actual metadata
management policy.

---

## 10. Demo-Style Registration Flow

The practical demo flow is:

1. source the target database environment
2. connect RMAN to both target and catalog
3. issue `REGISTER DATABASE;`
4. let RMAN perform the full resync
5. optionally run a quick verification command

Representative session:

```text
rman target "/ as sysbackup" catalog rcatowner@rcatpdb
```

Then:

```rman
REGISTER DATABASE;
REPORT SCHEMA;
```

`REPORT SCHEMA` is not the registration command.
It is just a nice sanity check afterward, because it confirms RMAN is now
working with the target metadata you expected.

---

## 11. Demo-Style Manual Resync Flow

Manual resync is even simpler:

```text
rman target "/ as sysbackup" catalog rcatowner@rcatpdb
```

Then:

```rman
RESYNC CATALOG;
```

If a full resync is required, RMAN uses a snapshot control file.
If only partial catalog updates are needed, RMAN performs the lighter refresh.

Either way, the goal is the same:

- make the catalog match the current truth of the target control file

Which is a noble goal in databases, because truth has a habit of decaying when
you stop maintaining it.

---

## 12. Practice-Style Notes on Registering the Catalog Database Itself

The practice then takes the wonderfully responsible step of asking:

"What protects the recovery catalog database?"

Answer:

- you protect it like any other important database

Because if your recovery catalog is operationally important, not backing it up
would be a very strange commitment to irony.

The practice flow includes:

- configuring retention policy for the catalog database itself
- configuring the Fast Recovery Area
- enabling `ARCHIVELOG` mode
- taking a base image-copy backup

Representative RMAN configuration example:

```rman
SHOW RETENTION POLICY;
CONFIGURE RETENTION POLICY TO REDUNDANCY 2;
```

Representative FRA sizing step:

```sql
ALTER SYSTEM SET DB_RECOVERY_FILE_DEST_SIZE = 12G SCOPE=BOTH;
```

Representative base image-copy backup:

```rman
BACKUP AS COPY INCREMENTAL LEVEL 0 DATABASE;
```

That gives the catalog database its own backup baseline so you can later use an
incrementally updated image-copy strategy for fast restore.

Which is exactly the sort of thing you should do for the database that stores
the metadata about everyone else's backups.

---

## 13. Practical Takeaways

Use the recovery catalog properly:

1. create it
2. register targets
3. connect to it during RMAN work when practical
4. resync it manually when needed
5. back up the catalog database itself

The catalog is helpful because it centralizes memory.
That also means you should not let its own protection be an afterthought.

Otherwise you end up with a backup brain that nobody bothered to back up, which
is a very Oracle-flavored tragedy.

---

## 14. Wrap-Up

Managing target records in the recovery catalog means:

- registering the target database
- understanding automatic and manual resync
- knowing when full versus partial sync happens
- cataloging older backup artifacts when needed
- unregistering databases cleanly
- and protecting the recovery catalog database itself

At this point, the recovery catalog stops being just "that extra RMAN database"
and becomes what it actually is: the long-term memory and administrative center
for your backup metadata.
