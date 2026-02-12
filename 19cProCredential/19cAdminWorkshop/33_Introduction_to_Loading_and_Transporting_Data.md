# 33 - Introduction to Loading and Transporting Data

This chapter introduces Oracle data movement fundamentals: when to use Data
Pump, when to use SQL*Loader, and how not to confuse the two just because both
move bytes.

---

## 1. The Two Main Tools

Primary tools covered:

- Oracle Data Pump (`expdp`, `impdp`, `DBMS_DATAPUMP`)
- SQL*Loader (`sqlldr`)

Both sit on Oracle data movement infrastructure, but they solve different
problems.

---

## 2. Data Pump at a Glance

Data Pump is Oracle's logical export/import framework.

Entry points:

- CLI: `expdp`, `impdp`
- PL/SQL: `DBMS_DATAPUMP`
- GUI/orchestration tools

Capabilities called out:

- selective object/data movement
- parallel execution
- version-aware import/export behavior
- network-link transfer
- remap operations (schema/table/tablespace)
- restart and reattach support
- metadata/data compression
- encryption for secure dump files

This is the "serious migration workhorse" tool.

---

## 3. Data Pump Path Styles Mentioned

The lecture references:

- conventional path behavior
- direct path behavior
- external-table based loading paths

Also included: transportable-oriented movement scenarios where supported.

---

## 4. SQL*Loader at a Glance

SQL*Loader ingests flat-file data into Oracle tables.

It needs:

- input file(s)
- a control file that explains record/field format and mapping

It processes records, classifies failures, and produces operational artifacts:

- `.bad` for rejected rows
- optional discard file for discarded records
- run log with totals/status

---

## 5. Use-Case Positioning

Use Data Pump when moving Oracle logical objects/data between Oracle systems
with lifecycle controls.

Use SQL*Loader when ingesting file-based external data into Oracle tables.

They are complementary, not interchangeable twins.

---

## 6. Key Takeaways

- Data Pump is export/import governance with restartability and remap power.
- SQL*Loader is deterministic file-to-table ingestion.
- both matter, but for different stages of a data movement pipeline.