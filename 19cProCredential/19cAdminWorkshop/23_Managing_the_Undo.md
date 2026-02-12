# 23 - Managing the Undo (Old Data, New Problems, Zero Patience)

Chapter 4 focuses on undo: what it is, how Oracle uses it, how retention really
works, and how not to break long-running queries while trying to "optimize."

---

## 1. What Undo Actually Is

Undo is the old version of changed data.

Oracle uses undo for:

- transaction rollback
- read consistency (consistent query snapshots)
- flashback features
- instance/database recovery workflows

Undo is stored in undo tablespace segments. Redo is separate and stores change
records for roll-forward/recovery.

---

## 2. Undo vs Redo (Different Jobs)

Undo:

- old row/block state
- supports rollback, flashback, consistent reads
- stored in undo tablespace

Redo:

- change vectors/formulas of what changed
- supports roll-forward and crash/media recovery
- written to redo logs (and archived for long retention)

If undo is the "before photo," redo is the "how to replay the change" script.

---

## 3. Commit Flow and Where Undo Fits

High-level sequence from lecture:

1. DML changes are made in memory buffers.
2. Redo records are generated in redo buffer.
3. On commit, redo is flushed to redo logs.
4. DBWR later writes dirty blocks to datafiles.
5. Old versions tied to changes are maintained in undo segments per policy.

Important nuance:

- dirty data blocks can be written before commit in some buffer-management paths
- commit durability depends on redo flush, not immediate datafile write

---

## 4. Undo Retention and Reality

`UNDO_RETENTION` defines how long committed undo should be kept.

Why keep it:

- avoid `snapshot too old` for long queries
- support flashback/query back-in-time logic

But retention is not guaranteed unless guarantee mode is enabled.

Without guarantee:

- Oracle may overwrite unexpired undo under space pressure to keep transactions moving

With guarantee:

- Oracle honors retention strictly
- but if undo fills, new transactions can hang

So size and retention must be designed together, not guessed separately.

---

## 5. Sizing Strategy and Advisor Use

DBA responsibilities:

- estimate undo generation rate
- set retention based on longest important queries/jobs
- size undo tablespace so retention can be realistically honored

Use undo advisor tooling (including EM Express paths) to map:

- desired retention -> required undo size

If you still see `snapshot too old`, either retention is too low or space is too small (or both, which is always fun).

---

## 6. Multitenant Undo: Shared vs Local

12.1 behavior:

- shared undo at CDB level

12.2+ preferred behavior:

- local undo enabled for better PDB independence

Benefits of local undo:

- cleaner hot operations for PDB backup/clone/recovery/management

Enable local undo (if not already) via maintenance sequence:

1. shutdown
2. startup in upgrade mode
3. enable local undo
4. restart normally

Oracle manages local undo partitions/structures behind the scenes after conversion.

---

## 7. RAC Undo Rule

In RAC:

- each instance needs its own undo tablespace
- undo is not shared the same way across instances

---

## 8. Operational Pattern: Secondary Undo Tablespace

Lecture pattern:

- keep primary undo as default
- create secondary undo as overflow/control option
- switch temporarily during pressure events
- switch back when primary stabilizes

This is operational flexibility, not a replacement for proper sizing.

---

## 9. Temporary Undo for Temporary Tables

Modern Oracle supports temp undo behavior for temp-table workflows.

Why it matters:

- heavy reporting uses temp tables for staged result sets
- rollback/rework inside temp workflows becomes easier
- avoids excessive pressure on permanent undo in those scenarios

Temp-table data/undo lifecycle follows temp object/session semantics and does
not behave like permanent object logging strategy.

---

## 10. LOB and Undo Caution

For very large object workloads (CLOB/BLOB/video-style payloads):

- avoid designs that explode undo unnecessarily
- prefer reload/rebuild patterns where appropriate instead of forcing massive
 undo retention burden for huge transient operations

Because gigantic undo for one-off payload workflows is how you end up with a
very expensive queue of waiting transactions.

---

## 11. Key Takeaways

- undo is essential for consistency, rollback, and flashback.
- retention without sizing is wishful thinking.
- guarantee mode protects retention but can block throughput if undersized.
- local undo is the practical default in modern multitenant.
- monitor long jobs and tune undo with advisor evidence, not guesswork.
