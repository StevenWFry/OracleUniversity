# 34 - Loading Data with SQL*Loader

This chapter dives into SQL*Loader for loading data from non-Oracle sources or
client-delivered flat files.

---

## 1. SQL*Loader Building Blocks

Core artifacts:

- input data file(s)
- control file (parsing/mapping rules)
- optional discard file
- default bad file (`.bad`) for rejected rows
- log file for end-to-end run summary

Control file defines:

- field datatypes
- delimiters
- row terminators
- column-to-table mapping
- reject/discard thresholds

---

## 2. Processing Pipeline

1. parse control file
2. evaluate record readability
3. discard unreadable records
4. validate field-level mapping/constraints
5. reject invalid rows (written to `.bad`)
6. insert accepted rows
7. write run log

Simple in principle, unforgiving when your control file lies.

---

## 3. Conventional Path Load

Conventional path behaves like row-by-row insert processing:

- redo generated
- constraints enforced
- insert triggers fired
- index entries maintained during load
- table concurrency generally less restrictive

Tradeoff: more complete row-level behavior, lower throughput.

---

## 4. Direct Path Load

Direct path is optimized for speed:

- lower overhead vs conventional path
- enforces core constraints (`PRIMARY KEY`, `UNIQUE`, `NOT NULL`)
- insert triggers are not fired
- index maintenance deferred until load end
- table locking behavior is stricter during load

Tradeoff: faster load, less flexible runtime behavior.

---

## 5. Data Save in Direct Path

Direct path supports save behavior to reduce loss exposure in instance-failure
scenarios.

Parameters like `ROWS` influence save checkpoints for large direct-path loads.

---

## 6. Express Mode

Express mode is "skip manual control-file authoring" when file and table layout
match exactly.

Characteristics:

- system derives load behavior automatically
- expects strict table-column order/format alignment
- best for clean, predictable inbound files

Great when input is well-behaved. Painful when it is not.

---

## 7. Loading into PDBs

SQL*Loader works for PDB targets as long as connection/service points to the
correct PDB.

Same loader semantics, different destination endpoint.

---

## 8. Key Takeaways

- SQL*Loader is the primary file-ingestion utility.
- control-file accuracy decides success rate.
- choose conventional for full row semantics.
- choose direct path when throughput wins and restrictions are acceptable.
- express mode saves time when structure alignment is exact.