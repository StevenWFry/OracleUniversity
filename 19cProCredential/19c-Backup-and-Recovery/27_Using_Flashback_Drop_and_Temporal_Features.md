## Lesson 27 - Using Flashback Drop and Temporal Features (in which Oracle stores your dropped objects, your historical rows, and your business-valid time periods in three different ways just to keep you alert)

And look, by now Oracle Flashback has already split itself into multiple overlapping concepts, which is exactly the sort of naming strategy that keeps DBAs humble. This lesson covers three of the most commonly blurred ideas: Flashback Drop and the recycle bin, Flashback Archive history, and temporal validity. They are related. They are not interchangeable. Oracle will be thrilled to prove that if you mix them up.

By the end of this lesson, you should be able to:

- Explain how Flashback Drop and the recycle bin work
- Recover a dropped table when the recycle bin still has it
- Bypass or purge the recycle bin intentionally
- Create and use a Flashback Archive for longer-term row history
- Understand user-context capture, DDL support, and disassociate/reassociate operations for Flashback Archive
- Distinguish temporal validity from transaction-time history

---

## 1. Flashback Drop Is a Recycle-Bin Feature, Not an Undo Trick

When you drop a table and the recycle bin is enabled, Oracle does **not**
immediately destroy the table segment.

Instead:

- the object is renamed to a system-generated recycle-bin name
- dependent objects may also be renamed and placed in the recycle bin
- the space remains allocated until the object is purged or Oracle reclaims it

This is Flashback Drop.

It is **not**:

- undo-based Flashback Query
- Flashback Table to timestamp/SCN
- Flashback Archive history

This is just Oracle saying:

- "I did not really throw it away yet, in case you regret yourself quickly."

---

## 2. The Recycle Bin Parameter and Security Reality

The source says the recycle bin may or may not be turned on and raises security
concerns.

That is fair.

In `19c`, the `RECYCLEBIN` parameter:

- defaults to `ON`
- is modifiable at the session level
- is modifiable with deferred system scope
- is modifiable in a `PDB`

That means you can do:

```sql
ALTER SESSION SET recyclebin = OFF;
```

or:

```sql
ALTER SYSTEM SET recyclebin = OFF;
```

Security concern:

- dropped objects can remain recoverable for a while
- sensitive data may therefore continue to exist on disk and count against quota

Operational correction:

- disabling the recycle bin does **not** purge objects already in it

So if the concern is real, the job is not just:

- "turn it off"

It is:

- "turn it off and purge what is already there if policy requires it"

---

## 3. What You Actually Query

The source says objects go into the "DBA recycle bin."
That is not the clean way to say it.

Useful views and synonyms are:

- `RECYCLEBIN` synonym
- `USER_RECYCLEBIN`
- `DBA_RECYCLEBIN`

Example:

```sql
SELECT object_name,
       original_name,
       type,
       droptime
FROM user_recyclebin;
```

The system-generated names look like:

- `BIN$...==$0`

The good news is you usually do not need to memorize the gibberish, because the
views also show:

- `ORIGINAL_NAME`

which is Oracle's merciful concession to the fact that no human being wants to
restore a table called `BIN$jsleilx392mk2=293$0` on purpose.

---

## 4. Flashback Drop: Bring the Table Back

To restore a dropped table:

```sql
FLASHBACK TABLE hr.employees TO BEFORE DROP;
```

Or restore and rename in one move:

```sql
FLASHBACK TABLE hr.employees TO BEFORE DROP RENAME TO employees_recovered;
```

This is fast because Oracle is not reconstructing the table from backup.
It is restoring the dropped object from the recycle bin.

Important Oracle nuance:

- indexes and constraints can come back with recycle-bin names
- materialized view logs are dropped but are **not** placed in the recycle bin
- if space pressure caused Oracle to purge dependent indexes, then not all
  indexes may return with the table

So Flashback Drop is convenient.
It is not a legally binding promise that every dependent object will return with
perfect dignity and original naming.

---

## 5. How to Bypass or Purge the Recycle Bin

If you want a drop to be final, use:

```sql
DROP TABLE hr.employees PURGE;
```

That bypasses the recycle bin entirely.

And once purged:

- the object cannot be recovered with Flashback Drop

That is not Oracle being moody.
That is the whole point of `PURGE`.

