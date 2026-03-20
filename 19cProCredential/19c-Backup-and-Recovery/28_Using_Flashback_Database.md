## Lesson 28 - Using Flashback Database (in which Oracle lets you rewind the whole database quickly, provided you prepared the FRA, kept the control file current, and did not make history impossible first)

And look, Flashback Database is the feature people hear about and immediately imagine as a giant universal undo button. It is not. It is a fast whole-database rewind mechanism that depends on flashback logs, archived redo, a current control file, and a distressing number of prerequisites that Oracle will enforce with enthusiasm.

By the end of this lesson, you should be able to:

- Explain how Flashback Database differs from undo-based flashback features
- Configure the database to support Flashback Database correctly
- Understand the roles of `DB_FLASHBACK_RETENTION_TARGET`, the FRA, RVWR, and flashback logs
- Use `FLASHBACK DATABASE` for a whole CDB or for a `PDB` where supported
- Explain guaranteed restore points and clean `PDB` restore points
- Recognize the major limitations that prevent Flashback Database from working

---

## 1. Flashback Database Is Not "Flashback Query but Bigger"

The source starts by mixing:

- undo retention
- undo tablespaces
- flashback query
- Flashback Database

Those are related only in the broad sense that they all involve time and regret.

For Flashback Database itself, the core requirements are:

- database in `ARCHIVELOG` mode
- fast recovery area configured
- flashback logging enabled, unless you are rewinding to a guaranteed restore
  point
- current control file

Undo settings matter for:

- Flashback Query
- Flashback Table
- other undo-based logical flashback features

They are **not** the primary enabling architecture for Flashback Database.

Flashback Database works mainly by writing **before-images of changed blocks**
into flashback logs in the fast recovery area, not by relying on your undo
tablespace to remember the entire world.

So if someone says:

- "We set `UNDO_RETENTION`, therefore Flashback Database is ready"

that person is about to have a very educational outage.

---

## 2. What Flashback Database Is Actually For

Flashback Database is a fast alternative to incomplete recovery when the problem
is logical and you want to rewind the whole database or a supported `PDB`.

Typical use cases:

- bad batch job
- bad deployment
- bad patch
- bad data fix
- bad human confidence

It is faster than restore-and-recover because it rewinds changed blocks using
flashback logs instead of restoring data files from backup.

Important limit:

- this is not media-failure repair
- it does not replace backups

If a data file is gone, Flashback Database is not your savior.
If the data is merely wrong and the files are intact, now we are talking.

---

## 3. The Core Architecture

The source's architectural picture is basically correct.

The important moving parts are:

- redo buffer and redo logs for moving the database **forward**
- flashback buffer and flashback logs for moving the database **backward**
- `RVWR` (Recovery Writer) for writing flashback data

When Flashback Database is enabled:

- before-images of changed data blocks are written to flashback logs
- those logs live in the fast recovery area as Oracle-managed files

So:

- redo says "apply the change"
- flashback logs say "here is what the block looked like before that"

That is why Flashback Database can rewind quickly without a conventional datafile
restore.

It is not magic.
It is just Oracle keeping a second, more pessimistic diary.

---

## 4. Real Prerequisites

To use Flashback Database in `19c`, Oracle requires:

- `ARCHIVELOG` mode
- fast recovery area configured
- database mounted with a **current** control file
- flashback logging enabled with `ALTER DATABASE FLASHBACK ON`, unless you are
  flashing back to a guaranteed restore point

And the administrative privileges required are:

- `SYSDBA`
- `SYSBACKUP`
- `SYSDG`

Important correction to the source:

- there is no generic little "flashback privilege" you hand out to random users
  for whole-database rewind

This is administrative work.
Oracle would prefer not to let the summer intern time-travel the CDB.

---

## 5. `DB_FLASHBACK_RETENTION_TARGET`: A Goal, Not a Guarantee

The database-wide parameter is:

```sql
ALTER SYSTEM SET db_flashback_retention_target = 1440 SCOPE = BOTH;
```

This value is in:

- **minutes**

The default is:

- `1440` minutes
- which is one day

Important correction:

- this parameter is only a **target**
- it does **not** guarantee that flashback logs for that whole period will
  remain available

If there is pressure in the fast recovery area, Oracle can delete older
flashback logs.

If you need a true guarantee for a known point, use:

- guaranteed restore points

