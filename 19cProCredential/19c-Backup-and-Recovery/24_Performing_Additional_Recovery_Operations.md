## Lesson 24 - Performing Additional Recovery Operations (in which Oracle reminds you that the database is held together by many files, all of which would love to fail at once)

And look, by the time you reach this part of backup and recovery, you have already accepted that data files can disappear, archived logs can go missing, and control files are apparently small but deeply dramatic. This lesson is about the other categories of file loss that are not glamorous, but are absolutely capable of ruining your day.

By the end of this lesson, you should be able to:

- Recover or rebuild an `SPFILE`
- Recover from loss of one control file copy or all control file copies
- Understand when recovery with a backup control file requires `RESETLOGS`
- Explain why `NOLOGGING` operations are a trap for media recovery
- Respond to online redo log member or group loss safely
- Recreate a password file with `orapwd`

---

## 1. The General Theme

This lesson is a collection of recovery scenarios for files that are not
ordinary data files:

- `SPFILE`
- control files
- redo log files
- password file

These are all critical in different ways:

- some are required to start the instance
- some are required to mount the database
- some are required to continue database operation
- some are required only for administrative authentication

So not every lost file means the same thing.
Oracle has many failure modes, and the practical skill is knowing which ones are
annoying, which ones are dangerous, and which ones mean "stop improvising."

---

## 2. Recovering or Rebuilding an `SPFILE`

An `SPFILE` is the binary server parameter file used for instance startup.

Yes, it is effectively the binary counterpart to the text initialization
parameter file (`PFILE`), but the source oversimplifies one important point:

- losing the `SPFILE` does **not** necessarily crash a currently running
  instance immediately
- it becomes a startup problem on the next restart, or immediately if the file
  is needed and unavailable

### Creating a `PFILE`

You can create a text parameter file from the current `SPFILE`:

```sql
CREATE PFILE FROM SPFILE;
```

Or name it explicitly:

```sql
CREATE PFILE = '/u01/app/oracle/product/19.0.0/dbhome_1/dbs/initorcl.ora'
FROM SPFILE;
```

You can also create it from memory:

```sql
CREATE PFILE = '/tmp/initorcl.ora' FROM MEMORY;
```

### Creating an `SPFILE`

You can create an `SPFILE` from a `PFILE`:

```sql
CREATE SPFILE FROM PFILE;
```

Or from memory:

```sql
CREATE SPFILE = '/u01/app/oracle/product/19.0.0/dbhome_1/dbs/spfileorcl.ora'
FROM MEMORY;
```

### Important correction to the source

The default location is **not** simply "whatever directory you happen to be
sitting in like a confused intern."

Oracle documents platform-specific default names and locations, for example:

- `spfileORACLE_SID.ora`
- `initORACLE_SID.ora`
- typically under `ORACLE_HOME/dbs` on Unix/Linux for parameter files
- or in ASM, if the database was created that way

And startup search order matters:

1. `spfileORACLE_SID.ora`
2. `spfile.ora`
3. `initORACLE_SID.ora`

If your `SPFILE` lives somewhere nondefault, then you usually need a stub
`PFILE` in the default location pointing to it.

---

## 3. Restoring an `SPFILE` from Autobackup

If control file autobackup is enabled, RMAN can back up the `SPFILE` as part of
the autobackup process.

The typical `RMAN` pattern in `NOCATALOG` mode is:

```rman
SET DBID 1620189241;
STARTUP FORCE NOMOUNT;
RESTORE SPFILE FROM AUTOBACKUP;
STARTUP FORCE;
```

Important corrections:

- in `NOCATALOG` mode, `FROM AUTOBACKUP` is required when the target is not
  mounted
- if RMAN is not connected to a recovery catalog and the backup is outside the
  default location, you may need `SET CONTROLFILE AUTOBACKUP FORMAT`
- if you do **not** know the `DBID`, find it before disaster, not after

This is why "write the DBID down somewhere sane" is a very adult backup and
recovery habit.

---

## 4. Losing One Multiplexed Control File Copy

The source says, more or less, that you always have two control files by
default.

That is not quite right.

Oracle documents this more carefully:

- at least one copy is created by default
- on some operating systems multiple copies may be created
- Oracle strongly recommends multiplexing and having at least two copies

What matters operationally is:

- the database writes to all control files listed in `CONTROL_FILES`
- if one becomes unavailable while the database is open, the instance becomes
  inoperable and should be terminated

### If one current copy still exists

You do **not** need RMAN restore-and-recover theater if one current good copy
survives.

If the original path is still usable, copy the good control file over the bad
one with the instance down:

```powershell
copy good_control.ctl bad_control.ctl
```

Then start the database normally.

If the original location is gone permanently:

1. copy the good control file to a new location
2. update `CONTROL_FILES` in the parameter file
3. start the database

No media recovery is required in this case, because you are using a **current**
control file copy, not a backup control file.

---

## 5. Losing All Control File Copies

Once all control file copies are gone, you are in a different class of problem.

You typically recover by restoring a backup control file:

```rman
SET DBID 1620189241;
STARTUP NOMOUNT;
RESTORE CONTROLFILE FROM AUTOBACKUP;
ALTER DATABASE MOUNT;
RESTORE DATABASE;
RECOVER DATABASE;
ALTER DATABASE OPEN RESETLOGS;
```

Important corrections to the source:

- the recovery path is not determined by a cute little matrix of "online log
  status versus data file status" in the simplified way the lecture suggests
- the real dividing line is whether you have a **current control file copy** or
  must use a **backup control file**
- if you recover the database using a backup control file, then `OPEN
  RESETLOGS` is required

That requirement is not Oracle being dramatic for fun.
It is Oracle acknowledging that you are opening the database from a recovered
state that begins a new incarnation.

---

## 6. Restoring Both `SPFILE` and Control File

If you lose both files, the order matters.

### With a recovery catalog

The catalog knows your metadata, so the process is easier:

```rman
STARTUP FORCE NOMOUNT;
RESTORE SPFILE FROM AUTOBACKUP;
STARTUP FORCE NOMOUNT;
RESTORE CONTROLFILE FROM AUTOBACKUP;
ALTER DATABASE MOUNT;
RESTORE DATABASE;
RECOVER DATABASE;
ALTER DATABASE OPEN RESETLOGS;
```

### Without a recovery catalog

You usually need:

```rman
SET DBID 1620189241;
STARTUP FORCE NOMOUNT;
RESTORE SPFILE FROM AUTOBACKUP;
STARTUP FORCE NOMOUNT;
RESTORE CONTROLFILE FROM AUTOBACKUP;
ALTER DATABASE MOUNT;
RESTORE DATABASE;
RECOVER DATABASE;
ALTER DATABASE OPEN RESETLOGS;
```

The source gets the broad idea right:

- restore the `SPFILE`
- restart with it
- restore the control file
- mount
- restore/recover
- open with `RESETLOGS`

That is the sequence that matters.

---

## 7. `NOLOGGING` Means "Enjoy Your Incomplete Recovery"

The source is absolutely right to warn about `NOLOGGING`, but it understates how
treacherous it is.

If you perform `NOLOGGING` operations:

- the affected changes are not fully recorded in redo in a recoverable way
- media recovery cannot reconstruct them from archived redo alone

That means after a restore:

- you can recover only to the last backup that already contains those changes
- not through the `NOLOGGING` operation using archived logs

This is why Oracle documentation treats `NOLOGGING` as a serious recovery
consideration, not a fun little performance trick.

Practical rule:

- if you use `NOLOGGING`, take a fresh backup afterward
- if you need stronger protection, use `FORCE LOGGING`

Because `NOLOGGING` is basically a signed waiver saying:

- "I would like speed now and remorse later."

---

## 8. Losing One Redo Log Member

A redo log **group** can have one or more **members**.
LGWR writes to all members of the current group concurrently.

If you lose one member of a multiplexed group but another member survives:

- the database can usually keep running
- Oracle marks the bad member invalid
- errors are written to the alert log and LGWR trace file

The fix is normally:

1. drop the bad member
2. add a replacement member

That is a member-level problem, not yet a group-level disaster.

---

## 9. Redo Log Group Status: A More Accurate Version

The source explains `CURRENT`, `ACTIVE`, and `INACTIVE`, but blurs an important
detail.

### `CURRENT`

- LGWR is writing to this group now

### `ACTIVE`

- the group is not current
- but it is still needed for instance recovery

### `INACTIVE`

- the group is no longer needed for instance recovery

Important correction:

- `INACTIVE` does **not** automatically mean "already archived"
- archive status is tracked separately

So do not collapse these into one concept.
Oracle absolutely does not.

Useful views:

```sql
SELECT group#, thread#, sequence#, archived, status
FROM v$log;
```

```sql
SELECT group#, status, member
FROM v$logfile;
```

---

## 10. Losing an Entire Redo Log Group

This is more serious than losing one member.

### If the lost group is `INACTIVE`

If all members of an inactive group are lost:

- fix the media problem if possible, or
- clear the log group if that is safe

Example:

```sql
ALTER DATABASE CLEAR LOGFILE GROUP 3;
```

If the log was not archived and you intentionally want to discard it:

```sql
ALTER DATABASE CLEAR UNARCHIVED LOGFILE GROUP 3;
```

But this comes with a giant Oracle warning label:

- clearing an unarchived log can make backups unusable if that redo is needed
  for recovery

And if the log is needed to recover an offline data file, Oracle requires:

```sql
ALTER DATABASE CLEAR UNARCHIVED LOGFILE GROUP 3 UNRECOVERABLE DATAFILE;
```

Which is one of Oracle's more honest commands, because it is basically saying:

- "Yes, you can do this, but please understand you are sawing off part of your
  recovery branch."

### If the lost group is `CURRENT` or `ACTIVE`

This is worse.

If the database is still open and the problem is survivable, you may attempt:

- forced log switch
- checkpoint

But if the lost current redo is required and cannot be recovered, you may be
facing:

- incomplete recovery
- possible data loss

So the source's general instinct is right:

- `CURRENT` group loss is the scary one

But the clean operational rule is:

- member loss may be routine
- group loss, especially current/active group loss, is not routine

---

## 11. Clearing Redo Logs: Use With Suspicion

The `CLEAR LOGFILE` command exists because sometimes you cannot drop and recreate
the group cleanly.

Oracle allows it, but the warnings are not decorative.

If you clear a log that is needed for recovery:

- some backups become unusable for complete recovery

After clearing an unarchived redo log, Oracle recommends taking a fresh backup.

That advice is sound and should be treated as mandatory unless you enjoy finding
out later that your recovery plan had a hole in it the size of a missing log
group.

---

## 12. Recovering from Password File Loss

The password file is used for privileged administrative authentication, such as:

- `SYSDBA`
- `SYSOPER`
- `SYSBACKUP`
- `SYSDG`
- `SYSKM`

It is **not** the general user authentication store for the whole database.

If the password file is lost, privileged remote logins fail.

The fix is to recreate it with `orapwd`.

Example:

```powershell
orapwd FILE='/u01/app/oracle/dbs/orapworcl' SYS=password FORMAT=12.2
```

Important corrections to the source:

- current Oracle docs for Unix/Linux describe the required password file
  location as `ORACLE_BASE/dbs`
- not the older simplified `ORACLE_HOME/dbs` phrasing often seen in training
- the exact required name is platform-specific

Also important:

- `REMOTE_LOGIN_PASSWORDFILE` must be configured appropriately
- granting admin privileges such as `SYSDBA` adds users to the password file
  when the password file is in the right mode

If you change or recreate the file in a new location, Oracle documents that you
may need:

```sql
ALTER SYSTEM FLUSH PASSWORDFILE_METADATA_CACHE;
```

to make the new file metadata take effect cleanly.

---

## 13. Practical Takeaways

- A lost `SPFILE` is usually a startup problem, and RMAN autobackups can save
  you.
- If one current control file copy survives, copy it and move on. If all copies
  are gone, you are in backup-control-file territory and likely headed for
  `RESETLOGS`.
- `NOLOGGING` operations do not magically become recoverable because you regret
  them later.
- Redo log **member** loss is often manageable. Redo log **group** loss is where
  things become much less funny.
- `INACTIVE` does not mean the same thing as `ARCHIVED`.
- Clearing an unarchived redo log is possible, but it is a deliberately risky
  act.
- Password file loss is annoying rather than existential, but it still matters
  for admin access and should be recreated correctly.

This lesson is a tour of Oracle's supporting cast of disaster.
Not every missing file destroys the database.
Some just remove the steering wheel, the ignition key, the dashboard, or the
brakes one at a time.

---

## 14. Practice 6.1 - Recovering from the Loss of a Parameter File

The first lab creates the mild but very annoying startup failure where the
instance cannot find `initorclcdb.ora`.

The useful sequence is:

1. run the setup script to create a small recoverable workload
2. run the break script to remove the parameter file
3. confirm that `STARTUP` fails in `SQL*Plus`
4. connect to `RMAN`
5. restore the `SPFILE` from autobackup
6. restart the instance normally
7. clean up and re-establish the backup baseline

What the lab is really teaching is this:

- a missing parameter file is often a startup problem, not a data-loss problem
- if control file autobackup is configured, restoring the `SPFILE` is usually
  straightforward

The narration says RMAN starts with a "dummy parameter file."
That is a fuzzy way of describing the fact that RMAN can start enough of the
instance in `NOMOUNT` to restore the real `SPFILE`, provided you also tell RMAN
where to look for the autobackup when it cannot infer the location.

So the lab conceptually boils down to:

```rman
RESTORE SPFILE FROM AUTOBACKUP
  DB_RECOVERY_FILE_DEST '/u01/app/oracle/fast_recovery_area'
  DB_NAME 'orclcdb';
SHUTDOWN IMMEDIATE;
STARTUP;
```

