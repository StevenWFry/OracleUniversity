## Lesson 26 - Using Logical Flashback Features (in which Oracle lets you reverse bad decisions without restoring backups, provided undo still remembers and the feature you want is actually supported)

And look, this is the part of Oracle recovery everyone wants to believe in: no tape, no auxiliary instance, no giant restore, just a clean rewind of the mistake. Which is wonderful right up until you discover that each logical flashback feature has different prerequisites, different scope, and different ways to refuse service.

By the end of this lesson, you should be able to:

- Use Flashback Query, Version Query, and Transaction Query to investigate bad changes
- Understand when Flashback Table is the right recovery tool
- Explain when Flashback Transaction Backout is useful and when it is not
- Distinguish transaction-time flashback from temporal validity and temporal history
- Plan undo, privileges, and supplemental logging so the feature you want still works when you need it

---

## 1. What This Lesson Covers

This lesson is about the **logical** Flashback features:

- Flashback Query
- Flashback Version Query
- Flashback Transaction Query
- Flashback Table
- Flashback Transaction Backout
- temporal validity support
- temporal history through Flashback Archive / Flashback Time Travel

These are the tools you use when:

- the database is physically fine
- the data is logically wrong

So this is not media recovery.
This is "the system kept working, and that was unfortunately the problem."

---

## 2. Flashback Query: Read the Past Without Changing the Present

Flashback Query uses the `AS OF` clause to show committed data as it existed at
an earlier `SCN` or timestamp.

Examples:

```sql
SELECT *
FROM hr.employees AS OF SCN 3817001
WHERE employee_id = 200;
```

```sql
SELECT *
FROM hr.employees AS OF TIMESTAMP
     TO_TIMESTAMP('2026-03-20 11:00:00','YYYY-MM-DD HH24:MI:SS')
WHERE employee_id = 200;
```

This is investigation only.
No data is changed.

It is useful when:

- you want to see what a row looked like before the mistake
- you need a point of comparison
- you are trying to choose a safe target for a later rewind

Important Oracle `19c` guidance:

- if precision matters, prefer `SCN` over timestamp
- timestamp mapping can be off by as much as about 3 seconds

That sounds small until the wrong transaction commits in those 3 seconds and
you get to explain why "close enough" was not, in fact, close enough.

---

## 3. Flashback Version Query: See Every Committed Version

Version Query shows all row versions that existed during a specified interval.

Example:

```sql
SELECT versions_startscn,
       versions_endscn,
       versions_operation,
       versions_xid,
       employee_id,
       salary
FROM hr.employees
VERSIONS BETWEEN SCN 3816000 AND 3817000
WHERE employee_id = 200;
```

You can also use timestamps:

```sql
SELECT versions_starttime,
       versions_endtime,
       versions_operation,
       versions_xid,
       employee_id,
       salary
FROM hr.employees
VERSIONS BETWEEN TIMESTAMP
       TO_TIMESTAMP('2026-03-20 10:00:00','YYYY-MM-DD HH24:MI:SS')
   AND TO_TIMESTAMP('2026-03-20 11:00:00','YYYY-MM-DD HH24:MI:SS')
WHERE employee_id = 200;
```

This is useful when:

- one row was updated multiple times
- you need to see the whole sequence of committed versions
- you want the transaction ID (`XID`) for later analysis

So Flashback Query tells you:

- "what did it look like at one point?"

Version Query tells you:

- "what are all the committed versions in this interval?"

That is a much more useful question when users say:

- "something changed sometime this morning"

which, as always, is not a timestamp.

---

## 4. Flashback Transaction Query: See What the Transaction Did

Flashback Transaction Query reads from:

- `FLASHBACK_TRANSACTION_QUERY`

Example:

```sql
SELECT xid,
       operation,
       logon_user,
       table_name,
       commit_scn,
       undo_sql
FROM flashback_transaction_query
WHERE xid = HEXTORAW('000200030000002D');
```

This is where you inspect:

- which transaction changed the data
- what operations it performed
- what `UNDO_SQL` Oracle can generate to reverse it logically

It fits naturally with Version Query:

1. use Version Query to get `VERSIONS_XID`
2. use that `XID` in `FLASHBACK_TRANSACTION_QUERY`
3. inspect the transaction details and undo SQL

Important prerequisites and caveats:

- supplemental logging should be enabled
- this is about committed transaction history, not uncommitted work

