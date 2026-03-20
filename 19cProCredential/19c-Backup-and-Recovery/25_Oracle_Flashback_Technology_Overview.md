## Lesson 25 - Oracle Flashback Technology Overview (in which Oracle offers you time travel, but only within the limits of undo, logging, privilege, and basic reality)

And look, Oracle Flashback sounds like one giant feature that rewinds anything you regret. It is not. It is a family of features, each with different storage requirements, scope, privileges, and ways to disappoint you if you assume they are interchangeable.

By the end of this lesson, you should be able to:

- Explain what Oracle Flashback Technology is for
- Distinguish undo-based flashback features from `FLASHBACK DATABASE`
- Identify when Flashback Query, Version Query, Transaction Query, Flashback Table, Flashback Drop, or Flashback Database fits the problem
- Understand the role of undo, flashback logs, and the recycle bin
- Recognize the main prerequisites and limitations, especially in multitenant environments

---

## 1. What Flashback Is Actually Trying to Save You From

Oracle Flashback Technology exists to help you recover from **logical** mistakes
faster than a full restore-and-recovery cycle.

Typical examples:

- someone updated the wrong rows
- someone deleted the right rows for the wrong reason
- someone dropped the table they absolutely meant to keep
- someone truncated something and then developed immediate spiritual growth

The point of Flashback is:

- use data already retained in the database environment
- avoid hauling backups into the room unless you really have to

This is why Flashback is often much faster than traditional media recovery for
logical damage.

But not all Flashback features use the same source of truth.
That is the first thing you need to keep straight.

---

## 2. Undo Is the Center of Gravity for Most Flashback Features

The source correctly starts with transactions, buffer cache, locks, and undo.

That matters because most Flashback features rely on **undo data**.

When you modify data, Oracle records enough undo information to:

- roll back uncommitted work
- provide read consistency
- support certain Flashback operations against committed data

Undo is therefore not some cute side channel.
It is the historical memory that lets Oracle pretend your last commit was maybe
not your best work.

Important `19c` point from Oracle docs:

- undo data persists across shutdown
- retention depends on `UNDO_RETENTION` and tuned undo retention under AUM

So if you want Flashback features that rely on undo to work reliably, you need:

- Automatic Undo Management
- enough undo space
- realistic retention planning

Because "I assumed the database remembered" is not a valid retention strategy.

---

## 3. One Big Distinction: Undo-Based Flashback vs `FLASHBACK DATABASE`

The source eventually gets here, but it helps to say it plainly.

### Undo-based Flashback features

These rely mainly on undo data:

- Flashback Query
- Flashback Version Query
- Flashback Transaction Query
- Flashback Transaction Backout
- Flashback Table to timestamp or SCN

### Not undo-based in the same way

- Flashback Drop uses the **recycle bin**
- Flashback Data Archive / Flashback Time Travel uses **history tables in
  flashback archives**
- `FLASHBACK DATABASE` uses **flashback logs** in the fast recovery area

That means these are not interchangeable.

If you confuse:

- undo
- recycle bin
- flashback archive
- flashback logs

then Oracle will happily correct you by refusing to do what you assumed it
could do.

---

## 4. Flashback Query: "Show Me the Data as It Was"

Flashback Query lets you retrieve past committed data using:

- `AS OF TIMESTAMP`
- `AS OF SCN`

Example:

```sql
SELECT *
FROM hr.employees AS OF TIMESTAMP
     TO_TIMESTAMP('2026-03-20 10:15:00','YYYY-MM-DD HH24:MI:SS');
```

Or:

```sql
SELECT *
FROM hr.employees AS OF SCN 3817001;
```

This does **not** change data.
It is purely a query mechanism.

It is useful for:

- comparing current and prior values
- reconstructing accidentally changed rows
- identifying the point in time you may want to rewind to

Important Oracle nuance:

- time-to-SCN mapping is only at about 3-second granularity
- `SCN` is more precise than time for exact targeting

So if you want precision, use `SCN`.
If you want convenience, use time and accept that Oracle rounds like a bureaucracy with a stopwatch.

---

## 5. Flashback Version Query: "Show Me Every Version"

Version Query shows how rows changed over a period of time.

Use the `VERSIONS BETWEEN` clause:

```sql
SELECT versions_startscn,
       versions_endscn,
       versions_operation,
       versions_xid,
       employee_id,
       salary
FROM hr.employees
VERSIONS BETWEEN SCN 3816000 AND 3817000
WHERE employee_id = 100;
```

This is useful when:

- multiple changes happened
- you need to know which version is the right one
- you want transaction IDs for later analysis

Again, this is read-only investigation.
It does not rewind anything by itself.

It is the forensic phase, not the cleanup crew.

---

## 6. Flashback Transaction Query: "Show Me the Transaction History"

Flashback Transaction Query reads from:

- `FLASHBACK_TRANSACTION_QUERY`

This view shows:

- transaction ID
- start/commit SCNs
- start/commit timestamps
- user
- operation
- `UNDO_SQL`

Example:

```sql
SELECT xid,
       operation,
       commit_scn,
       logon_user,
       table_name,
       undo_sql
FROM flashback_transaction_query
WHERE table_owner = 'HR';
```

This is powerful because it tells you not just what changed, but the SQL Oracle
believes could logically undo it.

Important prerequisite from Oracle docs:

- at least minimal supplemental logging should be enabled

Important multitenant caveat:

- Oracle's `19c` flashback documentation describes Flashback Transaction Query as
  unsupported in a CDB
- error documentation also specifically calls out shared undo mode as a cause of
  failure

So if you are in a multitenant environment, do not casually assume transaction
query/backout will work exactly like your nostalgic single-instance lecture.

---

## 7. Flashback Transaction Backout: "Undo This Transaction, Not the Whole Table"

Flashback Transaction Backout is done through:

- `DBMS_FLASHBACK.TRANSACTION_BACKOUT`

It creates compensating transactions to reverse a committed transaction while
the database remains online.

This is much more delicate than the cheerful marketing version suggests.

Oracle documents restrictions, including cases involving:

- DDL that changed logical table structure
- LOB usage
- unsupported LogMiner scenarios
- complex dependency graphs

And yes, dependencies matter.
Deeply.

Oracle gives you options such as:

- `NOCASCADE`
- `CASCADE`
- `NOCASCADE_FORCE`
- `NONCONFLICT_ONLY`

Which is Oracle's way of asking:

- "Would you like surgical reversal, cautious reversal, or some controlled damage in exchange for speed?"

Also important:

- the procedure does not commit the compensating work automatically
- you must decide whether to `COMMIT` or `ROLLBACK`

So this is not a casual button.
This is a scalpel.

---

## 8. Flashback Table: Rewind One Table

Flashback Table lets you move a table back to a previous SCN or timestamp while
the database stays online.

Example:

```sql
FLASHBACK TABLE hr.employees
TO SCN 3817001;
```

Or:

```sql
FLASHBACK TABLE hr.employees
TO TIMESTAMP TO_TIMESTAMP('2026-03-20 10:15:00','YYYY-MM-DD HH24:MI:SS');
```

Important prerequisite:

- row movement must be enabled

Example:

```sql
ALTER TABLE hr.employees ENABLE ROW MOVEMENT;
```

If you forget this, Oracle answers with the memorable:

- `ORA-08189: cannot flashback the table because row movement is not enabled`

Why row movement matters:

- rowids may change after the flashback

So this is one of those features that works beautifully right after you satisfy
the prerequisite you forgot five minutes ago.

---

## 9. Flashback Drop: Recycle Bin, Not Undo

This is where the source blurs things a little.

If a table is dropped and the recycle bin is enabled, then you can often recover
it with:

```sql
FLASHBACK TABLE hr.regions_hist TO BEFORE DROP;
```

That uses:

- the recycle bin

It does **not** use undo in the ordinary query sense.
It does **not** use flashback logs.

Important `19c` fact:

- `RECYCLEBIN` defaults to `ON`

And an equally important consequence:

- `PURGE` removes the object from the recycle bin
- after that, Flashback Drop is no longer an option

So "we can always flash it back later" is true only until someone types
`PURGE`, at which point Oracle becomes a much less sentimental product.

---

## 10. Flashback Database: Rewind the Whole Database

`FLASHBACK DATABASE` is a separate administrative feature.

It rewinds the database to a previous time or SCN using:

- flashback logs in the fast recovery area

This is **not** the same thing as querying older row versions from undo.

It is also not a fix for media failure.

Oracle explicitly documents that `FLASHBACK DATABASE` is for rewinding database
changes, not for repairing:

- lost data files
- damaged data files
- restored or recreated control file scenarios

And after flashing back the database, you must reopen with:

```sql
ALTER DATABASE OPEN RESETLOGS;
```

So yes, whole-database rewind still carries incarnation consequences.
Oracle is not going to let you casually reverse time and then pretend history
continued in a straight line.

---

## 11. Flashback Data Archive / Flashback Time Travel

The source calls this "temporal data" or "data archive," which is close enough
to the idea but muddy on the implementation.

In current Oracle terminology, this is:

- Flashback Archive
- also described in newer docs as Flashback Time Travel

This feature:

- automatically stores historical versions of enabled tables
- uses dedicated archive history storage
- is meant for long-term historical querying, auditing, and compliance

This is different from ordinary undo-based Flashback because:

- it is designed for longer retention
- it avoids the ordinary undo-window limitations

So if the requirement is:

- "show me what this row looked like months ago for audit purposes"

then undo is not the grown-up answer.
Flashback Archive is.

---

## 12. What Each Flashback Feature Is Good For

### Use Flashback Query when:

- you want to see old committed data
- you need comparison, not recovery yet

### Use Flashback Version Query when:

- you want all row versions over a period
- you need transaction IDs and sequence of changes

### Use Flashback Transaction Query when:

- you need transaction history and undo SQL

### Use Flashback Transaction Backout when:

- one committed transaction should be reversed online
- dependencies are manageable

### Use Flashback Table when:

- one table should go back in time
- row movement is enabled

### Use Flashback Drop when:

- a table was dropped
- recycle bin still has it

### Use Flashback Database when:

- the whole database must be rewound quickly
- flashback logging was configured ahead of time

### Use Flashback Archive / Time Travel when:

- long-term historical access is required

The common theme is:

- choose the smallest feature that fixes the actual mistake

Because rewinding the whole database to correct one bad `UPDATE` is technically
possible in the same way that remodeling the house to replace one light switch
is technically possible.

---

## 13. What the Source Gets Right, and Where It Wanders

The source is correct that:

- Flashback often avoids using backups
- undo is central to many Flashback features
- Flashback Query and Version Query are investigative
- Flashback Table and Flashback Transaction Backout actually change data

Where it wanders:

- it blurs undo-based Flashback with Flashback Database
- it treats recycle-bin recovery as if it were just another undo trick
- it understates prerequisites such as row movement, supplemental logging, and
  flashback logging
- it makes transaction-based flashback sound more universally available than it
  really is in multitenant environments, especially in `19c` CDBs

So the cleaned-up rule set is:

- undo-based features inspect or rewind within the undo window
- recycle-bin recovery restores dropped objects from the recycle bin
- Flashback Archive stores long-term history
- Flashback Database rewinds the whole database using flashback logs

Those are cousins, not clones.

---

## 14. Practical Takeaways

- Flashback is for logical mistakes, not ordinary media failure repair.
- Most row-level flashback features depend on undo and therefore on undo
  retention and undo capacity.
- Flashback Drop depends on the recycle bin, not on undo or flashback logs.
- Flashback Table requires row movement for ordinary rewind-to-time operations.
- Flashback Transaction Query and Backout have real prerequisites and
  multitenant restrictions.
- Flashback Database is a separate, bigger feature using flashback logs in the
  FRA and requiring `RESETLOGS` afterward.
- Flashback Archive is for long-term history, not just short-term regret.

Oracle Flashback Technology is excellent, but only if you stop thinking of it
as one magic undo button.
It is really a toolkit of different time-travel privileges, each with its own
costs, scope, and opportunities for administrative embarrassment.

---

## 15. Demo and Practice - Preparing to Use Flashback Technologies

The preparation demo and the first practice are mostly about making sure the
database can actually remember enough of the past for logical flashback features
to be useful.

That means checking:

- tuned undo retention
- undo management mode
- undo tablespace sizing
- whether retention is merely hoped for or actually guaranteed
- whether the recycle bin is enabled