The follow-up backup at the end is not busywork.
It restores the environment to a sane post-lab baseline before the next
practice starts breaking other files for sport.

---

## 15. Practice 6.2 - Restoring One Lost Control File

The second lab deletes one control file copy and then walks through diagnosis
with:

- `ADRCI`
- `LIST FAILURE`
- `ADVISE FAILURE`
- `REPAIR FAILURE`

This is historically understandable.
It is also not the path you should instinctively reach for in `19c`.

The practical lesson is simpler:

- if one current multiplexed control file copy still exists, you usually recover
  fastest by copying the good one over the missing one while the instance is
  down, or by copying it to a new location and updating `CONTROL_FILES`

That means the lab is really useful for two things:

1. learning how the failure appears in the alert log
2. seeing Oracle identify the missing control file clearly

The DRA-driven repair flow in the lab is legacy context, not modern best
practice.

The meaningful technical takeaway is:

- one good current control file copy saves you from needing backup-control-file
  recovery
- this is a file replacement problem, not a full restore-and-recover drama

The lab's cleanup and post-lab backup are again important because the later
practices assume a healthy baseline.

---

## 16. Practice 6.3 - Recovering from the Loss of All Control Files

This is the real control-file recovery lab.

Once both control files are gone, the recovery logic changes completely.
Now you are no longer copying a surviving current file.
You are restoring a backup control file and resynchronizing the database to it.

The lab usefully demonstrates:

1. startup fails because no control files can be identified
2. `ADRCI` confirms both files are missing
3. `LIST FAILURE` and `ADVISE FAILURE` identify the damage
4. the cleaner practical route is to restore the control file manually
5. mounting succeeds
6. `ALTER DATABASE OPEN` fails because `RESETLOGS` is required
7. `OPEN RESETLOGS` still fails until the database is recovered
8. `RECOVER DATABASE` synchronizes the control file and data files
9. `ALTER DATABASE OPEN RESETLOGS` succeeds
10. the `PDB` is reopened and verified

That sequence matters because it teaches the exact thing people get wrong under
stress:

- restoring the backup control file is not the end
- you still need recovery
- and then you still need `RESETLOGS`

So the lab's manual path is the right one:

```rman
RESTORE CONTROLFILE FROM AUTOBACKUP;
ALTER DATABASE MOUNT;
RECOVER DATABASE;
ALTER DATABASE OPEN RESETLOGS;
```

Then in SQL:

```sql
ALTER PLUGGABLE DATABASE orclpdb1 OPEN;
```

The optional `V$DATABASE` query in both root and `PDB` is a good verification
habit. The exact `CURRENT_SCN` values will vary slightly because the database is
alive and doing database things again.

And yes, the final backup after `RESETLOGS` is mandatory in spirit even when a
lab workbook pretends it is merely the next step.

---

## 17. Practice 6.4 - Restoring the Password File

This is the most recoverable-looking disaster in the set because the database
itself is still fine.
What fails is privileged remote authentication.

The lab flow is:

1. remove the password file
2. attempt remote `SYSDBA` login
3. confirm the login fails
4. inspect `orapwd` usage
5. recreate the password file
6. test the remote login again
7. query `V$PWFILE_USERS`

That is a clean little exercise, and it teaches two practical truths.

### First: `orapwd` is the actual fix

The key command is conceptually:

```powershell
orapwd file=$ORACLE_BASE/dbs/orapworclcdb entries=15
```

And then provide the `SYS` password when prompted, or use the supported command
syntax for your platform and version.

The lab's failed short password attempt is also useful.
Oracle enforces password complexity rules here too, because of course it does.

### Second: recreating the password file does not resurrect everyone

After recreating the file, the lab queries:

```sql
SELECT * FROM v$pwfile_users;
```

and sees only `SYS`.

That is the part people should actually remember.

If you had other administrative users previously granted privileges such as:

- `SYSDBA`
- `SYSBACKUP`
- `SYSDG`
- `SYSKM`

then recreating the password file means those entries must be re-established by
granting the privileges again.

So password-file recovery is not just "make the file exist."
It is also:

- verify who is in it now
- re-grant administrative privileges where needed

Which is why this practice is more useful than it first appears.

---

## 18. What These Practices Add to the Lesson

The four labs together reinforce a pattern that matters more than the specific
commands:

- not every file loss requires the same scope of response
- surviving current copies are much better than restored backups
- backup control files drag `RESETLOGS` and incarnation changes into your life
- password file loss is an authentication failure, not a data recovery failure

They also quietly demonstrate a more general Oracle truth:

- the fastest recovery is usually the one that uses the most current surviving
  file
- the more you fall back into backed-up metadata, the more ceremony Oracle
  demands before it trusts you again

And frankly, that is one of the more reasonable instincts the product has.