You can also purge after the fact:

```sql
PURGE TABLE hr.employees;
PURGE RECYCLEBIN;
PURGE DBA_RECYCLEBIN;
```

And yes, Oracle is explicit:

- you cannot roll back a `PURGE`
- you cannot recover an object after it is purged

Additional correction to the lecture:

- dropping a tablespace, especially `INCLUDING CONTENTS`, does not send its
  objects politely into the recycle bin for future nostalgia
- dropping a user purges that user's recycle-bin objects as part of the process

So "drop tablespace including contents" and "drop user cascade" are not
recycle-bin preservation strategies.
They are cleanup operations with a flamethrower.

---

## 6. Flashback Archive: Longer-Term History Than Undo

The next topic in the source shifts away from dropped objects and into what it
calls data archive, temporal history, and Flashback Data Archive.

The modern Oracle terminology here is:

- Flashback Archive
- Flashback Time Travel

This feature stores row history in:

- internal history tables
- inside one or more Flashback Archive tablespaces

That history is then queryable using:

- `AS OF`
- `VERSIONS BETWEEN`

But now the source can be:

- Flashback Archive history

instead of:

- ordinary undo

This matters because Flashback Archive is for longer-lived historical access.

Undo says:

- "I can remember for a while."

Flashback Archive says:

- "I have retention policy and storage planning, because your auditors never go away."

---

## 7. Creating a Flashback Archive

You create a Flashback Archive with:

```sql
CREATE FLASHBACK ARCHIVE fda1
  TABLESPACE fda_tbs1
  QUOTA 10M
  RETENTION 1 YEAR;
```

Or make it the default:

```sql
CREATE FLASHBACK ARCHIVE DEFAULT fda1
  TABLESPACE fda_tbs1
  QUOTA 10M
  RETENTION 1 YEAR;
```

Important Oracle `19c` details:

- if quota fills up, DML on tracked tables can fail
- old history is purged after the retention period
- `OPTIMIZE DATA` is available, but requires the Advanced Compression option

Example:

```sql
CREATE FLASHBACK ARCHIVE fda1
  TABLESPACE fda_tbs1
  QUOTA 10M
  RETENTION 1 YEAR
  OPTIMIZE DATA;
```

And yes, that "optimize" clause is one of those Oracle features that quietly
means:

- "this is lovely, provided your licensing people remain calm."

---

## 8. Enabling Flashback Archive on a Table

Once the archive exists, enable it for a table:

```sql
ALTER TABLE hr.employees FLASHBACK ARCHIVE fda1;
```

Or use the default archive:

```sql
ALTER TABLE hr.employees FLASHBACK ARCHIVE;
```

Important prerequisites from Oracle docs:

- you need the `FLASHBACK ARCHIVE` object privilege on that archive
- the table cannot be nested, temporary, remote, or external
- the table cannot contain `LONG` or nested columns

Also useful:

- Flashback Archive is disabled for tables by default

So if the source talks as though your table will naturally start writing to a
temporal history table the moment you become emotionally attached to history,
no, Oracle requires explicit enablement.

---

## 9. How Flashback Archive History Is Stored

The source describes changes being copied from the base table into the history
store after commit.

That is the right mental model at a high level.

Operationally, Oracle maintains:

- base table with current rows
- internal history tables for older versions

This gives you benefits such as:

- longer retention than ordinary undo
- support for many historical queries without depending on the undo window
- partitioning and storage management in the history structures

Oracle also supports some common DDL on Flashback Archive-enabled tables, which
is one of the reasons this feature is practical instead of just academically
interesting.

But not all DDL is supported.
Unsupported cases can raise:

- `ORA-55610`

which is Oracle's polite way of saying:

- "You have exceeded the set of schema changes I agreed to track historically."

---

## 10. User Context Tracking in Flashback Archive

The source's "typical" context discussion is real and useful.

Oracle lets you choose how much user context should be captured by using:

```sql
BEGIN
  DBMS_FLASHBACK_ARCHIVE.SET_CONTEXT_LEVEL('TYPICAL');
END;
/
```

Possible levels include:

- `NONE`
- `TYPICAL`
- `ALL`

In `19c`, `TYPICAL` captures:

- user ID
- global user ID
- hostname

The source lists more items under typical than the package reference does, so
this is one of those places where Oracle's own package docs win the argument.

