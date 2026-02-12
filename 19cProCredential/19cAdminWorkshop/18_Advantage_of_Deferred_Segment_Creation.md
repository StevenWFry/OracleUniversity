# 18 - Advantage of Deferred Segment Creation (Allocate Late, Regret Less)

This lesson demo ties storage theory to real metadata inspection: why deferred
segment creation helps, how to verify object/tablespace behavior from data
dictionary views, and what EM Express can and cannot show you.

---

## 1. Why Deferred Segment Creation Is Useful

Deferred segment creation means:

- object metadata exists immediately
- physical segment space is not allocated until real data arrives

Advantages:

- avoids pre-allocating segment space for empty objects
- reduces early fragmentation pressure
- lets you choose better initial extent sizing when you actually load data
- keeps metadata cleaner for environments with many provisioned-but-unused tables

In short: stop paying storage rent for objects that contain exactly nothing.

---

## 2. Demo Context: Start at CDB Root

The demo starts in root:

```sql
CONNECT / AS SYSDBA
SHOW CON_NAME
SHOW CON_ID
```

Why that matters:

- root context can inspect root-only metadata (`DBA_` views)
- root can also inspect all containers via `CDB_` views

---

## 3. Inspecting Tablespace Metadata

View structure:

```sql
DESC dba_tablespaces;
```

This exposes details like:

- tablespace configuration and management attributes
- logging flags
- in-memory and sharing properties
- statistics and status metadata

Basic listing pattern:

```sql
SELECT tablespace_name
FROM   dba_tablespaces;
```

Cross-container pattern from root:

```sql
SELECT con_id, tablespace_name
FROM   cdb_tablespaces
ORDER  BY con_id, tablespace_name;
```

---

## 4. Inspecting Table-Level Fragmentation Signals

Table metadata structure:

```sql
DESC dba_tables;
```

Key columns highlighted in lesson:

- `CHAIN_CNT` (combined chained + migrated rows)
- `AVG_SPACE` (space usage signal)
- row count and storage behavior attributes

Example inspection query:

```sql
SELECT table_name, chain_cnt, avg_space
FROM   dba_tables;
```

Interpretation from lecture:

- nonzero `CHAIN_CNT` means investigate chaining vs migration
- fix true chaining first (block-size/design issue)
- remaining counts are migration candidates (maintenance/reorg issue)

---

## 5. Why This Connects Back to Deferred Segments

Deferred segments help keep the starting state cleaner:

- fewer unnecessary early allocations
- less "empty-but-reserved" segment clutter
- easier initial extent planning at load time

Then ongoing health is monitored with the same dictionary views above.

So deferred creation is not magic, but it gives you a better opening move.

---

## 6. Tooling Reality: EM Express vs Cloud Control

Demo takeaway:

- EM Express gives basic visibility (undo, redo groups, control files, users)
- EM Express does not provide full deep tablespace/extent management workflows
- Cloud Control provides richer diagnostics (including detailed extent mapping
 and deeper storage analysis)

Translation:

- EM Express is fine for quick checks
- serious storage forensics usually needs SQL + Cloud Control

---

## 7. Practical Administration Pattern

1. Use dictionary views (`DBA_`, `CDB_`) to baseline storage metadata.
2. Track `CHAIN_CNT`, extent growth patterns, and space usage trends.
3. Apply maintenance where needed (table move/redefinition, index rebuilds, etc.).
4. Use deferred segment creation by default for newly provisioned objects unless
 workload demands immediate allocation.

---

## 8. Key Takeaways

- deferred segment creation is a storage hygiene advantage, especially at scale.
- root vs container view choice (`DBA_` vs `CDB_`) changes what you can see.
- chaining and migration diagnostics start in metadata, not guesswork.
- EM Express is useful but intentionally limited for advanced storage management.
