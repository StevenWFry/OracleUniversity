## Lesson 19 - Data Recovery Advisor (in which Oracle tries to diagnose failures for you, but also quietly admits this feature is on its way out)

And look, Data Recovery Advisor sounds like exactly what every DBA wants: a tool that tells you what failed, suggests what to do, and sometimes even writes the repair script for you. Which is wonderful, except for the awkward little detail that in `19c` Oracle has already marked it deprecated.

By the end of this lesson, you should be able to:

- Explain what Data Recovery Advisor (`DRA`) was designed to do
- Understand the classic `LIST` / `ADVISE` / `REPAIR` workflow
- Recognize the kinds of failures DRA can diagnose
- Know the supporting `V$IR_*` views
- Place DRA in the correct `19c` context: useful to understand, not a future-proof strategy

---

## 1. What Data Recovery Advisor Was Meant to Be

Data Recovery Advisor is an Oracle diagnostic and repair framework that can:

- detect failures already recorded by the database
- list and classify them
- suggest repair options
- generate repair scripts for some failures
- run certain repairs automatically through RMAN

It can be accessed through:

- RMAN
- Enterprise Manager Cloud Control

That made it appealing because it could shorten outage time by turning:

- "something failed"

into:

- "here is what failed, here are your repair options, and here is the script if
  Oracle knows how to fix it"

An excellent concept.
Also, in classic Oracle fashion, now deprecated.

---

## 2. The Big `19c` Caveat

Starting in Oracle Database `19c`, Data Recovery Advisor is deprecated.

That deprecation includes these RMAN commands:

- `LIST FAILURE`
- `ADVISE FAILURE`
- `REPAIR FAILURE`
- `CHANGE FAILURE`

Oracle documentation says there is **no replacement feature**.

So the right way to study this in `19c` is:

- understand how it works
- recognize the terminology
- know what older environments and older course material mean
- do not treat it as the long-term centerpiece of your operational practice

In other words, this is now part of Oracle history that still matters on exams,
older systems, and inherited environments.

---

## 3. Where DRA Fits in the Bigger Picture

DRA is not the only recovery tool.

Use the broader toolbox correctly:

- `Data Guard`
  for rapid failover to a standby system
- `Flashback`
  for many logical error scenarios
- `RMAN`
  for restore, recovery, block repair, and backup-based recovery
- `ADR` / `ADRCI`
  for diagnostics and incident evidence

DRA sits in the middle as an advisory layer:

- identify failures
- show repair options
- automate some of the repair workflow

So it is not the whole recovery strategy.
It is Oracle's attempt to put a helpful layer on top of the strategy.

---

## 4. Scope and Limitations

The source is broadly right about the limitations.

DRA is intended for:

- single-instance databases

It does **not** support repair operations for:

- RAC databases
- physical standby databases

It can, however, work in environments that include standby configurations in the
sense that primary-side failures may still be diagnosed, but you do not use DRA
to analyze or repair the standby itself as if it were a single-instance local
toy.

So yes, DRA has boundaries, and they matter.

---

## 5. The Basic DRA Workflow

The classic RMAN workflow is:

1. list failures
2. get repair advice
3. repair the failure
4. confirm that the failure is gone or closed

The commands are:

```rman
LIST FAILURE;
LIST FAILURE DETAIL;
ADVISE FAILURE;
REPAIR FAILURE;
```

And optionally:

```rman
CHANGE FAILURE ... ;
```

Again: these are deprecated in `19c`.
But if you are learning what older material means, this is the sequence it is
talking about.

---

## 6. Failure Severity and What It Means

Failures are classified by priority / severity levels such as:

- `CRITICAL`
- `HIGH`
- `LOW`

General idea:

- `CRITICAL`
  impacts critical files or operations required for database operation
- `HIGH`
  serious issue, often affecting non-critical files or significant data access
- `LOW`
  lower-priority issue, often less urgent operationally

The priority system exists so you do not waste the first minutes of an incident
treating a papercut and a severed artery as identical administrative moods.

---

## 7. What Kinds of Failures DRA Can Diagnose

Oracle documents that DRA can diagnose failures such as:

- missing or inaccessible data files and control files
- permission problems on needed files
- offline files or related access issues
- physical corruptions such as checksum failures or invalid block headers
- inconsistencies such as a data file older than the rest of the database
- I/O failures such as hardware errors or OS resource-limit problems