Important `19c` multitenant caveat:

- Oracle documents Flashback Transaction Query as **not supported in a CDB**

So if the source makes this sound universally available in multitenant, that is
lecture optimism, not documented `19c` behavior.

---

## 5. Restrictions on Flashback Query and Version Query

The source is directionally right that you cannot just point Flashback clauses
at every kind of object in the database.

Oracle explicitly documents limitations such as:

- no past data from dynamic performance (`V$`) views
- structural DDL can break your ability to query older states cleanly
- Flashback features depend on the relevant undo or historical data still being
  available

The source also mentions temporary, external, and fixed tables in a broad way.
The clean operational rule is:

- use these features against ordinary application tables
- expect system, temporary, and special object classes to have restrictions

This is not the moment to experiment on `SYS` objects and then act surprised
when Oracle reacts like you just licked the wiring.

---

## 6. Flashback Table: Rewind the Table In Place

Flashback Table is the feature that actually changes the table back to an
earlier state.

Example:

```sql
FLASHBACK TABLE hr.employees
TO SCN 3817001;
```

Or:

```sql
FLASHBACK TABLE hr.employees
TO TIMESTAMP TO_TIMESTAMP('2026-03-20 10:55:00','YYYY-MM-DD HH24:MI:SS');
```

Important prerequisites:

- `FLASHBACK` object privilege or `FLASHBACK ANY TABLE`
- `SELECT` or `READ`
- `INSERT`
- `DELETE`
- `ALTER`
- row movement enabled

Enable row movement first:

```sql
ALTER TABLE hr.employees ENABLE ROW MOVEMENT;
```

Why?

- because Flashback Table does not preserve rowids

If you forget, Oracle gives you:

- `ORA-08189: cannot flashback the table because row movement is not enabled`

And that is Oracle being unusually direct for once.

---

## 7. Flashback Table: What It Does and Does Not Preserve

This is where the source is mostly right, but Oracle's wording is sharper.

### What Flashback Table does well

- rewinds table data in place
- keeps the database online
- preserves currently existing indexes by adjusting them to match the flashed
  back table state
- preserves currently existing dependent objects better than a brute-force
  export/import stunt would

### What it does not do

- it does **not** rewind table statistics to their older state
- it does **not** preserve rowids
- it does **not** cross structural DDL boundaries cleanly

So after Flashback Table, Oracle recommends:

- gather statistics again

Because otherwise the optimizer is working with statistics from the wrong
timeline, which is one of those details that turns a successful recovery into a
fresh performance problem.

Example:

```sql
EXEC DBMS_STATS.GATHER_TABLE_STATS('HR','EMPLOYEES');
```

---

## 8. Flashback Table Restrictions That Actually Matter

Oracle documents several important restrictions.

You cannot use ordinary `TO SCN` or `TO TIMESTAMP` Flashback Table across
structural changes such as:

- truncating a table
- moving a table
- adding or dropping columns
- modifying column definitions
- many partition maintenance operations

The source says you "cannot span DDL operations."
That is the right instinct.
The real rule is:

- if the table structure changed, Flashback Table to a time before that change
  may fail

Also, certain object classes are excluded, including examples such as:

- clustered tables
- materialized views
- AQ tables
- system tables
- remote tables

So yes, use this on ordinary application tables.
Not on Oracle's internal plumbing unless you have developed a powerful desire to
read error messages all afternoon.

---

## 9. Flashback Transaction Backout: Reverse the Transaction, Not the Whole Table

Flashback Transaction Backout uses:

- `DBMS_FLASHBACK.TRANSACTION_BACKOUT`

This creates **compensating transactions** that logically reverse the committed
transaction you want to undo.

That is different from Flashback Table:

- Flashback Table rewinds the whole table to a prior point
- Transaction Backout targets one transaction and optionally its dependencies

This is ideal when:

- one bad transaction should be reversed
- the rest of the table should remain where it is

This is less ideal when:

- the dependency tree resembles a plate of dropped spaghetti

The database can analyze dependencies and back out:

- just the transaction
- the transaction plus dependent transactions

Common options include:

- `NOCASCADE`
- `CASCADE`
- `NOCASCADE_FORCE`
- `NONCONFLICT_ONLY`

These are not cosmetic.
They define how aggressively Oracle should unwind transactional relationships.

---

## 10. Transaction Backout Prerequisites and Reports

