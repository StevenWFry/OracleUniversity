# 37 - Using External Tables to Load and Transport Data

This chapter covers external tables: what they are, why they are useful, and how
to use Oracle Loader or Data Pump access drivers to move data in and out without
pretending every intermediate dataset needs a permanent internal table.

---

## 1. What an External Table Actually Is

An external table is metadata in Oracle that points to data stored outside the
database (usually files on the server filesystem).

Oracle manages:

- table definition
- access rules

Oracle does **not** manage:

- external file data blocks like internal segments

Think of it as "SQL over files" with Oracle-level structure and privileges.

---

## 2. Why Use External Tables

Common use cases:

- inspect inbound client files before committing to internal tables
- stage intermediate result sets
- combine multiple inbound files in one query surface
- simplify complex data prep workflows before final insert

This is especially useful when the data is temporary, messy, or both.

---

## 3. Access Drivers: Oracle Loader vs Data Pump

External table organization uses access drivers:

- `ORACLE_LOADER`
- `ORACLE_DATAPUMP`

### `ORACLE_LOADER`

Best for reading flat files into SQL-accessible table structures.

### `ORACLE_DATAPUMP`

Supports Data Pump-style file handling and can be used in two-way movement
patterns (export-style generation and import-style readback).

---

## 4. Creating External Tables with `ORACLE_LOADER`

Pattern:

```sql
CREATE TABLE ...
ORGANIZATION EXTERNAL
(
  TYPE ORACLE_LOADER
  DEFAULT DIRECTORY ...
  ACCESS PARAMETERS (...)
  LOCATION (...)
)
PARALLEL ...
REJECT LIMIT ...;
```

Important controls include:

- record delimiters
- field delimiters
- null handling
- bad/log/discard file settings
- datatype conversion rules (for example date masks)
- source file list in `LOCATION`

If dates are character-formatted in files, specify the date mask explicitly or
enjoy debugging at 2 a.m.

---

## 5. Rejects, Bad Files, and Logging

During load/read operations:

- unreadable records/fields are rejected
- bad rows are captured in bad files
- run details go to log files
- `REJECT LIMIT` controls whether processing stops

You can set strict limits or allow broad continuation depending on data quality
policy.

---

## 6. Using `ORACLE_DATAPUMP` with External Tables

You can define external tables backed by Data Pump driver behavior, then use SQL
and Data Pump-style flows to stage or extract datasets.

Transcript pattern highlighted:

- create external table with `ORACLE_DATAPUMP`
- populate/select data from internal joins
- produce multiple output files with parallel processing
- consume that data through external-table definitions

In short: this gives you flexible round-trip pathways, not one-way ingestion.

---

## 7. Querying and Combining External Data

If you have privileges, external tables behave like tables for SQL query logic:

- simple `SELECT`
- joins with internal tables
- `INSERT /*+ APPEND */ ... SELECT ...` into internal tables

That means you can stage externally, validate, enrich, and then persist
internally when ready.

---

## 8. Privileges and Dictionary Views

You need directory/table privileges to use external-table paths.

Useful views mentioned:

- `DBA_EXTERNAL_TABLES`
- `USER_EXTERNAL_LOCATIONS`
- `ALL_/DBA_/USER_TABLES`
- `ALL_/DBA_DIRECTORIES`
- `ALL_TAB_COLUMNS` (or DBA equivalents)

View prefix still means what it always means:

- `USER_`: yours
- `ALL_`: yours + granted
- `DBA_`: everything (if privileged)

---

## 9. Key Takeaways

- external tables are controlled SQL access to external files.
- `ORACLE_LOADER` is flat-file ingestion oriented.
- `ORACLE_DATAPUMP` enables broader movement patterns.
- control/access parameters determine whether loads are clean or chaotic.
- external + internal table joins make staged processing practical and fast.