That is the grown-up answer when "I really must get back to this exact moment"
matters more than "I sort of hoped there would still be enough space."

---

## 6. Enabling Flashback Database

The normal enablement sequence is:

```sql
ALTER SYSTEM SET db_recovery_file_dest_size = 50G SCOPE = BOTH;
ALTER SYSTEM SET db_recovery_file_dest = '/u01/app/oracle/fast_recovery_area' SCOPE = BOTH;
ALTER SYSTEM SET db_flashback_retention_target = 1440 SCOPE = BOTH;
```

Then, with the database mounted:

```sql
ALTER DATABASE FLASHBACK ON;
```

Check status:

```sql
SELECT flashback_on FROM v$database;
```

Important correction to the source:

- you do **not** enable Flashback Database by tuning undo retention and hoping
  hard
- you enable it by configuring the FRA and turning flashback logging on

Undo still matters elsewhere in the flashback family, just not as the primary
engine here.

---

## 7. Monitoring the Flashback Window

The source is right that you should inspect how much rewind is really available.

Useful view:

```sql
SELECT oldest_flashback_scn,
       oldest_flashback_time,
       estimated_flashback_size
FROM v$flashback_database_log;
```

This tells you:

- how far back you can currently flash back
- the approximate storage Oracle thinks the flashback logs need

And for activity stats:

```sql
SELECT *
FROM v$flashback_database_stat;
```

That view helps you understand logging rates and flashback generation behavior
over time.

Practical Oracle guidance:

- a rough rule of thumb is that flashback log generation is on the same order of
  magnitude as redo generation

So if your database produces lots of redo, your FRA had better not be sized like
a fragile personal opinion.

---

## 8. Whole-Database Flashback Workflow

The basic whole-database flow is:

1. shut down and mount the database
2. issue `FLASHBACK DATABASE`
3. open with `RESETLOGS`

Example by time:

```sql
SHUTDOWN IMMEDIATE;
STARTUP MOUNT;
FLASHBACK DATABASE TO TIMESTAMP
  TO_TIMESTAMP('2026-03-20 10:15:00','YYYY-MM-DD HH24:MI:SS');
ALTER DATABASE OPEN RESETLOGS;
```

Or by `SCN`:

```sql
FLASHBACK DATABASE TO SCN 3817001;
```

Or by restore point:

```sql
FLASHBACK DATABASE TO RESTORE POINT before_upgrade;
```

Important `19c` requirement:

- the database must be **mounted but not open** for the flashback operation

The source says to open read only to inspect before `RESETLOGS`.
That can be part of a cautious operational workflow, but the authoritative rule
is:

- perform the flashback while mounted
- reopen read/write with `RESETLOGS` when satisfied

So if you do use a read-only validation pass, treat it as an extra validation
step, not the core documented mechanism.

---

## 9. `RESETLOGS` Still Happens

Flashback Database is faster than restore-and-recover, but it still changes the
database timeline.

So after flashing back and reopening read/write, you must:

```sql
ALTER DATABASE OPEN RESETLOGS;
```

That means:

- new incarnation
- new redo stream

Flashback Database is fast, but it is not a magical way to reverse history while
pretending history continued uninterrupted.

Oracle insists on paperwork even for time travel.

---

## 10. Limitations You Ignore at Your Own Expense

The source lists several limits. Here is the cleaned-up version.

You cannot use Flashback Database if:

- the control file has been restored from backup
- the control file has been re-created
- a data file was shrunk at the target time boundary in ways Oracle cannot
  reverse cleanly
- a tablespace drop or similar structural change prevents the rewind of the
  affected files
- any online tablespace has `FLASHBACK OFF`

The most important one operationally is:

- Flashback Database requires a **current** control file

If the control file has been restored or recreated, all existing flashback log
information is effectively unusable for Flashback Database.

Which is Oracle's way of saying:

- "No, you cannot damage the metadata, replace it, and then expect my old
  time-travel logs to make perfect sense."

---

## 11. `TO BEFORE RESETLOGS`

The source mentions the `TO BEFORE RESETLOGS` option.

That is real and useful when you need to flash back to the database state just
before the last `OPEN RESETLOGS`.

Example:

```sql
FLASHBACK DATABASE TO BEFORE RESETLOGS;
```

This is one of those options that sounds bizarre until you actually need it,
and then it suddenly feels like Oracle did one clever thing on purpose.

---

## 12. Flashback Database and `NOLOGGING`

