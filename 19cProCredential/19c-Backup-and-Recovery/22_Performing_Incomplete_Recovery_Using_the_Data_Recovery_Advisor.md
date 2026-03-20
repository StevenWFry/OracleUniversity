## Lesson 22 - Performing Incomplete Recovery Using the Data Recovery Advisor (in which Oracle demonstrates a legacy tool, then quietly forces you to learn point-in-time recovery anyway)

And look, the phrase "incomplete recovery" sounds like something has gone terribly wrong, and that is because something has gone terribly wrong. Specifically, you needed redo that no longer exists, so Oracle can no longer bring the database all the way forward to the present. At that point, the job stops being "fix the file" and becomes "choose which part of history you are willing to lose."

By the end of this lesson, you should be able to:

- Recognize when media recovery cannot continue because required archived redo is missing
- Understand why this becomes an incomplete recovery or point-in-time recovery problem
- See where the source demo uses legacy Data Recovery Advisor ideas that do not fit cleanly in `19c`
- Perform the core RMAN workflow for incomplete recovery safely
- Understand why `RESETLOGS` and a new incarnation are unavoidable after this operation

---

## 1. What This Demo Is Really About

The source presents a missing data file, begins with `LIST FAILURE` and
`ADVISE FAILURE`, and then eventually lands on a manual recovery path when
RMAN cannot complete media recovery because a required archived log is gone.

That is the real lesson.

The important sequence is:

1. a data file is missing or damaged
2. you restore the file from backup
3. you try to recover it
4. RMAN tells you a required archived redo log is missing
5. current-time recovery is no longer possible
6. you must recover to an earlier SCN, time, restore point, or log sequence

So despite the title, this is not mainly a Data Recovery Advisor lesson.
It is an **incomplete recovery** lesson triggered by **missing redo**.

And Oracle terminology matters here:

- **complete recovery** means recover as far forward as the available redo
  allows, ideally to the latest committed point
- **incomplete recovery** means stop at an earlier chosen point
- in practice, incomplete recovery is a **point-in-time recovery** problem

---

## 2. Important 19c Correction: Data Recovery Advisor Is Legacy Material

Oracle documents the Data Recovery Advisor (`DRA`) as **deprecated in 19c**.
This includes the RMAN commands:

- `LIST FAILURE`
- `ADVISE FAILURE`
- `REPAIR FAILURE`
- `CHANGE FAILURE`

So if the training material demonstrates these commands, treat that as legacy
knowledge, not as the center of a modern runbook.

There is a second correction that matters even more for this example:

- DRA is **not supported for individual PDBs**
- in a multitenant database, DRA works at the **root/CDB** level, not as a
  PDB-specific repair assistant

That means the demo's "problem inside one PDB, now let DRA walk me through it"
storyline is not really how you should model the recovery in a modern `19c`
environment.

The useful takeaway is this:

- if the issue is confined to one PDB, think in terms of **PDB PITR**
- if the whole CDB must be rolled back, think in terms of **whole-database
  incomplete recovery**
- do not expect DRA to be the adult supervision in either case

---

## 3. When a Data File Restore Stops Being Enough

Suppose you try the obvious repair first:

```rman
RESTORE DATAFILE 19;
RECOVER DATAFILE 19;
```

If RMAN can apply all required archived redo and online redo, then fine:

- the file is restored
- the file is recovered
- the database or PDB can continue

But if RMAN reports that a needed archived log is missing, then the file cannot
be brought up to the current SCN.

That is the moment the problem changes.

You are no longer deciding:

- "How do I restore this file?"

You are now deciding:

- "To which earlier point can I safely return the database or PDB?"

That is why this lesson naturally overlaps with the previous point-in-time
recovery lesson. The trigger is different, but the solution category is the
same.

---

## 4. Finding the Right Recovery Target

Before you run incomplete recovery, you must identify a target that is:

- earlier than the missing redo requirement
- late enough to preserve as much valid data as possible
- consistent with the scope of what you are recovering

Your possible targets include:

- `UNTIL TIME`
- `UNTIL SCN`
- `UNTIL SEQUENCE`
- `TO RESTORE POINT`

The source tries to discover this by checking:

- `V$DATABASE`
- `V$ARCHIVED_LOG`
- current open mode and log mode

That general idea is sound.

The syntax in the narration, however, is a mess in places. The useful views are:

```sql
SELECT name, dbid, current_scn, log_mode, open_mode
FROM v$database;
```

```sql
SELECT sequence#, first_change#, first_time, status
FROM v$archived_log
ORDER BY sequence#;
```

This lets you identify:

- which archived log sequence is missing
- what SCN range that sequence covers
- how far back you must stop recovery

Practical note:

- `UNTIL TIME` is often easiest for humans
- `UNTIL SCN` is often safest for precision
- `UNTIL SEQUENCE` works, but only if you are absolutely sure about the redo
  thread and sequence boundaries

And because Oracle likes precision more than optimism, the safest habit is to
set the recovery target **once** and then let both restore and recover obey it.

---

## 5. Preferred RMAN Pattern: Set the Target Once

The source demo says things such as:

```rman
RESTORE DATABASE UNTIL SEQUENCE 66;
RECOVER DATABASE UNTIL SEQUENCE 66;
```

That is not the clean pattern you want to learn.

In RMAN, the usual approach is:

```rman
RUN {
  SET UNTIL SEQUENCE 66 THREAD 1;
  RESTORE DATABASE;
  RECOVER DATABASE;
}
```

Or with an SCN:

```rman
RUN {
  SET UNTIL SCN 3812034;
  RESTORE DATABASE;
  RECOVER DATABASE;
}
```

Or with a time:

```rman
RUN {
  SET UNTIL TIME "TO_DATE('2026-03-20 10:15:00','YYYY-MM-DD HH24:MI:SS')";
  RESTORE DATABASE;
  RECOVER DATABASE;
}
```

This matters because you want one recovery boundary for the whole operation,
not a collection of separately improvised guesses.

---

## 6. Whole-Database Incomplete Recovery Workflow

If you truly need to roll back the whole database, the typical RMAN workflow is:

### If the control file is still usable

```rman
SHUTDOWN IMMEDIATE;
STARTUP MOUNT;

RUN {
  SET UNTIL SCN 3812034;
  RESTORE DATABASE;
  RECOVER DATABASE;
}

ALTER DATABASE OPEN RESETLOGS;
```

### If you must restore the control file first

```rman
STARTUP NOMOUNT;
RESTORE CONTROLFILE FROM AUTOBACKUP;
ALTER DATABASE MOUNT;

RUN {
  SET UNTIL SCN 3812034;
  RESTORE DATABASE;
  RECOVER DATABASE;
}

ALTER DATABASE OPEN RESETLOGS;
```

Why `RESETLOGS` is required:

- you did not recover to the current end of redo
- Oracle must start a new incarnation
- old online redo cannot simply continue as if nothing happened

That new incarnation is Oracle's way of saying:

- "We have chosen a different timeline. Do try to keep up."

---

## 7. Why the Source PDB Example Is Awkward

The demo starts with a missing data file in one `PDB`, but then drifts toward a
heavier whole-database recovery style.

That is a larger blast radius than you usually want.

In a multitenant database, ask this question first:

- is the damage really confined to one PDB?

If yes, then the more appropriate modern solution is usually:

- restore and recover the specific file if all redo exists, or
- perform **PDB point-in-time recovery** if required redo is missing

That keeps the rest of the `CDB` out of your little historical disaster.

For example:

```sql
ALTER PLUGGABLE DATABASE pdb1 CLOSE;
```

```rman
RUN {
  SET UNTIL SCN 3812034;
  RESTORE PLUGGABLE DATABASE pdb1;
  RECOVER PLUGGABLE DATABASE pdb1;
}
```

```sql
ALTER PLUGGABLE DATABASE pdb1 OPEN RESETLOGS;
```

That is much cleaner than treating one missing PDB data file as an excuse to
drag the entire CDB backward through time.

Important related correction:

- you cannot use DRA to diagnose and repair **individual PDBs**

So if the source appears to do that, the correct interpretation is:

- this is training material illustrating the concept
- the modern recovery skill you actually want is **manual RMAN PITR**

---

## 8. What DRA Can and Cannot Tell You in This Situation

Historically, DRA could still be useful in the early stages:

- `LIST FAILURE` showed the recognized failure
- `ADVISE FAILURE` showed repair options
- if all required backups and redo existed, `REPAIR FAILURE` might automate the
  restore/recover path

But if a required archived log is missing, DRA cannot perform magic.

At that point it may:

- suggest manual actions
- show that no automatic repair is available
- force you into a manual incomplete-recovery decision

So the source is directionally correct when it moves from:

- "tell me the failure"

to:

- "fine, I will do the incomplete recovery manually"

That transition is the real DBA lesson.

---

## 9. Verification After `RESETLOGS`

After incomplete recovery, do not just stare at the successful RMAN output and
declare victory like a reckless optimist.

Verify the result.

At the CDB level:

```sql
SHOW PDBS;
```

At the current container:

```sql
SHOW CON_NAME;
```

General database state:

```sql
SELECT name, dbid, current_scn, log_mode, open_mode
FROM v$database;
```

And most importantly:

- check the actual application objects that mattered
- verify the missing or corrupted data file problem is gone
- confirm that you are at the intended historical point

After any `RESETLOGS`, you should also take a fresh backup soon afterward,
because you now have a new incarnation and a new recovery baseline.

---

## 10. What the Practice Is Actually Teaching

The practice flow is useful if you interpret it correctly:

- create a recoverable setup
- deliberately remove or corrupt a file
- attempt normal recovery
- discover that required redo is missing
- determine a safe target before the gap
- restore and recover only to that earlier point
- open with `RESETLOGS`
- verify that the database or PDB is usable again

That is a good lab.

The modern correction is just this:

- do not build your operational runbook around deprecated DRA commands
- use DRA in the course as historical context
- use supported RMAN point-in-time recovery techniques as the real skill

---

## 11. Practical Takeaways

- Missing archived redo turns a routine media recovery into an incomplete
  recovery decision.
- Incomplete recovery is really a point-in-time recovery workflow with a more
  dramatic origin story.
- In `19c`, Data Recovery Advisor is deprecated and not supported for
  individual `PDB`s.
- For CDBs, keep the recovery scope as small as possible: recover the affected
  `PDB` instead of the entire database whenever that is feasible.
- Use `SET UNTIL` once, restore and recover to that boundary, then `OPEN
  RESETLOGS`.
- Verify the result in SQL, not just in RMAN, and back up again after the new
  incarnation begins.
