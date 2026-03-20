## Lesson 23 - Performing Block Media Recovery (in which Oracle lets you repair a few bad blocks without punishing the whole data file for their misconduct)

And look, block corruption is one of those wonderfully specific Oracle disasters: not "the database is gone," not "the disk exploded," just "these few blocks have chosen violence." Which is annoying, but also precisely the kind of problem RMAN block media recovery was built to handle.

By the end of this lesson, you should be able to:

- Explain the difference between physical and logical block corruption
- Recognize the usual corruption indicators such as `ORA-01578`
- Use validation and diagnostic views to identify corrupted blocks
- Understand the prerequisites for block media recovery
- Use RMAN to repair one block, several blocks, or all blocks listed as corrupt
- Place legacy Data Recovery Advisor examples in the correct `19c` context

---

## 1. What Block Corruption Actually Means

A database block is considered corrupted when Oracle reads or writes it and the
block contents fail integrity checks.

The source is broadly correct that Oracle checks things such as:

- block structure
- block address consistency
- block header information
- checksum information, if checksum protection is enabled

When those checks fail, the corruption may be classified as:

- **physical corruption**
  the block is malformed or unreadable at the storage level
- **logical corruption**
  the block is structurally readable, but the contents are internally wrong

This distinction matters because physical corruption is often a storage or media
problem, while logical corruption may indicate software issues, bad memory,
bad writes, or application-level damage that somehow made it onto disk.

---

## 2. The Error Users Usually See

The classic user-facing error is:

- `ORA-01578: ORACLE data block corrupted`

This error typically identifies:

- file number
- block number

That information is gold, because it tells you exactly which block Oracle is
unhappy about instead of making you guess which part of the database has gone
feral today.

The corruption is typically also recorded in:

- the alert log
- diagnostic trace information
- `V$DATABASE_BLOCK_CORRUPTION`

Important correction to the source:

- the dynamic performance view is `V$DATABASE_BLOCK_CORRUPTION`
- not `V$_DATABASE_BLOCK_CORRUPTION`

Oracle's naming is already sufficiently theatrical without inventing new views.

---

## 3. How You Detect Corruption

You do not have to wait for a user to discover corruption by stepping on it.

You can proactively check with RMAN validation commands such as:

```rman
VALIDATE DATABASE;
```

Or, if you also want logical checks:

```rman
VALIDATE CHECK LOGICAL DATABASE;
```

You can validate more narrowly as well:

- a specific data file
- a specific backup set
- a tablespace

Useful query:

```sql
SELECT * FROM v$database_block_corruption;
```

This view records blocks Oracle has identified as corrupt and not yet repaired.

The source is also right that `ANALYZE ... VALIDATE STRUCTURE` can reveal some
object-level issues, but for backup/recovery practice RMAN validation is the
more central tool.

---

## 4. Parameters That Help Detect Trouble

Two commonly discussed parameters are:

- `DB_BLOCK_CHECKING`
- `DB_BLOCK_CHECKSUM`

They help detect corruption earlier, but at a performance cost.

That trade-off is real:

- more protection
- more overhead

So the practical rule is:

- use them intentionally
- understand the workload cost
- do not switch them on casually and then act surprised when the database
  notices

The source's high-level guidance is fine here: these settings can be useful
during investigation, especially if you suspect memory or storage corruption.

---

## 5. What Block Media Recovery Is For

Block media recovery is the targeted repair mechanism for a **small number of
corrupted blocks** when the rest of the file is still usable.

Its advantages are exactly why people like it:

- the data file can stay online
- the tablespace can stay online
- only the corrupted blocks are inaccessible during repair
- recovery time is much smaller than restoring a whole file

This is why block media recovery is a gift from Oracle to DBAs who would prefer
not to restore a multi-gigabyte file because two blocks decided to become
abstract art.

---

## 6. Prerequisites for Block Media Recovery

The important prerequisites are:

- database in `ARCHIVELOG` mode
- target database open or mounted with a current control file
- suitable backups available
- archived redo needed to bring the repaired blocks current

Oracle can use one of the following as the source for the good block image:

- full backup
- level `0` incremental backup
- flashback logs, when suitable and available

The source is directionally correct here.
Block media recovery is not something RMAN can improvise from pure optimism.
It needs a known good copy of the block and redo to roll it forward.

---

## 7. Correct RMAN Syntax

This is where the source gets sloppy.

It says things like "from SQL you can recover data file and block."
That is not the syntax you want to learn.

The RMAN command is:

```rman
RECOVER DATAFILE 20 BLOCK 129;
```

For multiple blocks:

```rman
RECOVER
  DATAFILE 20 BLOCK 129
  DATAFILE 20 BLOCK 130
  DATAFILE 21 BLOCK 42;
```

And if you want RMAN to repair every block currently listed as corrupt:

```rman
RECOVER CORRUPTION LIST;
```

Important correction:

- this is an **RMAN** command
- not a `SQL*Plus` command
- block media recovery cannot be used for the data file header block (`block 1`)

So if the source says "from SQL issue the recover data file block command," what
it really means is "connect to RMAN and use the correct syntax like a grown-up."

---

## 8. Where RMAN Gets the Good Block

When you run block recovery, RMAN finds a usable copy of the block from backup
or flashback and then applies redo so the block becomes current again.

You can also restrict how RMAN searches, for example by tag, but the basic idea
is:

1. find good copy
2. restore the bad block from that copy
3. apply redo
4. return the block to civilized society