To retrieve stored context for a versioned row, use:

```sql
SELECT versions_xid,
       DBMS_FLASHBACK_ARCHIVE.GET_SYS_CONTEXT(
         versions_xid,
         'USERENV',
         'SESSION_USER'
       ) AS session_user,
       versions_starttime,
       versions_endtime,
       employee_id,
       salary
FROM hr.employees
VERSIONS BETWEEN SCN 3816000 AND 3817000;
```

This is useful when row history alone is not enough and you want:

- who changed it
- when they changed it
- what version window applied

Which is very close to what people often try to force out of auditing after the
fact, except here Oracle can give you the historical row versions too.

---

## 11. Disassociating and Reassociating Flashback Archive

The source mentions temporarily separating the base table from its history so
you can do unsupported schema changes.

That is a real feature.

Oracle provides package procedures for:

- disassociating the table from Flashback Archive
- making your unsupported schema changes
- reassociating the table later

This is operationally important because it means:

- not all DDL is blocked forever
- but history capture is interrupted during the disassociated window

And that means exactly what you think it means:

- changes made during that window are not protected in the same continuous way

So if you disassociate and later reassociate, do not tell auditors the history
remained magically immaculate.
It did not.

---

## 12. Temporal Validity: Business Time, Not Transaction Time

Now we get to the part the lecture really wanted to teach but kept stirring into
the same pot as Flashback Archive.

Temporal validity is about:

- the **business-valid period** of a row

Examples:

- employee benefits valid from one date to another
- contract terms valid during a specific period
- pricing valid only within an approved effective window

This is not the same as transaction-time history.

Transaction time asks:

- "When did the database record or change this row?"

Valid time asks:

- "For what real-world period is this row considered true?"

Those are different questions, and confusing them is how data models end up
needing PowerPoint apologetics.

---

## 13. Defining a Valid-Time Period

You can define temporal validity with a `PERIOD FOR` clause.

Explicit columns:

```sql
CREATE TABLE hr.emp_temporal (
  employee_id        NUMBER PRIMARY KEY,
  salary             NUMBER,
  user_time_start    TIMESTAMP,
  user_time_end      TIMESTAMP,
  PERIOD FOR user_time (user_time_start, user_time_end)
);
```

Or let Oracle generate hidden period columns:

```sql
CREATE TABLE hr.emp_temporal (
  employee_id NUMBER PRIMARY KEY,
  salary      NUMBER,
  PERIOD FOR user_time
);
```

That gives you valid-time semantics at the table design level.

So when the source says Oracle can define hidden start and end columns for you,
that is basically the idea, just expressed with less drama and fewer verbal cartwheels.

---

## 14. Querying by Valid Time

Once temporal validity is defined, you can query by period or use session-level
visibility controls.

The source conceptually shows restricting what a user can see to:

- current valid data
- data valid as of a chosen point
- data valid within a chosen interval

Oracle supports session-level valid-time control with:

```sql
BEGIN
  DBMS_FLASHBACK_ARCHIVE.ENABLE_AT_VALID_TIME('CURRENT');
END;
/
```

Or:

```sql
BEGIN
  DBMS_FLASHBACK_ARCHIVE.ENABLE_AT_VALID_TIME(
    'ASOF',
    TO_TIMESTAMP('2012-12-31 00:00:00','YYYY-MM-DD HH24:MI:SS')
  );
END;
/
```

Or restore full visibility:

```sql
BEGIN
  DBMS_FLASHBACK_ARCHIVE.ENABLE_AT_VALID_TIME('ALL');
END;
/
```

This is session-level visibility filtering.

That means you are not changing the data.
You are changing what the session is allowed to see through the valid-time lens.

Which is useful, but also exactly the sort of thing people forget is active and
then spend an hour wondering why their rows have "disappeared."

---

## 15. What This Lesson Actually Separates

This topic contains three related but different ideas:

### Flashback Drop

- recover dropped tables from the recycle bin

### Flashback Archive / Time Travel

- keep long-term transaction-time row history in archive tablespaces

### Temporal Validity

- filter and model data according to real-world validity periods

They overlap in the sense that all three are about "past" or "time."
They do **not** overlap in storage model or purpose.

That is the main thing the source needed help saying clearly.

---

## 16. Practical Takeaways

