# 21 - Space Management Features (Logical Space, Real Consequences)

This chapter goes deeper into tablespace space management: shrinking,
fragmentation control, thresholds, resumable operations, and why "we have plenty
of disk" is not the same as "the database is efficient."

---

## 1. Logical Space Is the Main Battle

Lecture emphasis:

- most Oracle space management is logical, not physical
- goal is compact, efficient block usage in buffer cache
- this is about reducing metadata churn and I/O overhead, not just saving disk GB

Disk is cheap. Wasted logical structure is expensive.

---

## 2. Defragmentation: What and Why

Defragmentation targets:

- object compaction
- extent cleanup/reuse
- reclaiming free space for reuse by other objects/files

Methods discussed:

- shrink operations (fast, practical, partial compaction)
- rebuild/redefinition (heavier, deeper cleanup)

Key nuance:

- shrink helps fragmentation efficiency
- shrink does not solve all row migration issues
- full rebuild/redefinition may still be needed for severe cases

---

## 3. Temp Tables Are Back in the Spotlight

Why temp tables matter more now:

- heavy reporting and transformation workloads
- materialized view maintenance can be expensive in some architectures
- read-only standby patterns complicate some refresh workflows

Global temp tables:

- shared definition, session-private data

Private temp tables (18c+):

- tighter scope/control
- better fit where security and object isolation matter

---

## 4. Compression Tradeoff

Compression can reduce I/O footprint and improve read efficiency, but:

- compression/decompression costs CPU
- choice depends on workload type and access pattern

There is no universal "best compression." There is only "best for this workload."

---

## 5. Space Growth and Resumable Operations

When loads run out of space, you can avoid full rollback pain with resumable
session behavior:

- operation suspends on space error
- alert is raised
- DBA extends/fixes space
- operation resumes from suspension point

This is how you avoid throwing away 90% completed loads because one file filled
up at the end.

---

## 6. OMF and Container-Aware Defaults

OMF is still recommended for operational sanity, but in multitenant:

- ensure per-container destinations are configured correctly
- keep CDB and PDB file placement from stepping on each other

Automation is great, right up until it automatically puts files in the wrong place.

---

## 7. LMT + ASSM Basics (and When Manual Still Wins)

Locally managed tablespaces:

- bitmap-based extent tracking

Segment space management auto:

- automatic free-space handling within blocks

But default automation is not always ideal:

- static data may perform better with manual tuning and lower reserved free space
- update-heavy data often benefits from auto free-space reserve behavior

Same theme as earlier chapters: choose by workload behavior.

---

## 8. Thresholds and Alerts

Warning/critical thresholds are essential for proactive space management.

Set them well:

- too low -> alert spam, ignored signals
- too high -> late detection, emergency changes

Thresholds are only useful when they match actual growth and autoextend behavior.

---

## 9. Block-Level Fragmentation Recap

Two different row issues:

- chaining: row larger than block size (design/block-size problem)
- migration: row moved after growth update (maintenance/layout problem)

Migration cleanup pattern from lecture:

- isolate affected rows
- delete/reinsert (or equivalent controlled rebuild flow)
- restore compact row placement and remove pointer overhead

---

## 10. Extent Bitmaps and Metadata Efficiency

With auto management, bitmaps track:

- extent allocation state
- block usage/free state
- insert eligibility

Benefit:

- fewer dictionary lookups for basic allocation decisions
- faster internal bookkeeping at scale

---

## 11. Index Fragmentation and Rebuild Strategy

Indexes fragment quickly due to ordered structure and mid-range insert patterns.

Operational guidance:

- monitor index usability/efficiency
- rebuild when fragmentation cost outweighs benefit
- for initial bulk loads, create definition then rebuild post-load for compact structure

Because an index that is technically present but operationally useless is just decorative metadata.

---

## 12. Temporary Table Lifecycle Notes

Temp table behavior options:

- `ON COMMIT PRESERVE ROWS` (session scope)
- `ON COMMIT DELETE ROWS` (transaction/commit scope behavior)

Temp workflows can pair with undo/statistics features in modern releases to
support iterative reporting transforms without constant reload.

---

## 13. Key Takeaways

- manage logical space first; physical space is secondary in most cases.
- pick shrink vs rebuild based on problem type and severity.
- use thresholds/resumable operations to prevent avoidable rollback waste.
- treat temp and index space strategy as active tuning domains, not set-and-forget.
- automation helps, but only when container-level configuration is intentional.