That is why the source correctly says you do not normally lose committed data if
the required redo exists.

---

## 9. `V$DATABASE_BLOCK_CORRUPTION` Is Your Ledger

After corruption is discovered, Oracle records it in:

```sql
SELECT * FROM v$database_block_corruption;
```

This view is useful for:

- identifying which files and blocks are affected
- confirming whether corruption remains after repair

If you use:

```rman
RECOVER CORRUPTION LIST;
```

RMAN reads directly from this view.

And after successful repair, Oracle removes the repaired blocks from the view.

Which is nice, because nothing says "confidence" like a corruption list that
actually becomes shorter after you fix something.

---

## 10. Best Practices Before You Recover

Before repairing blocks, do at least some adult supervision:

- check the alert log
- check `V$DATABASE_BLOCK_CORRUPTION`
- run `VALIDATE` if you suspect broader damage
- determine whether the issue is isolated or systemic

Because if repeated corruption appears on the same device, then your real
problem may be:

- failing disk
- failing controller
- bad storage path
- bad memory

In other words, if you repair the block but ignore the hardware fault, you are
not solving the problem.
You are just resetting the countdown timer.

---

## 11. A Useful 19c Correction: DRA Is Not the Main Story

The source keeps wrapping block recovery in Data Recovery Advisor:

- `LIST FAILURE`
- `ADVISE FAILURE`
- generated repair script
- `REPAIR FAILURE`

That is historically understandable.
It is also not the best mental model for `19c`.

Data Recovery Advisor is deprecated in `19c`.

Oracle's `19c` documentation goes further and states that administrators no
longer have access to the classic DRA RMAN commands.

So the right modern framing is:

- DRA can appear in old labs and older systems
- the core skill you actually want is plain RMAN block recovery

If you know the file and block numbers, RMAN already gives you the direct tool:

```rman
RECOVER DATAFILE 20 BLOCK 129;
```

No theatrical middleman required.

---

## 12. Another Correction: This Is Not "Data Guard Advisor"

The demo title says:

- "using the Data Guard Advisor as well as RMAN"

That is almost certainly a mistake in the source narration.

This lesson is about:

- RMAN block media recovery
- optionally wrapped in the old Data Recovery Advisor workflow

There is, however, a real Data Guard angle worth knowing:

- with a properly configured physical standby, Oracle can sometimes perform
  automatic block repair between primary and standby

That is a real feature.
It is not what the narrated demo is actually showing.

---

## 13. Demo Flow: Recovering Corrupt Blocks

The demo's useful sequence is:

1. user encounters corruption error
2. DBA identifies file and block number
3. RMAN confirms the failure
4. a repair script is shown
5. RMAN repairs the corrupted blocks
6. failures are checked again

The example repair script is conceptually just this:

```rman
RECOVER DATAFILE 20 BLOCK 129 DATAFILE 20 BLOCK 130;
```

That is the real takeaway.

The generated script is not magical.
It is just RMAN writing down the command you would have typed yourself if Oracle
had trusted you from the beginning.

---

## 14. Practice 5.1 - Repairing Block Corruption

The practice is a good compact lab because it deliberately creates one
recoverable corruption event and walks through the full cycle.

### What the setup does

The lab script creates:

- user `BC`
- tablespace `BCTBS`
- table `BCCOPY`

It then:

- populates the table
- backs it up
- updates data so there is meaningful redo to apply during recovery

That is exactly what a useful block recovery lab needs:

- a good copy of the block in backup
- subsequent redo to make recovery nontrivial

### What the break step does

The corruption script damages a specific block and then forces a query to hit
that block.

That produces the expected corruption error and tells you the affected block
number.

In the sample narration, that block was:

- block `129`

Your environment may differ.
Use the block number your lab actually reports.

Because Oracle is many things, but "same file and block numbers on everyone's
machine forever" is not one of them.

---

## 15. Practice Recovery Flow

The lab then goes through the old DRA path:

```rman
LIST FAILURE;
ADVISE FAILURE;
REPAIR FAILURE;
```

That is acceptable for understanding old material.

But the practical modern interpretation is:

- corruption has been identified
- RMAN knows which blocks are bad
- the repair is a block media recovery operation

If doing this directly, the equivalent repair concept is:

```rman
RECOVER DATAFILE 20 BLOCK 129;
```

Or, if the corruption view has the entries already logged:

```rman
RECOVER CORRUPTION LIST;
```

After the repair, the correct follow-up checks are:

- `LIST FAILURE` if you are still following the old DRA flow
- `SELECT * FROM V$DATABASE_BLOCK_CORRUPTION;`
- application query or full table scan to verify the object is readable again

The practice's verification query:

```sql
SELECT * FROM bc.bccopy;
```

is exactly the right kind of final check.

Do not trust recovery merely because RMAN said nice things.
Trust it because the damaged object can actually be read.

---

## 16. Practical Takeaways

- Block corruption is often isolated enough that block media recovery is the
  right answer.
- Learn the real RMAN syntax: `RECOVER DATAFILE ... BLOCK ...`
- Use `VALIDATE` and `V$DATABASE_BLOCK_CORRUPTION` to discover and confirm
  corruption.
- Treat DRA examples as legacy context in `19c`, not as the main skill.
- If corruption repeats, investigate hardware and storage, not just Oracle.
- After repair, verify the object at the SQL level, not just in RMAN output.

Block media recovery is one of Oracle's more humane recovery features.
It lets you fix the actual damaged part instead of bulldozing the entire file
because two blocks had a nervous breakdown.
