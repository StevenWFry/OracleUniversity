# 17 - Storage of Data in Blocks (Where Performance Goes to Live or Die)

You have now seen the storage hierarchy. This lesson goes one level deeper into
block behavior, fragmentation mechanics, and why Oracle storage tuning is mostly
about avoiding tiny disasters that become giant disasters later.

---

## 1. Context: This Is the "Foundations" Pass

This module introduces structure and operational basics. Full performance tuning
dives deeper into:

- high watermark effects on full scans
- defragmentation strategies by layer
- compacting objects and metadata overhead tradeoffs

For now: understand the moving parts well enough to avoid self-inflicted pain.

---

## 2. Logical Fragmentation vs Physical Fragmentation

Oracle performance impact is mostly about **logical** fragmentation in database
structures, not traditional file-system defrag panic.

Why:

- all reads/writes are served through buffer cache
- half-full blocks waste cache capacity
- more blocks = more logical I/O and less effective cache density

So when we say "fragmentation," we mean "you are wasting logical memory and
metadata effort," not "go run OS defrag and pray."

---

## 3. Storage Layers (Again, But This Time With Teeth)

Hierarchy:

- database -> tablespaces
- tablespaces -> segments (objects)
- segments -> extents
- extents -> Oracle blocks
- blocks -> rows

Physical files matter for resilience and capacity.
Logical structure matters for performance and manageability.

You need both right. One without the other is how outages become hobbies.

---

## 4. Block Size, Chaining, and Migration Are Not the Same Problem

Row chaining:

- row too large for block size from day one
- row split across blocks
- fixed by bigger block-size tablespace or schema redesign

Row migration:

- row originally fit
- update made row larger
- row moved; old location keeps pointer
- extra I/O to chase pointer

Rebuilds can help migration cleanup.
Rebuilds do **not** fix chaining caused by block-size mismatch.

---

## 5. Buffer Cache Is the Battlefield

If blocks are sparsely used:

- more blocks required per object read
- more buffer cache churn
- less room for other workloads

For static data, forcing lots of reserved free space is just buying empty seats
in the cache and then acting surprised when the system feels crowded.

---

## 6. Bigfile vs Smallfile Tablespaces

Bigfile:

- one very large datafile per tablespace

Smallfile:

- multiple files per tablespace

Choice depends on storage platform limits and workload behavior. With highly
distributed object placement, metadata navigation overhead can increase.

As always, storage design is engineering, not astrology.

---

## 7. Core Default Tablespaces and Their Roles

Created by default:

- `SYSTEM`
- `SYSAUX`
- `UNDO`
- `TEMP`
- `USERS`
- `EXAMPLE` (if sample schemas are installed)

Key nuance:

- `SYSTEM` is critical for live dictionary-driven operation
- `SYSAUX` is largely historical/auxiliary workload and can often be managed
 with less direct runtime impact

---

## 8. Segment Creation Timing: Deferred vs Immediate

Deferred segment creation is generally preferred:

- metadata created first
- physical segment allocated at first real data load

Why this helps:

- lets you control initial extent sizing around actual load volume
- avoids premature tiny allocations and fragmented early growth

Scope control options:

- database default
- session override (`ALTER SESSION`)
- object-level override in DDL (`SEGMENT CREATION IMMEDIATE|DEFERRED`)

Use object-level override when you need precision without flipping system knobs
all day like a game-show host.

---

## 9. Space Threshold Alerts: Useful, Until Misconfigured

Oracle monitors tablespace utilization with warning and critical thresholds.

Important catch:

- allocated extents count as used at tablespace level even if not fully consumed
 inside the segment yet

If thresholds are set too aggressively, you get alert spam.
Then real alerts get ignored. Then bad things happen.

Set thresholds according to actual autoextend and growth behavior so alerts stay
signal, not noise.

---

## 10. What to Tune at Which Level

Tablespace-level:

- extent management mode
- segment space management mode
- block-size strategy (where needed)

Database/session/object-level:

- deferred/immediate segment creation behavior
- parameter defaults and selective overrides

Design principle:

- defaults should fit most objects
- exceptions should be explicit and local

---

## 11. Takeaways

- block-level behavior drives real-world performance
- chaining and migration are different and need different fixes
- buffer cache efficiency is the true cost center for fragmentation
- deferred segment creation is usually the sane default
- threshold alerts are only useful if calibrated to reality

Next step from lecture:

- live demo navigating tablespaces and storage metadata in a container database.