This is good operational prep, because Flashback features are wonderful right up
until the database calmly tells you it no longer remembers the time period you
care about.

---

## 16. Reading `V$UNDOSTAT` Properly

The source queries:

```sql
SELECT tuned_undoretention
FROM v$undostat
WHERE ROWNUM = 1;
```

In this specific case, that works because Oracle documents `V$UNDOSTAT` as being
ordered by descending `BEGIN_TIME`, so the first row is the current partial
interval.

What that value means:

- the latest tuned amount of time, in seconds, for which committed undo is
  expected not to be recycled

It does **not** mean:

- a contractual promise from Oracle that your exact row versions will be there
  forever because you felt optimistic

If the demo reports `1182`, that means roughly:

- about 19.7 minutes of currently tuned undo retention

If the practice reports `900`, that is simply:

- the normal 15-minute default threshold at that moment

Both are snapshots of current behavior, not eternal truths engraved in redo.

---

## 17. Setting `UNDO_RETENTION` More Intentionally

The source increases undo retention to four hours:

```sql
ALTER SYSTEM SET undo_retention = 14400 SCOPE = BOTH;
```

That is perfectly reasonable for a lab.

But there are two important Oracle nuances:

### First

`UNDO_RETENTION` is a **low threshold**, not an absolute guarantee.

Oracle can still overwrite unexpired undo when space pressure requires it,
unless retention guarantee is enabled.

### Second

In multitenant environments, changing `UNDO_RETENTION` at the `PDB` level is
only supported when the database is using **local undo**.

So if the lab sets it inside `orclpdb1`, the hidden assumption is:

- local undo mode is in effect

That is fine in a lab.
It is just one of those multitenant details Oracle prefers you learn after
reading the small print.

---

## 18. Retention Guarantee: Helpful, Dangerous, Honest

The lab then does:

```sql
ALTER TABLESPACE undotbs1 RETENTION GUARANTEE;
```

This is the step that turns:

- "please try to keep enough undo"

into:

- "do not overwrite unexpired undo even if active DML starts suffering"

Oracle's documentation is very clear about the tradeoff:

- retention guarantee can cause DML to fail if the undo tablespace runs out of
  space

So the source is right to warn that active user work may fail.
That is not a side effect.
That is the actual price of the guarantee.

In other words, retention guarantee is excellent if your real concern is:

- "Flashback must work"

and less excellent if your real concern is:

- "transactions should continue succeeding while I pretend storage is infinite"

---

## 19. Autoextend on the Undo Datafile

The practice then checks the undo datafile and enables autoextend:

```sql
ALTER DATABASE DATAFILE 11 AUTOEXTEND ON MAXSIZE UNLIMITED;
```

That is good lab hygiene, because undo guarantee without room to grow is just a
beautiful way to manufacture user anger.

Practical correction:

- `MAXSIZE UNLIMITED` still means "up to Oracle and file-type limits"
- it does **not** mean "this file may consume the known universe"

So the narration's aside about smallfile, bigfile, and platform/file limits is
directionally correct.

---

## 20. Recycle Bin Check as Part of Flashback Prep

The preparation practice ends by checking:

```sql
SHOW PARAMETER recyclebin;
```

That belongs in flashback prep because Flashback Drop depends on the recycle
bin.

If the parameter is `ON`, then:

- dropped tables can go into the recycle bin

If it is `OFF`, then:

- new drops bypass it

But remember the correction from the later lesson:

- turning it off does not purge objects already stored there

So "recycle bin is on" is not just a trivia point.
It changes whether Flashback Drop is even a viable answer for the next mistake.

---

## 21. What Practice 2.1 Actually Teaches

Practice 2.1 is really teaching the prerequisites mindset:

- Flashback is not just a command you run after a mistake
- Flashback is something you prepare for before the mistake

The useful operational sequence is:

1. start the database and connect to the right `PDB`
2. inspect current undo-retention reality
3. increase `UNDO_RETENTION` if needed
4. decide whether to enable `RETENTION GUARANTEE`
5. make sure the undo datafile can grow
6. confirm the recycle bin setting

That is a good practice because it forces the most important realization:

- if the system is not retaining the past, no amount of later confidence will
  make Flashback work anyway