The source correctly emphasizes supplemental logging and dependency analysis.

To use Transaction Backout safely, you want:

- sufficient undo and history still available
- supplemental logging enabled
- appropriate privileges

And then you inspect the reports from:

- `DBA_FLASHBACK_TXN_STATE`
- `DBA_FLASHBACK_TXN_REPORT`

These views tell you:

- what Oracle backed out
- what dependencies were involved
- what state the transaction now has

Very important practical detail:

- `TRANSACTION_BACKOUT` does **not** commit automatically

So after you inspect the result, you decide:

- `COMMIT` if the backout is correct
- `ROLLBACK` if Oracle reversed something you did not want reversed

That is one of those delightful enterprise features where Oracle says:

- "Here is the loaded scalpel. I trust you to make the last decision."

---

## 11. Important `19c` CDB Caveat for Transaction Backout

The source talks about Flashback Transaction Backout as though it is just
available whenever you want it.

Oracle `19c` documentation is less enthusiastic.

Oracle documents Flashback Transaction Backout as:

- **not supported in a CDB**

So in modern multitenant environments, this is a big caveat, not a footnote.

That means for a `19c` `CDB`:

- study the concept
- recognize it in older course material
- do not assume it is a supported everyday answer in your multitenant
  production environment

That is exactly the kind of Oracle feature gap that people discover only after
they have already promised management that the reversal will be easy.

---

## 12. Temporal Validity: Filter Data by Business Valid Time

The source introduces "temporal validity" and "temporal history," which are
related to flashback but are not the same thing as ordinary undo-based rewind.

Temporal validity support is about:

- the time period in the real world during which data is valid

Examples:

- employee contract valid from one date to another
- insurance policy coverage period
- customer address valid only for a specified range

Oracle supports valid-time dimensions and session-level valid-time filtering.

This is typically used so that queries show only:

- data valid now
- data valid at a chosen time
- data valid during a chosen interval

In newer Oracle terminology, this is often discussed alongside:

- Flashback Time Travel
- Flashback Archive

because both deal with historical or time-aware views of data, but they solve
different problems:

- temporal validity = business validity of rows
- flashback query = transaction-time view from undo

Those are not the same timeline.

One is:

- "when was this row valid in the business?"

The other is:

- "what did the database row look like at a past committed point?"

That difference matters a lot.

---

## 13. Temporal History / Flashback Archive / Flashback Time Travel

If you need longer-term historical querying than undo can provide, then you are
really in Flashback Archive territory.

Oracle's current terminology here is:

- Flashback Archive
- Flashback Time Travel

This stores historical versions in internal history tables rather than relying
only on undo retention.

That is what you want for:

- compliance
- auditing
- history over long periods
- avoiding `snapshot too old` limits for historical queries

So when the source says "temporal history," the cleaned-up interpretation is:

- long-term historical querying belongs with Flashback Archive / Time Travel,
  not ordinary undo-based Flashback Query alone

Undo is for regret with a short memory.
Flashback Archive is for regret with governance requirements.

---

## 14. Best Practices for Logical Flashback

The source's best-practice section is good in spirit.
Here is the cleaned-up version.

### Size and retain undo properly

- use Automatic Undo Management
- monitor undo usage
- use Undo Advisor
- make sure the undo window is long enough for the flashback features you want

### Prefer `SCN` when precision matters

- timestamps are convenient
- `SCN` is more exact

### Gather statistics after Flashback Table

- statistics are not flashed back
- optimizer plans should not be left to guesswork

### Know your DDL boundaries

- structural DDL can make older states inaccessible to Flashback Table
- transaction-oriented flashback features also become more complicated when the
  underlying object definitions changed

### Do investigation before recovery

- Flashback Query and Version Query are your reconnaissance tools
- do not jump straight into rewinding data because a user said "something looks
  weird"

### In `19c` multitenant, do not overpromise transaction backout

- CDB restrictions matter
- support boundaries matter

Because nothing ruins a recovery meeting like discovering the feature is
beautiful in theory and unsupported in your actual architecture.

---

## 15. Practical Takeaways

- Flashback Query and Version Query are investigative tools. They do not change
  data.
- Flashback Transaction Query helps you identify the exact transaction and its
  undo SQL.
- Flashback Table is the fast in-place rewind for a table, but it needs row
  movement and enough undo.