The source is a little too cheerful here.

Oracle documentation warns that if you flash back to a point **during** a
`NOLOGGING` operation, affected objects can be left with block corruption.

So the safer rule is:

- avoid target times that intersect active `NOLOGGING` operations
- take a fresh backup after `NOLOGGING` work

Do not assume Flashback Database makes `NOLOGGING` harmless.
It does not.
It just changes how the regret arrives.

---

## 13. Guaranteed Restore Points

A guaranteed restore point is an `SCN` alias that Oracle promises you can flash
back to.

Create one like this:

```sql
CREATE RESTORE POINT before_upgrade GUARANTEE FLASHBACK DATABASE;
```

Why people love them:

- exact known rewind point
- they never age out of the control file until explicitly dropped

And here is the important Oracle nuance the source gets mostly right:

- you can use a guaranteed restore point even when Flashback Database logging is
  otherwise disabled

When flashback logging is disabled and you create a guaranteed restore point,
Oracle still logs changed blocks as needed to support flashing back to that
restore point.

But there is a catch:

- without full flashback logging, you can flash back directly to the guaranteed
  restore point
- you cannot freely flash back to arbitrary intermediate `SCN`s between that
  point and now

So guaranteed restore points are brilliant for:

- "take me back to exactly before the upgrade"

They are less brilliant for:

- "take me back to some fuzzy point two hours after the restore point"

That is not what they are for.

---

## 14. FRA Pressure and Guaranteed Restore Points

Guaranteed restore points can keep required flashback logs from being reused.

That means:

- the fast recovery area can fill up
- if nothing is eligible for deletion, the database can behave as though the
  disk is full

This is not an abstract risk.
Oracle documentation is very blunt about it.

So when using guaranteed restore points:

- size the FRA generously
- drop the restore point as soon as it is no longer needed

Example:

```sql
DROP RESTORE POINT before_upgrade;
```

If you forget this step, Oracle may eventually remind you in the least friendly
way possible: by running out of recovery area while your production system is
trying to breathe.

---

## 15. Flashbacking a `PDB`

The source says you cannot flash back the root on its own, which is basically
the right instinct.

Cleaned up:

- flashing back the root means flashing back the whole `CDB`
- if you want to flash back only one pluggable database, use `FLASHBACK
  PLUGGABLE DATABASE`

Oracle `19c` behavior depends on undo mode.

### Local undo

- no auxiliary instance required
- you can flash back the `PDB` to a restore point, `SCN`, or time

### Shared undo

- an auxiliary destination is required
- in `19c`, you can flash back the `PDB` only to a **clean PDB restore point**

This is a major correction to the source, which makes `PDB` flashback sound more
uniform than it really is.

Oracle documents the differences explicitly.

---

## 16. Clean `PDB` Restore Points

A clean `PDB` restore point:

- is created when the `PDB` is closed
- has no outstanding transactions
- is only applicable in shared undo mode

Example:

```sql
ALTER PLUGGABLE DATABASE my_pdb CLOSE;
CREATE CLEAN RESTORE POINT mypdb_crp_before_patch
  FOR PLUGGABLE DATABASE my_pdb;
ALTER PLUGGABLE DATABASE my_pdb OPEN;
```

For shared undo mode, this is faster than other `PDB` flashback approaches
because Oracle does not need to restore backups during the rewind.

That is why clean guaranteed `PDB` restore points are so popular before:

- patching
- upgrades
- application changes

They are the grown-up version of:

- "let's do something risky, but with an escape hatch."

---

## 17. Best Practices

The source's best-practice section has good instincts.
Here is the cleaned-up version.

### Size the FRA realistically

- use `V$FLASHBACK_DATABASE_LOG`
- estimate based on redo generation
- leave headroom for archived logs, backups, and guaranteed restore points

### Monitor flashback behavior

- `V$FLASHBACK_DATABASE_STAT`
- `V$SESSION_LONGOPS` during actual flashback operations

### Use guaranteed restore points for planned risky changes

- especially upgrades and patching
- especially when you want a single exact rollback target

### Drop guaranteed restore points promptly

- do not let them quietly pin flashback logs forever

### Do not confuse undo tuning with Flashback Database enablement

- undo supports other flashback features
- FRA + flashback logs support Flashback Database

### Remember control-file fragility

- once the control file is restored or recreated, Flashback Database history is
  no longer usable for that path

