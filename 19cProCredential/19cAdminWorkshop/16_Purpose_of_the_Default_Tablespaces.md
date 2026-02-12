# 16 - Purpose of the Default Tablespaces (And Why "Just Put It Anywhere" Is Not a Storage Strategy)

This lesson continues storage design by answering a deceptively simple question:
what are default tablespaces for, and how do tablespace/segment/block choices
affect performance, maintenance, and your future stress levels.

---

## 1. Why Tablespaces Exist in the First Place

Tablespaces are logical containers for objects with similar behavior.

That means you separate by workload pattern, not by vibes:

- small indexes vs very large indexes
- read-only reference data vs transactional data
- static data vs frequently updated data

Reason:

- different object classes need different backup/recovery and maintenance
 strategies.

Example from lecture:

- small indexes can often be rebuilt quickly
- very large indexes are expensive to rebuild, so they deserve separate backup
 and recovery planning

---

## 2. Extents and Fragmentation: What Actually Hurts

Inside tablespaces:

- objects are segments
- segments grow by extents
- extents are contiguous sets of Oracle blocks (logically contiguous)

Modern Oracle extent auto-management usually works well, but fragmentation still
happens as objects grow and compete for space.

Important nuance:

- performance pain is often metadata overhead from too many tiny extents, not
 "physical disk contiguity" in the old-school sense.

So yes, rebuilding objects still has a place when extent sprawl gets silly.

---

## 3. Block Internals and Row Behavior

An Oracle block is where rows live. Rows are tracked via row directory entries
in block headers.

When block free space is exhausted, inserts go elsewhere. Then two different row
problems appear:

1. Row migration:
 - row originally fits block
 - update makes row bigger
 - row moves, pointer remains at original location
 - extra I/O needed to chase pointer

2. Row chaining:
 - row is too large to fit block size from the start
 - row is split across multiple blocks
 - design problem, not a "rebuild and hope" problem

---

## 4. RowID and Why Migration Costs You

Row access uses RowID components that identify:

- datafile/tablespace context
- segment context
- block
- row slot

Row migration keeps logical identity but adds indirection. So indexed access can
become:

- read block from RowID
- discover pointer
- read second block

That extra read path is exactly the kind of tiny overhead that becomes a big
problem at scale.

---

## 5. Fixing Migration vs Fixing Chaining

Migration cleanup approaches:

- move affected rows out (temp table/export method)
- delete/reinsert (or reimport) so new RowIDs are assigned cleanly

Chaining cleanup:

- rebuild alone does not solve block-size mismatch
- move object to tablespace with larger block size
- or redesign table structure if row width is inherently too large

Short version:

- migration = operational maintenance issue
- chaining = storage design issue

---

## 6. Segment Space Management: AUTO vs MANUAL

`SEGMENT SPACE MANAGEMENT AUTO`:

- Oracle reserves free space in blocks (lecture calls out ~25%) for row growth
- reduces migration risk on update-heavy transactional tables

`SEGMENT SPACE MANAGEMENT MANUAL` with low `PCTFREE`:

- packs rows more tightly
- ideal for mostly static/read-only data where row size rarely changes
- reduces buffer cache waste from half-empty blocks

Why this matters:

- this is not just "disk efficiency"
- it directly affects buffer cache consumption and scan efficiency

So for static tables, reserving lots of update headroom is just paying rent on
empty space forever.

---

## 7. Default Tablespaces and Core Design Rules

Default/system tablespaces exist for Oracle internals.
Application storage should be intentionally separated by behavior:

- transactional tables with growth-by-update patterns
- static lookup/reference tables
- large-object tables
- index classes by size/usage
- temp workloads by report/sort intensity

Extent auto-management is generally fine almost everywhere.
Segment space strategy should be chosen based on update behavior.

---

## 8. Practical Takeaways

- tablespaces are workload boundaries, not decorative folders.
- too many tiny extents increase metadata overhead.
- migration and chaining are different problems with different fixes.
- AUTO segment space is great for volatile rows; MANUAL/low `PCTFREE` is better
 for static data.
- good storage design is mostly about reducing future maintenance, not looking
 clever on day one.
