# Lab 6-1 - Viewing Basic Initialization Parameters with SQL*Plus

And look, initialization parameters are how the instance tells you what kind of day it's having. In this lab you'll use `SHOW PARAMETER` to check the core settings that actually matter.

This aligns to the **basic parameter** section of Practice 6-2 in the activity guide.

---

## 1. Assumptions

- You are logged in as the `oracle` OS user.
- The CDB `orclcdb` is running.

---

## 2. Set the Environment and Connect

```bash
. oraenv
ORACLE_SID = [orclcdb] ? orclcdb

sqlplus / as sysdba
```

---

## 3. Global Database Name: `DB_NAME` and `DB_DOMAIN`

```sql
SHOW PARAMETER db_name;
SHOW PARAMETER db_domain;
```

Together these form `db_name.db_domain`. In this lab, `db_domain` is typically blank.

---

## 4. Fast Recovery Area (FRA)

```sql
SHOW PARAMETER db_recovery_file_dest;
SHOW PARAMETER db_recovery_file_dest_size;
```

These define the FRA location and its hard size limit.

---

## 5. SGA Sizing

```sql
SHOW PARAMETER sga;
```

`sga_target` enables Automatic Shared Memory Management (ASMM). `sga_max_size` is the ceiling.

---

## 6. Undo Tablespace

```sql
SHOW PARAMETER undo_tablespace;
```

This tells you which undo tablespace is used at startup.

---

## 7. Compatibility

```sql
SHOW PARAMETER compatible;
```

This controls the minimum release compatibility (required to keep multitenant features available).

---

## 8. Control Files

```sql
SHOW PARAMETER control_files;
```

Oracle recommends multiplexing control files on separate disks.

---

## 9. Workload Limits: `PROCESSES` and `SESSIONS`

```sql
SHOW PARAMETER processes;
SHOW PARAMETER sessions;
```

`sessions` is derived from `processes`. If you change `processes`, review `sessions` and `transactions`.

---

## 10. Wrap-Up

```sql
EXIT;
```

You now have a readable baseline for the instance's core configuration, which is the minimum required knowledge before you start changing anything.
