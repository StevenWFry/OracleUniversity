# Lab 6-2 (Part 1) - Viewing Advanced Parameters and Parameter Views

And look, at some point in your DBA career you will stare at an Oracle parameter name like `commit_logging` and think, "That sounds important and I have absolutely no idea what it does." This lab is about turning that feeling into something slightly less horrifying.

This aligns to the **advanced parameter** and **view queries** sections of Practice 6-2 in the activity guide.

---

## 1. Assumptions

- You are logged in to the VM as the `oracle` OS user.
- `orclcdb` is up and running.

---

## 2. Connect to SQL*Plus

```bash
. oraenv
ORACLE_SID = [orclcdb] ? orclcdb

sqlplus / as sysdba
```

---

## 3. View Advanced Parameters with `SHOW PARAMETER`

1. `transactions`:

   ```sql
   SHOW PARAMETER transactions;
   ```

2. `db_files`:

   ```sql
   SHOW PARAMETER db_files;
   ```

3. Commit behavior:

   ```sql
   SHOW PARAMETER commit_logging;
   SHOW PARAMETER commit_wait;
   ```

4. Shared pool and block sizes:

   ```sql
   SHOW PARAMETER shared_pool_size;
   SHOW PARAMETER db_block_size;
   SHOW PARAMETER db_cache_size;
   ```

5. Undo mode and memory targets:

   ```sql
   SHOW PARAMETER undo_management;
   SHOW PARAMETER memory_target;
   SHOW PARAMETER memory_max_target;
   SHOW PARAMETER pga_aggregate_target;
   ```

---

## 4. Find Parameter Views in the Data Dictionary

```sql
SET pagesize 100
SELECT table_name
FROM   dictionary
WHERE  table_name LIKE '%PARAMETER%';
```

You should see views such as `V$PARAMETER`, `V$SPPARAMETER`, `V$PARAMETER2`, and `V$SYSTEM_PARAMETER`.

---

## 5. Explore `V$PARAMETER`

1. Describe the view:

   ```sql
   DESC v$parameter
   ```

   Pay attention to `ISSYS_MODIFIABLE`:

   - `FALSE` means static (restart required)
   - `IMMEDIATE` means dynamic

2. Query a subset of columns:

   ```sql
   COLUMN name  FORMAT a35
   COLUMN value FORMAT a20

   SELECT name, issys_modifiable, value
   FROM   v$parameter
   ORDER  BY name;
   ```

3. Filter for parameters with `pool` in the name:

   ```sql
   SELECT name, value
   FROM   v$parameter
   WHERE  name LIKE '%pool%';
   ```

---

## 6. Explore `V$SPPARAMETER`

```sql
DESC v$spparameter;

SELECT name, value
FROM   v$spparameter;
```

If the instance was not started with an SPFILE, `ISSPECIFIED` will be `FALSE` for all rows.

---

## 7. Explore `V$PARAMETER2`

```sql
DESC v$parameter2;

SELECT name, value
FROM   v$parameter2;
```

Multi-valued parameters (like `control_files`) appear once per value.

---

## 8. Explore `V$SYSTEM_PARAMETER`

```sql
DESC v$system_parameter;

SELECT name, value
FROM   v$system_parameter;
```

These are the instance-wide defaults new sessions inherit.

---

## 9. Exit

```sql
EXIT;
```

You now know where Oracle hides parameter metadata and how to tell static from dynamic without guessing.
