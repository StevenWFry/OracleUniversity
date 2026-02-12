# Lab 16-3 - Enabling Resumable Space Allocation (Part 2)

Part 2 resolves the suspended statement by enabling autoextend, then confirms
the resumable operation completed and suspension state cleared.

---

## 1. Starting State

From Part 1, `DBA_RESUMABLE` should show a suspended statement similar to:

- user `PDBADMIN`
- session suspended on `ORA-01653` (unable to extend table `PDBADMIN.X` in
  tablespace `INVENTORY`)

---

## 2. Window 2 - Apply Corrective Action

Connect as `system` to `ORCLPDB1`:

```bash
sqlplus system/cloud_4U@orclpdb1
```

Check current autoextend status for inventory datafile:

```sql
SELECT file_name, autoextensible
FROM   dba_data_files
WHERE  tablespace_name = 'INVENTORY';
```

Expected before fix:

- `AUTOEXTENSIBLE = NO`

Enable autoextend on the file:

```sql
ALTER DATABASE DATAFILE '/u01/app/oracle/oradata/ORCLCDB/orclpdb1/inventory01.dbf'
AUTOEXTEND ON;
```

Verify again:

```sql
SELECT file_name, autoextensible
FROM   dba_data_files
WHERE  tablespace_name = 'INVENTORY';
```

Expected after fix:

- `AUTOEXTENSIBLE = YES`

---

## 3. Window 1 - Confirm Resume and Completion

Return to Window 1 (where `CreateTableX.sql` had suspended).

Expected:

- statement resumes automatically after space correction
- insert/load completes
- commit completes

Transcript result:

- `2048` rows inserted and committed after resume

---

## 4. Window 2 - Verify No Suspended Sessions Remain

Query resumable view:

```sql
SELECT status, name, sql_text, error_msg
FROM   dba_resumable;
```

Expected:

- `no rows selected`

This confirms the suspension was resolved and alert condition cleared.

---

## 5. Exit and Close Windows

In both windows:

```sql
EXIT
```

Close terminal windows.

---

## 6. Lab Result

You corrected the space issue by enabling datafile autoextend, the suspended
resumable statement resumed and completed, and `DBA_RESUMABLE` returned empty.
