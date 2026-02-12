# 20 - Viewing Tablespace Information (Know What You Can See, and Why)

This chapter focuses on reading tablespace metadata correctly, understanding
view scope by privilege, and using Oracle Managed Files (`OMF`) so tablespace
creation does not turn into manual file-path choreography.

---

## 1. First Rule: `ALL_TABLES` Is About Privilege, Not "All Data Everywhere"

Important clarification from the lesson:

- `ALL_*` views show objects you have privilege to access
- they do **not** mean "all objects in the database/tablespace"

View scope summary:

- `USER_*`: objects owned by current user
- `ALL_*`: objects current user can access
- `DBA_*`: all objects in current container (for DBA privilege)
- `CDB_*`: all containers (from root, with CDB privilege)

So as DBA, you generally inspect `DBA_*` / `CDB_*`, not `ALL_*`, unless you are
debugging user-visible access behavior.

---

## 2. OMF Parameters That Matter

When Oracle Managed Files is enabled, these parameters drive destination logic:

- `db_create_file_dest` -> default datafile destination
- `db_create_online_log_dest_n` -> redo log destinations by group/member design
- `db_recovery_file_dest` -> recovery area (archives, backups, flashback, etc.)
- `diagnostic_dest` -> traces, alerts, incidents, diagnostics

With those set, you can often run:

```sql
CREATE TABLESPACE tbs_name;
```

and Oracle handles file naming/location defaults behind the scenes.

Without those settings, Oracle asks the obvious question: "Great. Where exactly
do you want the file?"

---

## 3. How Tablespaces Grow and Shrink

Tablespaces grow as segments consume extents.

Space management actions:

- add/release extents through object growth/reorg
- resize files manually when needed
- use bigfile/smallfile strategy according to storage behavior

Bigfile tablespace:

- single file defines overall growth path

Smallfile tablespace:

- multiple files can share growth burden

---

## 4. Moving Datafiles Online (12c+)

From 12c onward, datafile movement can be done online in many scenarios.

Common uses:

- move from one filesystem path to another
- move between storage technologies (for example, filesystem <-> ASM)
- rename/migrate file placement without taking workload offline

Command pattern:

```sql
ALTER DATABASE MOVE DATAFILE 'old_path' TO 'new_path';
```

Tablespace-level settings still use `ALTER TABLESPACE` where appropriate.

---

## 5. Metadata Views vs Runtime Views

Dictionary metadata view examples:

- `DBA_TABLESPACES`, `CDB_TABLESPACES`
- describe definition/state/config attributes

Runtime/performance state example:

- `V$TABLESPACE`
- reflects active operational status context

Together they answer:

- "How is this tablespace defined?"
- "What is it doing right now?"

You need both to troubleshoot properly.

---

## 6. Demo Flow Captured in Chapter

Transcript walkthrough sequence:

1. Connect as SYSDBA at root.
2. Compare object visibility logic (`DBA_*` vs `ALL_*` vs container-aware scope).
3. Inspect tablespace metadata:

```sql
DESC dba_tablespaces;
DESC v$tablespace;
```

4. Check destination parameters:

```sql
SHOW PARAMETER dest;
```

5. Attempt create without `db_create_file_dest` (fails due to missing file destination).
6. Set OMF destination:

```sql
ALTER SYSTEM SET db_create_file_dest='/u01/app/oracle/oradata/ORCL' SCOPE=BOTH;
```

7. Create tablespace with minimal syntax:

```sql
CREATE TABLESPACE ron1;
```

8. Drop behavior:

```sql
DROP TABLESPACE ron1;
```

If empty, drop succeeds directly. If not empty, Oracle requires explicit
contents/datafile handling.

---

## 7. Practical Takeaways

- do not misread `ALL_*` views as universal object inventories.
- OMF parameters remove repetitive DDL noise and reduce path mistakes.
- use `DBA_*`/`CDB_*` for metadata truth, `V$` views for runtime truth.
- online file movement is a major operational advantage in 12c+.
- Oracle's "are you sure?" behavior on drops is a feature, not an inconvenience.
