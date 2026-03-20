## Lesson 17 - Understanding Restore and Recovery Technology (in which Oracle teaches you that "bring it back" is actually several different verbs with consequences)

And look, restore and recovery sound like one thing until disaster arrives and Oracle calmly informs you they are, in fact, separate steps, separate tools, and separate opportunities for you to make the day worse.

By the end of this lesson, you should be able to:

- Distinguish restore from recovery
- Explain instance recovery versus media recovery
- Recognize logical failure versus physical failure responses
- Use the right recovery scope: database, tablespace, data file, block, or table
- Explain complete recovery versus point-in-time recovery
- Understand when `RESETLOGS` is required

---

## 1. Start with the Two Most Important Words

### Restore

Restore means:

- put back a file from backup
- or place a backup copy where Oracle can use it

This is the "get the bytes back onto storage" phase.

### Recover

Recover means:

- apply redo and undo logic as needed to make the restored files consistent

This is the "make the data usable again" phase.

So the basic sequence is:

1. restore what is missing or damaged
2. recover it to the desired point

People love to blur these together because English encourages bad habits.
Oracle does not.

---

## 2. Not Every Failure Is the Same

File loss can happen because of:

- user error
- application error
- corruption
- storage failure
- controller or LUN failure
- operating system mistakes

But the response depends on **what** was lost.

Examples of less catastrophic losses:

- a data file in an offline tablespace
- a tempfile
- an old backup piece

Examples of more serious losses:

- current control files
- active online redo logs
- critical data files needed by open parts of the database

So the right first question is not:

- "How do I recover?"

It is:

- "What failed, and how critical was it?"

Because Oracle recovery is very efficient right up until you ask it the wrong
question first.

---

## 3. Physical Failures vs Logical Failures

### Physical failure

Physical failure means the bytes on storage are missing or damaged.

Examples:

- missing data file
- corrupted block
- failed disk or storage path

Typical tools:

- restore from backup
- media recovery
- block media recovery
- image-copy switch

### Logical failure

Logical failure means the bytes are physically present, but the data state is
wrong because of human or application activity.

Examples:

- bad delete
- bad update
- application bug
- unwanted DDL or DML

Typical tools:

- Flashback Query
- Flashback Table
- Flashback Database
- database point-in-time recovery (`DBPITR`)
- tablespace point-in-time recovery (`TSPITR`)
- table recovery

This is the difference between:

- "the file is gone"
- and "the file is there, but what is inside it is now cursed"

An important distinction.

---

## 4. Data Recovery Advisor: Historically Useful, Currently Awkward

Older Oracle training loves the Data Recovery Advisor (`DRA`) because it could:

- list failures
- advise repair options
- generate repair actions
- integrate with RMAN and Cloud Control

That is all true historically.

Important 19c correction:

- `DRA` is deprecated in Oracle Database `19c`
- the RMAN commands `LIST FAILURE`, `ADVISE FAILURE`, `REPAIR FAILURE`, and
  `CHANGE FAILURE` are deprecated
- Oracle documents no replacement feature

So yes, understand what it was for.
No, do not build your operational hopes around it as if it were the hot new
thing.

That ship has sailed, and Oracle has already started removing the furniture.

---

## 5. Media Recovery Options That Actually Matter

When physical damage occurs, Oracle gives you several levels of response.

### Data file media recovery

Used when a data file is damaged or missing and must be restored and recovered.

### Block media recovery

Used when the damage is isolated to specific blocks rather than the whole file.

This is especially attractive because:

- it is surgical
- it avoids restoring an entire file
- it minimizes disruption

### Image copy switch

If you already have an image copy, RMAN can switch the database to use that copy
instead of the original damaged file location.

This can be a wonderfully efficient temporary rescue move, which is far nicer
than waiting on a full restore while everyone nearby learns new swear words.

---

## 6. Flashback Technology for Logical Damage

If the data is wrong but the storage is fine, flashback features may be the
better answer.

### Flashback Query

Used to see data as of an earlier time or SCN.

### Flashback Table

Used to rewind a table to an earlier point in time.

### Flashback Database

Used to rewind the entire database to an earlier point in time using flashback
logs in the Fast Recovery Area.

This is much faster than restoring backups, but only if:

- Flashback Database was enabled beforehand
- the required flashback logs still exist

Which is Oracle's way of saying:
you cannot decide after the disaster that you had a time machine all along.

---

## 7. Recovery Scope: Pick the Smallest Hammer That Works

Oracle gives you multiple recovery scopes.

### Whole database recovery

Use when the failure affects the whole database or you must rewind the entire
environment.

### Tablespace recovery

Use when the damage is contained to one tablespace.

### Data file recovery

Use when only one or a few data files are affected.

### Table recovery

Use when one table needs to be restored from backup-based recovery tooling.

### Block recovery

Use when the problem is just a corrupted block or a few blocks.

The practical rule:

- recover the smallest scope that solves the problem

Because recovering the whole database to fix one damaged corner is the Oracle
equivalent of repainting the house because one light switch looks tired.

---

## 8. Complete Recovery vs Point-in-Time Recovery

### Complete recovery

Complete recovery means:

- restore as needed
- apply all available redo up to the most current consistent point

Goal:

- get back to the point of failure with minimal data loss

This is what most people want when storage failed and the logical data itself
was fine.

### Point-in-time recovery

Point-in-time recovery means:

- restore from a backup
- recover only up to a chosen time, SCN, sequence, or restore point

Goal:

- stop **before** the bad change

This is what you use when the problem is logical and you want to rewind to a
known-good moment.

Cost:

- all transactions after that target point are discarded from the new
  incarnation

