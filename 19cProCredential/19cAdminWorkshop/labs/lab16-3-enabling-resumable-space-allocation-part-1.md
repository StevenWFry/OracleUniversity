# Lab 16-3 - Enabling Resumable Space Allocation (Part 1)

Practice 3-3 introduces resumable space allocation so long-running DML can
pause on space errors instead of failing immediately.

---

## 1. Practice Goal

In Part 1, you will:

- reproduce a space error in `INVENTORY`
- enable resumable mode
- rerun load to force suspension instead of failure
- inspect suspension state from a second terminal and alert log

This lab uses two terminals (`Window 1` and `Window 2`).

---

## 2. Prepare Two Terminals

Open two terminals and set titles:

- `Window 1`
- `Window 2`

Use terminal menu:

- `Terminal` -> `Set Title`

---

## 3. Window 1 - Baseline Failure Without Resumable

1. Connect to `ORCLPDB1` as `system`:

```bash
sqlplus system/cloud_4U@orclpdb1
```

2. Grant DBA to `pdb_admin`:

```sql
GRANT dba TO pdb_admin;
```

3. Connect as `pdb_admin`:

```sql
CONNECT pdb_admin@orclpdb1
```

4. Create unpopulated `INVENTORY` tablespace:

```sql
@/home/oracle/labs/storage/CreateINVENTORYtablespace.sql
```

5. Run load script:

```sql
@/home/oracle/labs/storage/CreateTableX.sql
```

Expected:

- `ORA-01653` unable to extend `PDB_ADMIN.X` in tablespace `INVENTORY`

Transcript note:

- about `2048` rows are inserted before failure.

---

## 4. Window 1 - Enable Resumable and Rerun

Reconnect as needed:

```sql
CONNECT pdb_admin@orclpdb1
```

Enable resumable mode:

```sql
ALTER SESSION ENABLE RESUMABLE;
```

Rerun load script:

```sql
@/home/oracle/labs/storage/CreateTableX.sql
```

Expected behavior:

- statement suspends instead of immediate hard failure
- default resumable timeout is about `7200` seconds (2 hours)

---

## 5. Window 2 - Verify Suspension

1. Set environment:

```bash
. oraenv
```

When prompted, enter:

```text
orclcdb
```

2. Connect as `system` to `ORCLPDB1`:

```bash
sqlplus system/cloud_4U@orclpdb1
```

3. Query resumable state:

```sql
SELECT session_id, status, timeout, error_msg
FROM   dba_resumable;
```

Expected:

- suspended resumable statement visible
- reason tied to inability to extend table/tablespace

4. Exit SQL*Plus (keep terminal open):

```sql
EXIT
```

5. Check alert log for suspension entry indicating table extension failure.

---

## 6. Part 1 Stopping Point

Part 1 ends after confirming the suspended load from `DBA_RESUMABLE` and alert
log evidence. Part 2 continues with corrective action (space increase) and
automatic resume/completion.
