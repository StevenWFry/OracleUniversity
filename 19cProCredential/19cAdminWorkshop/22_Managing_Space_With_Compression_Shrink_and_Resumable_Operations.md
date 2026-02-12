# 22 - Managing Space with Compression, Shrink, and Resumable Operations

This chapter extends space management from "know your tablespaces" into "keep
them efficient while the system is actually busy."

---

## 1. Global vs Private Temporary Tables

Global temporary tables:

- shared definition
- session-private data copies
- broad visibility of definition can be risky if used carelessly with sensitive source logic

Private temporary tables (18c+):

- private definition and private data scope
- better isolation for reporting/staging workflows
- useful naming control via `PRIVATE_TEMP_TABLE_PREFIX`

Commit behavior options:

- preserve rows for session scope
- delete rows at commit for transaction-scope behavior

---

## 2. Compression Options and Tradeoffs

`COMPRESS BASIC`:

- geared toward direct-path load scenarios
- classic warehouse-style "load then read" use cases

`ROW STORE COMPRESS ADVANCED`:

- row-store dedupe/squeeze behavior
- often good read efficiency with manageable overhead patterns
- can still introduce transactional overhead in specific moments

Use Compression Advisor (`DBMS_COMPRESSION` / advisor tooling) to estimate
benefit before flipping settings based on optimism alone.

---

## 3. Fragmentation, High Watermark, and Shrink Workflow

Shrink is a practical default for many defragmentation tasks:

1. `SHRINK SPACE COMPACT`
 - compacts rows, minimal disruptive locking profile
2. `SHRINK SPACE`
 - resets high watermark and releases reusable space metadata

Why two steps:

- compact first to reduce disruption
- then perform the quick HWM reset pass

Important note:

- table shrink generally preserves RowID semantics well enough that mandatory index rebuild is not the default outcome
- indexes may still need independent maintenance if they are fragmented

---

## 4. Chaining vs Migration (Again, Because This Keeps Breaking Teams)

Chaining:

- row too large for block size
- solved via larger block-size tablespace or schema redesign

Migration:

- row grows after update and moves
- solved via maintenance/reinsert/reorg techniques

One is design. One is lifecycle maintenance. Treating them as the same problem
is how tuning meetings become mythology.

---

## 5. Extent/Segment Metadata and Auto Management

Auto extent and segment space management use bitmaps to track:

- extent allocation
- block availability
- insert eligibility

This reduces metadata lookup overhead and improves allocation behavior, but
defaults are still workload-dependent, not universally perfect.

---

## 6. Index Space Maintenance

Indexes can fragment quickly due to ordered key behavior and insert patterns.

Maintenance choices:

- shrink
- coalesce
- rebuild

For bulk-load workflows:

- create definition (often unusable first)
- load data
- rebuild once for compact structure

---

## 7. Threshold Alerts and Capacity Signals

Use warning/critical thresholds on tablespaces to detect risk early.

Good thresholds:

- actionable
- aligned to growth behavior and autoextend realities

Bad thresholds:

- alert noise machine nobody trusts

---

## 8. Resumable Operations (Don’t Roll Back 90% of a Load)

Resumable session/operation lets large DDL/DML pause on space errors instead of
immediate rollback.

Flow:

1. enable resumable
2. run operation
3. if space issue occurs, operation suspends and alert is raised
4. DBA fixes space
5. operation resumes

If unresolved by timeout, operation can abort/rollback based on policy.

This is the difference between a controlled interruption and wasting hours of
completed work.

---

## 9. Practical Monitoring Loop

Use:

- `DBA_TABLESPACES`
- `DBA_SEGMENTS`
- `DBA_TABLES` (including chain/migration indicators)
- EM/Cloud Control extent maps where available

Monitor regularly, act early, and keep objects compact before they become
performance archaeology projects.