Because Oracle is willing to do time travel.
It is not willing to do time travel after you changed the map of time.

---

## 18. Practical Takeaways

- Flashback Database is a fast whole-database or supported-`PDB` rewind tool for
  logical problems, not a substitute for backups.
- The core prerequisites are `ARCHIVELOG`, FRA, flashback logging, and a
  current control file.
- `DB_FLASHBACK_RETENTION_TARGET` is only a target, not a guarantee.
- Guaranteed restore points are the real guarantee mechanism, but they can fill
  the FRA if you forget to drop them.
- `RESETLOGS` is required after reopening read/write.
- `PDB` flashback behavior differs sharply between local undo and shared undo.
- Undo retention matters for other flashback features, but it is not the main
  engine of Flashback Database.

Flashback Database is one of Oracle's better recovery features.
It is fast, practical, and deeply useful.
It just stops being magical the moment you forget the storage, logging, or
control-file prerequisites holding the whole trick together.

---

## 19. Demo - Enabling and Using Flashback Database

The source demo is trying to show two related ideas:

1. how to confirm Flashback Database status and enable the right support
2. how to actually flash the database back to a known earlier `SCN`

That is useful, but the lecture tangles together:

- guaranteed restore points
- full flashback logging
- undo tuning
- `PDB` inspection

So here is the cleaned-up version of what the demo is really teaching.

---

## 20. First Check: What Does `FLASHBACK_ON` Actually Say?

The starting query is correct:

```sql
SELECT flashback_on
FROM v$database;
```

Possible values you care about are:

- `NO`
- `YES`
- `RESTORE POINT ONLY`

That third value matters.

If you create a guaranteed restore point without enabling full Flashback
Database logging, `V$DATABASE.FLASHBACK_ON` can show:

- `RESTORE POINT ONLY`

That means:

- you can flash back to guaranteed restore points
- you cannot flash back freely to arbitrary times or `SCN`s the way you can when
  full flashback logging is enabled

So if the source implies that "create restore point, then flashback is on," the
clean Oracle answer is:

- not exactly
- you may only have guaranteed-restore-point rewind capability

And Oracle is annoyingly precise about that distinction.

---

## 21. Guaranteed Restore Point vs Full Flashback Logging

The source demo talks as though you can:

1. create a guaranteed restore point
2. check `FLASHBACK_ON`
3. proceed as though Flashback Database is just generally enabled

That needs a sharper boundary.

### Guaranteed restore point

```sql
CREATE RESTORE POINT rp1 GUARANTEE FLASHBACK DATABASE;
```

This guarantees rewind **to that restore point** if enough FRA space exists.

### Full Flashback Database

```sql
ALTER DATABASE FLASHBACK ON;
```

This enables flashback logging generally so you can later do things like:

- `FLASHBACK DATABASE TO SCN ...`
- `FLASHBACK DATABASE TO TIMESTAMP ...`

Those are not the same capability.

So the practical rule is:

- guaranteed restore point = exact escape hatch
- full flashback logging = broader rewind flexibility

You can want one, the other, or both.

---

## 22. The Demo's HR Test Data Flow

The demo then does something sensible:

1. record the current database `SCN`
2. inspect application data in `PDB1`
3. make known changes
4. commit them
5. verify the changes occurred
6. flash back to the earlier `SCN`
7. reopen read only and validate
8. finally open with `RESETLOGS`

That is a good recovery test pattern.

The query for the original `SCN` is straightforward:

```sql
SELECT current_scn
FROM v$database;
```

The application checks in the demo are also sensible:

```sql
SELECT SUM(salary)
FROM hr.employees;

SELECT COUNT(*)
FROM hr.employees
WHERE department_id = 90;
```

You save those values because after the flashback you want proof, not feelings.

Then the test script changes data and commits.
Only after the commit do those changes become the logical mistake you are
rewinding.

That part of the demo is good and worth preserving conceptually.

---

## 23. The Actual Flashback Operation

The source demonstrates the right target style:

- flash back to the earlier `SCN`, the one captured before the bad changes

The correct mounted-database flow is:

```sql
SHUTDOWN IMMEDIATE;
```

Then in RMAN or SQL, mount the database:

```rman
STARTUP MOUNT;
FLASHBACK DATABASE TO SCN 3827738;
```

If the database is not mounted, Oracle will object.

That is not optional theater.
`FLASHBACK DATABASE` requires the database to be mounted but not open.