- Flashback Table does not restore old statistics, so gather them afterward.
- Flashback Transaction Backout is powerful, but dependency-heavy and not
  supported in `19c` CDBs.
- Temporal validity is about business-valid time, not just undo-based rewind.
- Flashback Archive / Time Travel is the long-memory answer when undo is not
  enough.

Logical flashback features are excellent tools.
They just require one important adult skill:

- choosing the smallest correct rewind mechanism instead of grabbing the biggest
  time machine on the lot.

---

## 16. Practice 2.3 - Using Flashback Table

The third lab is the actual hands-on rewind exercise for Flashback Table.

The setup creates:

- `BAR.BARCOPY`
- `BAR.BARDEPT`

with a foreign-key relationship between them.

Then the break script scrambles the departmental data and employee assignments.
The business requirement is:

- restore both tables to the state they had at time `T1`

This is a good lab because it forces you to rewind:

- the correct objects
- to a known timestamp
- while respecting referential relationships

Which is exactly where Flashback Table stops being a toy and starts being a DBA
tool.

---

## 17. Capture the Timestamp Before You Break Things

The lab first records the current timestamp:

```sql
SELECT TO_CHAR(
         SYSDATE,
         'YYYY-MM-DD:HH24:MI:SS'
       )
FROM dual;
```

That value becomes `T1`, the moment to which the tables must be restored.

This is the same principle you saw in point-in-time recovery labs:

- capture the target before the damage

Because "some time before the break script ran" is not a recovery target.
It is just a future meeting where everyone gets irritated.

---

## 18. Enable Row Movement First

Before flashing back either table, the lab does:

```sql
ALTER TABLE bar.bardept ENABLE ROW MOVEMENT;
ALTER TABLE bar.barcopy ENABLE ROW MOVEMENT;
```

That is mandatory for ordinary Flashback Table-to-time operations because:

- rowids may change

If you skip this, Oracle gives you `ORA-08189`, which is one of the rare cases
where the product explains the actual problem instead of handing you a riddle.

So yes, this step is necessary.
No, it is not optional syntax theater.

---

## 19. Flashing Back Related Tables

The transcript flashes back the two tables separately:

```sql
FLASHBACK TABLE bar.bardept
TO TIMESTAMP TO_TIMESTAMP('2026-03-20:10:15:00','YYYY-MM-DD:HH24:MI:SS');

FLASHBACK TABLE bar.barcopy
TO TIMESTAMP TO_TIMESTAMP('2026-03-20:10:15:00','YYYY-MM-DD:HH24:MI:SS');
```

That works in a controlled lab if both succeed cleanly.

But with referential relationships, the tidier production habit is usually to
flash back the related tables in **one statement**:

```sql
FLASHBACK TABLE bar.bardept, bar.barcopy
TO TIMESTAMP TO_TIMESTAMP('2026-03-20:10:15:00','YYYY-MM-DD:HH24:MI:SS');
```

Why this is cleaner:

- one target
- one atomic operation
- less chance of leaving related tables at mismatched points if something fails

So the source's two-step method is acceptable for the lab, but the single
statement is the better mental model for related objects.

---

## 20. Verifying the Result

The lab finishes by running a verification script and confirming the expected
row count.

That is the right thing to do.

After Flashback Table, success means:

- the tables are back at the intended logical state
- the relationship between them still makes sense
- the data seen by the application matches the expected pre-error state

And because this is Flashback Table, not just a query:

- you should gather statistics afterward if the tables matter to performance

The source mentioned that earlier, and it still applies here.

Example:

```sql
EXEC DBMS_STATS.GATHER_TABLE_STATS('BAR','BARDEPT');
EXEC DBMS_STATS.GATHER_TABLE_STATS('BAR','BARCOPY');
```

The lab itself may not bother.
Real systems should.

---

## 21. What Practice 2.3 Actually Teaches

This practice teaches the clean operational pattern for Flashback Table:

1. know the safe target time before the bad change
2. enable row movement on all affected tables
3. rewind the smallest set of related objects necessary
4. verify the business result, not just the SQL output
5. gather statistics afterward if the tables matter

And the deeper lesson is:

- Flashback Table is fantastic when the damage is confined and the undo is still
  available
- it is much less fantastic if you treat related tables one by one without
  thinking through their relationships

So yes, Flashback Table is fast and elegant.
But, like most elegant tools, it assumes you are paying attention.