It may detect or help with some logical corruptions, but Oracle notes that
logical corruption often requires help from Oracle Support.

So DRA is good at structured diagnosis.
It is not magic.
And it is certainly not a replacement for understanding what kind of failure you
are actually staring at.

---

## 8. `LIST FAILURE` and `LIST FAILURE DETAIL`

The first classic step is simply:

```rman
LIST FAILURE;
```

That shows detected failures already known to Oracle.

If you want more detail:

```rman
LIST FAILURE DETAIL;
```

Oracle also supports filters such as:

- only critical failures
- only high or low failures
- specific failure numbers
- excluding certain failures

The point here is simple:

- find out what Oracle already knows before you start inventing theories

Which is a surprisingly valuable discipline in database work.

---

## 9. `ADVISE FAILURE`

Once the failure list exists, the next classic step is:

```rman
ADVISE FAILURE;
```

What this provides:

- summary of the failure
- recommended repair options
- warnings or prerequisites
- manual checklist items where needed
- a generated repair script when Oracle can automate the fix

This is important because "Oracle generated a script" does **not** always mean:

- run one command and go home

Sometimes the advice includes:

- manual preparation
- script execution
- post-repair manual steps

So the recommendation is a workflow, not always a one-button miracle.

---

## 10. Repair Scripts and the `hm` Directory

When DRA generates a repair script, Oracle stores it under the `hm` directory in
the ADR home.

The source is correct about the general pattern:

- the filename usually begins with `reco`
- it is associated with the diagnostic / process context that produced it

This matters because:

- the script is evidence of what Oracle thinks the repair should be
- you can inspect it before running it

Which is a wonderful idea, because blindly running repair commands is an
excellent way to discover the second problem hiding behind the first one.

---

## 11. `REPAIR FAILURE`

If you accept the recommended action, the classic command is:

```rman
REPAIR FAILURE;
```

Useful options include:

- specifying a repair option when multiple options exist
- `PREVIEW`
  to see what would happen without executing it
- `NOPROMPT`
  to avoid confirmation prompts

Important Oracle rule:

- only one RMAN session may run `REPAIR FAILURE` at a time
- `REPAIR FAILURE ... PREVIEW` is the exception

So no, this is not a command you casually fire off from three windows at once
like you are trying to summon a database demon.

---

## 12. `CHANGE FAILURE`

If you handle things manually, you may still need to adjust failure metadata.

That is where `CHANGE FAILURE` came in:

- close repaired failures
- change priority to `HIGH` or `LOW` where permitted

Important Oracle nuance:

- you cannot change a `CRITICAL` failure to another priority

Which is Oracle's way of saying:

- "no, you do not get to relabel a heart attack as mild inconvenience"

Again, in `19c` this syntax is deprecated.

---

## 13. Supporting Views

The source correctly points to the internal views that expose DRA-related
information.

Important ones include:

- `V$IR_FAILURE`
- `V$IR_MANUAL_CHECKLIST`
- `V$IR_REPAIR`
- `V$IR_FAILURE_SET`

What they are for:

- `V$IR_FAILURE`
  failure records, including status and priority
- `V$IR_MANUAL_CHECKLIST`
  manual actions associated with advised repair flows
- `V$IR_REPAIR`
  repair options / repair information
- `V$IR_FAILURE_SET`
  cross-reference between failures and advice / repair grouping

These views are useful when:

- you want more detail than RMAN text output gives you
- you want to inspect failure metadata directly
- you are trying to understand what DRA thinks is happening under the hood

Representative query:

```sql
SELECT failure#, priority, status, summary
FROM   v$ir_failure
ORDER BY failure#;
```

Again, useful mostly for understanding and legacy environments in `19c`, not
for building your future religion.

---

## 14. Practical Example Flow

The classic legacy DRA flow looks like this:

```rman
LIST FAILURE;
LIST FAILURE DETAIL;
ADVISE FAILURE;
REPAIR FAILURE PREVIEW;
REPAIR FAILURE;
LIST FAILURE;
```

Interpretation:

- first list what Oracle knows
- then ask for repair options
- preview before executing if you want sanity
- repair if appropriate
- then confirm whether the failure is gone or closed

And yes, one repaired failure can expose a second previously hidden one.
Oracle is generous like that.