The source's example where RMAN complains and the fix is `STARTUP MOUNT` is
actually useful because it teaches the failure mode, not just the happy path.

---

## 24. Read-Only Validation Is Legitimate

After the flashback, the source reopens the database in read-only mode and then
opens the `PDB` in read-only mode to check the results.

That is not the core minimum sequence, but Oracle does document it as a valid
cautious workflow.

So this pattern is legitimate:

```sql
ALTER DATABASE OPEN READ ONLY;
```

Then:

```sql
ALTER PLUGGABLE DATABASE pdb1 OPEN READ ONLY;
ALTER SESSION SET CONTAINER = pdb1;
```

Now verify the data:

```sql
SELECT SUM(salary)
FROM hr.employees;

SELECT COUNT(*)
FROM hr.employees
WHERE department_id = 90;
```

If the values match the saved pre-change state, then you know the flashback
landed where you intended.

This is exactly the kind of discipline that prevents a DBA from confidently
releasing the system at the wrong point in time.

---

## 25. Finalize the Rewind Properly

Once satisfied, the demo does the necessary finalization:

1. return to the root container
2. shut down the read-only open
3. start up mount
4. open with `RESETLOGS`

Conceptually:

```sql
SHUTDOWN IMMEDIATE;
STARTUP MOUNT;
ALTER DATABASE OPEN RESETLOGS;
```

That is correct.

And because this is a `CDB`, you then reopen the `PDB` as needed for normal
read-write operation.

The source also mentions optionally turning flashback off afterward or dropping
restore points.
That is reasonable in a lab where you do not want the FRA filling up forever
just because you left a training safety net lying around.

Examples:

```sql
ALTER DATABASE FLASHBACK OFF;
DROP RESTORE POINT rp1;
```

Only do the first one if you genuinely mean to stop flashback logging.
Do not casually remove your safety system because the demo ended and the
instructor sounded satisfied.

---

## 26. What This Demo Actually Teaches

This demo is valuable because it shows the full rhythm of a Flashback Database
operation:

1. confirm feature state with `V$DATABASE`
2. capture a known-good `SCN`
3. make and commit test changes
4. mount the database
5. flash back to the earlier `SCN`
6. validate in read-only mode
7. reopen with `RESETLOGS`
8. clean up restore points or flashback mode only if appropriate

It also teaches the two distinctions that matter most:

- guaranteed restore point support is not the same thing as full flashback
  logging
- undo tuning helps other flashback features, but flashback logs are the real
  engine here

And frankly, those are exactly the kinds of distinctions Oracle expects you to
miss once before you learn to respect them.

---

## 27. Practice 3.1 - Enabling Flashback Logging

The first practice is the setup lab:

- put the database in `ARCHIVELOG` mode
- confirm Flashback Database status
- enable flashback logging
- take a fresh backup
- create a guaranteed restore point in the `PDB`

This is a useful sequence because it separates:

1. database-wide flashback logging
2. per-`PDB` restore-point convenience

which the lecture sometimes stirs together like they are one feature.

### The core steps

At the root:

```sql
SHUTDOWN IMMEDIATE;
STARTUP MOUNT;
ALTER DATABASE ARCHIVELOG;
ALTER DATABASE OPEN;
ALTER SYSTEM SWITCH LOGFILE;
```

Then verify status:

```sql
SELECT flashback_on
FROM v$database;
```

Then enable flashback logging:

```sql
ALTER DATABASE FLASHBACK ON;
```

And verify again.

That is the real heart of Practice 3.1.

The later backup:

```rman
BACKUP DATABASE PLUS ARCHIVELOG DELETE INPUT;
DELETE NOPROMPT OBSOLETE;
```

is just good hygiene before the next practice starts intentionally damaging the
truth.

### About the guaranteed restore point

The practice then creates:

```sql
CREATE RESTORE POINT rp1 GUARANTEE FLASHBACK DATABASE;
```

That is useful for the next exercise, but remember:

- the guaranteed restore point is not what enabled Flashback Database here
- `ALTER DATABASE FLASHBACK ON` already did that

The restore point is just a pinned rewind marker.

That distinction matters.

---

## 28. Practice 3.2 - Performing Flashback Database on a `PDB`

The second practice records a baseline, makes deliberate bad changes, and then
rewinds the pluggable database.

At a high level, it does this:

1. record the original `SCN`
2. record baseline application values from `HR.EMPLOYEES`
3. run a script that changes the data
4. commit
5. confirm the values changed
6. close the `PDB`
7. flash back the `PDB`
8. reopen with `RESETLOGS`
9. confirm the original values are back
10. drop the restore point

That is a very good lab pattern.

---

## 29. Baseline First, Always

The practice starts by collecting the original state:

```sql
SELECT current_scn
FROM v$database;
```

and:

```sql
SELECT SUM(salary)
FROM hr.employees;

SELECT COUNT(*)
FROM hr.employees
WHERE department_id = 90;
```

That is exactly what you want in a flashback lab.

Because after the rewind, the only meaningful question is:

- did we get back to the known-good state?

Not:

- "the command worked, so I assume reality followed along."

---

## 30. The `PDB` Flashback Command and the Big Caveat

The practice uses:

```sql
ALTER PLUGGABLE DATABASE orclpdb1 CLOSE;
FLASHBACK PLUGGABLE DATABASE orclpdb1 TO SCN 3827738;
ALTER PLUGGABLE DATABASE orclpdb1 OPEN RESETLOGS;
```

That can be valid in `19c`, **but only in the right undo architecture**.

### If the CDB uses local undo

This SQL workflow is fine.

### If the CDB uses shared undo

Oracle documents an important restriction:

- with shared undo, SQL `FLASHBACK PLUGGABLE DATABASE` can only flash back to a
  **clean PDB restore point**

So if your environment uses shared undo, then:

- `FLASHBACK PLUGGABLE DATABASE ... TO SCN ...` in SQL is not the right path

Oracle's own error documentation points you to:

- RMAN for broader PDB flashback cases, or
- a clean `PDB` restore point, or
- local undo mode

This is one of the biggest hidden assumptions in the workbook.

So the clean interpretation is:

- the lab is valid if the environment uses local undo
- otherwise, the workbook is quietly depending on a capability the docs limit

And yes, that is a very Oracle kind of footnote.

---

## 31. Why `OPEN RESETLOGS` Follows the PDB Flashback

The lab first tries:

```sql
ALTER PLUGGABLE DATABASE orclpdb1 OPEN;
```

and then gets an error, after which it uses:

```sql
ALTER PLUGGABLE DATABASE orclpdb1 OPEN RESETLOGS;
```

That is the useful teaching moment.

After flashback, the `PDB` has been rewound to an earlier state.
Its redo stream for future work must begin from that new historical branch.

So yes:

- `RESETLOGS` is required

The reason is the same general reason as with whole-database rewind:

- you are reopening after moving to an earlier point in time

Oracle is not going to let you pretend the old redo future still applies.

---

## 32. Verifying the `PDB` Rewind

After reopening the `PDB`, the practice reconnects to `orclpdb1` and reruns:

```sql
SELECT SUM(salary)
FROM hr.employees;

SELECT COUNT(*)
FROM hr.employees
WHERE department_id = 90;
```

If the values match the originals, then the flashback succeeded logically.

That is the correct validation step.

It is also why the initial data capture mattered.
Without baseline values, you are just staring at numbers and hoping one of them
looks familiar.

---

## 33. Dropping the Restore Point Afterward

The practice ends by dropping the restore point:

```sql
DROP RESTORE POINT rp1;
```

That is good discipline.

Guaranteed restore points pin flashback logs and can put pressure on the fast
recovery area. If the restore point is no longer needed, dropping it is the
responsible thing to do.

It is also how you avoid waking up later to learn that your FRA is full because
last week's "temporary safety net" became a permanent storage hostage.

---

## 34. What These Practices Actually Teach

Together, Practices 3.1 and 3.2 teach the full operational rhythm:

1. prepare the database for flashback logging
2. confirm the feature is truly on
3. create a precise rollback anchor
4. measure the pre-change state
5. make controlled bad changes
6. rewind to the known-good point
7. reopen correctly with `RESETLOGS`
8. validate the result
9. clean up restore points and backup state afterward

And the most important hidden lesson is this:

- Flashback Database labs often look deceptively simple because the environment
  was prepared in advance

In real life, the success of the flashback depends on:

- undo mode
- flashback logging state
- FRA size
- restore-point planning
- and whether your chosen rewind path is actually supported in your specific
  multitenant architecture

Which is why Oracle flashback demos always look smoother than production change
windows do.