Which is why point-in-time recovery is powerful and also the sort of thing that
should not be chosen casually over lunch.

---

## 9. What `RESETLOGS` Really Means

After incomplete recovery or point-in-time recovery, you open the database with:

```sql
ALTER DATABASE OPEN RESETLOGS;
```

Why?

- Oracle starts a new incarnation of the database
- old redo history past the chosen recovery boundary is no longer part of the
  forward future

This is not cosmetic.
It is Oracle formally acknowledging:

- "we are now living on a different branch of history"

So yes, after `DBPITR`, `RESETLOGS` is required.
And yes, everything after the recovery target is gone from this incarnation.

---

## 10. Restore Destinations: The Source Material Hand-Waves This

The lecture suggests restored data just goes to the `FRA` by default.

That is not a good general rule.

More accurate reality:

- standard RMAN restore operations normally restore files to their original
  configured locations unless you redirect them
- `SET NEWNAME`, `AUXILIARY DESTINATION`, or other recovery workflows can place
  files elsewhere
- some specialized recoveries, such as table or tablespace point-in-time
  recovery, use auxiliary destinations and temporary clone environments

So the correct mental model is:

- restore goes where RMAN determines it should go based on the command and
  configuration
- not "everything goes to the FRA because Oracle likes convenience"

Oracle likes complexity far too much for that kind of simplicity.

---

## 11. Tablespace Recovery While the Database Stays Open

One of the most useful recovery ideas is that some recoveries do **not** require
shutting down the whole database.

Typical pattern for a damaged tablespace:

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

This is practical because:

- the damaged object is isolated
- the rest of the database can remain open

Which is far kinder to your users than casually dropping the entire instance off
a cliff because one tablespace had a rough day.

---

## 12. Instance Recovery Is Automatic

Instance failure is different from media failure.

Causes include:

- power outage
- shutdown abort
- instance crash
- critical background process failure

When the instance restarts, Oracle performs automatic instance recovery.

You do **not** manually restore files for this.

The process is:

1. roll forward changes from redo
2. roll back uncommitted work using undo

That is why a crash-recovered database can come back without you manually
rebuilding every file like a panicked carpenter.

---

## 13. How Instance Recovery Really Thinks

Oracle compares metadata and change state across:

- control files
- data files
- redo logs

The control file knows the checkpoint and structural metadata.
The data file headers may lag behind because some changed blocks were still only
in memory when the crash happened.
The redo logs contain the changes needed to catch the files up.

So Oracle does this:

1. examine control file state
2. examine redo
3. detect that data files are behind
4. roll forward the changes
5. open and finish rollback of uncommitted transactions

This is why redo contains more than just the story of committed happiness.
Oracle needs enough change history to recover the instance cleanly, and then let
undo sort out what never truly committed.

---

## 14. Media Failure Recovery Flow

When physical media fails, the general complete-recovery flow is:

1. restore the needed file or files from backup
2. apply archived redo and, if applicable, online redo
3. bring the files back to a consistent point

RMAN simplifies this by:

- choosing the backup to use
- optimizing failover among backup pieces or copies
- applying the redo it can find automatically

Which is good, because nobody wants to spend recovery time manually playing
"which backup piece looks least cursed."

---

## 15. Point-in-Time Recovery Flow

For point-in-time recovery, the broad logic is:

1. choose the target time, SCN, sequence, or restore point
2. restore the needed files from a backup prior to that target
3. recover only up to the chosen target
4. open with `RESETLOGS`
5. validate that you landed where you intended

Important operational truth:

- if you pick the wrong point in time, you may need to start again

Which is why the first DBA question in these situations should be:

- "When did the bad thing happen exactly?"

And the second question should be:

- "How sure are we?"

Because "sometime Tuesday-ish" is not a recovery plan.

---

## 16. Best Practices for Faster Recovery

### Use incrementals intelligently

Recovery speed improves when you reduce how much archived redo must be applied.

That is one reason incrementals matter.

### Cumulative level 1 backups

Pros:

- faster recovery
- fewer incremental pieces to apply

Cons:

- larger backups

### Differential level 1 backups

Pros:

- smaller backups
- less backup storage

Cons:

- more pieces may need to be applied during recovery

So the tradeoff is exactly what you would expect:

- cumulative favors recovery speed
- differential favors storage efficiency

And yes, Oracle once again makes you choose which kind of pain you prefer.

### Keep archived redo available on disk when possible

If recovery has to go fetch archived logs from slower media, recovery time gets
longer.

### Use block media recovery for isolated corruption

If the damage is narrow, act narrowly.

### Size RMAN memory and I/O sensibly

Recovery performance also depends on:

- buffer memory
- I/O throughput
- CPU availability
- storage layout

So backup strategy and recovery strategy are not separate kingdoms.
They are the same political mess viewed from different days.

---

## 17. Practical Takeaways

When something breaks, think in this order:

1. Was it physical or logical?
2. What is the smallest scope that fixes it?
3. Do I need restore plus recovery, or just flashback?
4. Do I want complete recovery or point-in-time recovery?
5. If point-in-time, exactly what target am I choosing?

If you answer those correctly, the commands become much easier.
If you do not, the commands become a very efficient way to make the database
match your misunderstanding.

---

## 18. Wrap-Up

Restore and recovery technology in Oracle is really about choosing the right
response:

- restore versus recover
- physical versus logical
- complete versus point-in-time
- database versus tablespace versus file versus block versus table
- flashback versus backup-based recovery

And once you understand that, Oracle recovery stops looking like dark magic and
starts looking like what it actually is: a collection of powerful tools waiting
for a DBA to choose the least stupid one.