---

## 15. Demo-Style Failure Repair Walkthrough

The demo version makes this much more concrete:

1. start the CDB
2. discover that a PDB will not open
3. see the missing or damaged data file error
4. switch to RMAN
5. use DRA commands to diagnose and repair
6. reopen the PDB and confirm it is usable again

Representative SQL*Plus flow:

```sql
STARTUP;
SHOW PDBS;
ALTER PLUGGABLE DATABASE pdb1 OPEN;
```

At that point, Oracle throws the real clue:

- a specific data file is missing or inaccessible

That is your pivot point.
You stop treating this like a mysterious mood disorder and start treating it
like file loss.

Then in RMAN:

```rman
LIST FAILURE;
ADVISE FAILURE;
REPAIR FAILURE;
LIST FAILURE;
```

What the demo emphasizes:

- `LIST FAILURE` shows the current failure records and priority
- `ADVISE FAILURE` gives the repair strategy and may point to the generated
  script in the ADR `hm` directory
- `REPAIR FAILURE` can run the generated repair steps for you
- a second `LIST FAILURE` confirms whether any failure remains open

In the sample scenario, the generated repair script effectively does what you
would otherwise type yourself:

- take the affected data file offline
- restore it
- recover it
- bring it back online

Then you return to SQL*Plus and try again:

```sql
ALTER PLUGGABLE DATABASE pdb1 OPEN;
SHOW PDBS;
SHOW CON_NAME;
```

If the repair succeeded, the PDB moves from `MOUNTED` to `READ WRITE`.

Which is Oracle's version of saying:

- "yes, the patient is alive, stop poking it nervously."

---

## 16. Practice-Style Lab Flow

The practice version is a more staged classroom disaster:

1. start the database and listener
2. run a recovery configuration script
3. run a setup script that creates a tablespace, data, and backup context
4. run a break script that deletes a data file
5. start the CDB and observe the PDB open failure
6. switch to RMAN and use DRA to diagnose and repair
7. reopen the PDB
8. run the cleanup script

The lab sequence is trying to teach one clean lesson:

- a missing PDB data file can leave the CDB up while the affected PDB refuses to
  open

That is exactly the kind of failure where the DRA demonstration feels neat,
because it is isolated enough to show off without the entire database collapsing
into theater.

Representative RMAN portion:

```rman
LIST FAILURE;
ADVISE FAILURE;
REPAIR FAILURE;
```

Then back in SQL*Plus:

```sql
ALTER PLUGGABLE DATABASE orclpdb1 OPEN;
SHOW PDBS;
```

And after the repair succeeds, the final confirmation is that the PDB opens in
`READ WRITE` mode again.

The practice also quietly reinforces a good habit:

- verify the post-repair state in SQL, not just in RMAN output

Because "RMAN said it ran" and "the application data is actually usable again"
are related ideas, not identical ones.

---

## 17. What to Use in `19c` Instead

Because DRA is deprecated in `19c`, your practical emphasis should move toward:

- `ADR` and `ADRCI`
- alert log and trace files
- `V$DIAG_INFO`
- `VALIDATE` and `BACKUP ... VALIDATE`
- `V$DATABASE_BLOCK_CORRUPTION`
- standard RMAN restore / recover workflows
- Flashback features for logical damage
- My Oracle Support for suspected bugs or internal errors

So the modern stance is:

- know DRA because Oracle training and older environments mention it
- use the broader diagnostic and recovery toolkit for real `19c` operations

Which is less elegant than "one tool solves everything," but considerably more
honest.

---

## 18. Practical Takeaways

If you see DRA content in Oracle material, remember:

1. it was designed to list, advise, and repair failures
2. it works through RMAN and Cloud Control
3. it can generate repair scripts and manual checklists
4. it is limited mainly to single-instance database scenarios
5. in `19c`, it is deprecated and should be treated as legacy knowledge

That last point matters the most.
Because the worst kind of study note is one that sounds confident and is already
gently rotting.

---

## 19. Wrap-Up

Data Recovery Advisor was Oracle's attempt to make failure diagnosis and repair
more guided:

- list the failures
- advise the repair
- generate scripts where possible
- execute or preview the repair
- track failure status in internal views

It is still useful to understand.
It is just no longer the future.

And that, frankly, is a very Oracle ending.