- Flashback Drop depends on the recycle bin, not undo and not Flashback Archive.
- The recycle bin defaults to `ON` in `19c`, but disabling it does not purge
  objects already stored there.
- `DROP ... PURGE` means final destruction, not delayed regret.
- Flashback Archive stores long-term row history in internal history tables and
  supports historical queries beyond the normal undo window.
- `OPTIMIZE DATA` on Flashback Archive is real and useful, but it is also a
  licensing conversation.
- User context tracking in Flashback Archive can make historical row analysis
  much more useful.
- Temporal validity is about business-valid time, not transaction-time history.
- Session-level valid-time filtering changes visibility, not the stored data.

Oracle gives you several ways to ask about the past.
The trick is remembering which past you mean:

- dropped-object past
- transaction-history past
- business-valid-time past

Because those are three different rabbit holes, and Oracle absolutely expects
you to bring the correct flashlight.

---

## 17. Demo and Practice - Recovering a Dropped Table from the Recycle Bin

The source demo and Practice 2.2 are built around a very realistic developer
request:

- "I dropped the table"
- "not that dropped copy"
- "the other dropped copy"
- "the one with the right column set"

Which is exactly the kind of clear, stable requirement definition Oracle DBAs
have always enjoyed.

This is a good lab because it forces you to do more than blindly issue:

```sql
FLASHBACK TABLE ... TO BEFORE DROP;
```

You first have to identify **which** recycled object is the right one.

---

## 18. Seeing the Right Recycle Bin

The lab correctly points out that:

```sql
SHOW RECYCLEBIN;
```

only shows recycle-bin objects for the **current user**.

So if you are connected as `SYS`, that command is useless for finding a dropped
developer table unless `SYS` was somehow the person doing development, in which
case you have a different problem.

The correct view for the DBA perspective is:

```sql
SELECT original_name,
       object_name,
       droptime
FROM dba_recyclebin
WHERE owner = 'BAR';
```

That is the real first step in the recovery:

- find the candidate dropped objects
- see their original names
- compare their drop times

The source uses this correctly once it stops trying to `SHOW` its way through a
dictionary problem.

---

## 19. Identifying the Correct Dropped Copy

The lab's developer needs the version of `BAR102` that contains:

- `LOCATION_ID`

And there are multiple dropped versions with the same original name.

This is why the lab tests each candidate directly by querying the recycle-bin
object name in quotes:

```sql
SELECT location_id
FROM bar."BIN$..."
WHERE ROWNUM = 1;
```

That is ugly, but correct.

Because recycle-bin object names contain special characters and mixed symbols,
they must be quoted.

The logic is sound:

1. list dropped copies
2. test each candidate
3. confirm which one has the required structure
4. restore that exact one

This is much better than recovering the first matching original name and then
discovering the developer really wanted the second-most-recent disaster.

---

## 20. Flashing Back the Specific Dropped Object

Once the correct recycle-bin object is identified, the lab restores it with:

```sql
FLASHBACK TABLE bar."BIN$..."
TO BEFORE DROP
RENAME TO bar102a;
```

That is a strong recovery pattern because:

- it recovers the exact dropped instance you chose
- it avoids colliding with the existing `BAR102` table
- it gives the developer a side-by-side comparison target

This is a small but very sensible choice.

The source is right to restore it as `BAR102A` instead of blindly reclaiming the
old name. When there is already a live object with that original name, renaming
the recovered copy is the adult move.

And yes, you should verify afterward:

```sql
SELECT *
FROM bar.bar102a
WHERE ROWNUM = 1;
```

Because successful `FLASHBACK TABLE` output is nice, but the actual test is:

- does the recovered table have the expected columns and rows?

---

## 21. What Practice 2.2 Actually Teaches

Practice 2.2 is not just "recover a dropped table."
It teaches the more valuable DBA habit:

- identify the object precisely before restoring it

The practical flow is:

1. set up the lab and generate multiple dropped versions
2. query `DBA_RECYCLEBIN`
3. identify the owner and original object name
4. test the internal recycle-bin object names directly
5. recover the correct one with `TO BEFORE DROP`
6. rename it to avoid name conflicts
7. validate the data
8. clean up, including recycle-bin purge

That is exactly how you keep Flashback Drop from becoming a very efficient way
to restore the wrong thing with complete confidence.
