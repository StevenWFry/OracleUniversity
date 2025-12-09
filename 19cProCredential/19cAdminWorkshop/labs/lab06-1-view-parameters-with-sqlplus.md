# Lab 6‑2 – Viewing Initialization Parameters with SQL*Plus

And look, initialization parameters are how the instance tells you what kind of day it’s having. In this lab you’ll use nothing more exotic than `SHOW PARAMETER` to see how `ORCLCDB` is configured and what those values actually mean.

You will:

- Set your environment to `ORCLCDB`
- Connect as `SYSDBA`
- Inspect core parameters with `SHOW PARAMETER`
- Relate each parameter to what it does for the instance

---

## 1. Assumptions and Setup

- You are logged in as the `oracle` OS user.
- The CDB `ORCLCDB` is already started (from earlier practices) or can be started with `dbstart.sh` if required.

1. Open a terminal.
2. Set the environment with `oraenv`:

   ```bash
   . oraenv
   # When prompted:
   ORACLE_SID = [ORCLCDB] ? ORCLCDB
   ```

3. Connect to the CDB root as SYSDBA:

   ```bash
   sqlplus / as sysdba
   ```

---

## 2. Global Database Name: `DB_NAME` and `DB_DOMAIN`

The **global database name** is effectively:

```text
db_name.db_domain
```

1. Show the `db_name` parameter:

   ```sql
   SHOW PARAMETER db_name;
   ```

   You should see something like:

   ```text
   NAME     TYPE   VALUE
   -------- ------ -------
   db_name  string orclcdb
   ```

   Notes:

   - `db_name` is the **database identifier** (up to 8 characters historically, longer allowed in practice).
   - On a host with multiple databases, `db_name` typically matches the `ORACLE_SID` for that database to avoid confusion.

2. Show the `db_domain` parameter:

   ```sql
   SHOW PARAMETER db_domain;
   ```

   In the lab, this is usually **empty**, meaning the database has no domain component:

   ```text
   NAME       TYPE   VALUE
   ---------- ------ -----
   db_domain  string
   ```

   In a distributed environment you would set `db_domain` so that `db_name.db_domain` is unique across the network.

---

## 3. Fast Recovery Area: `DB_RECOVERY_FILE_DEST` and Size

The **fast recovery area (FRA)** is where Oracle stores:

- Multiplexed control files
- Multiplexed online redo logs
- Archived redo logs
- Flashback logs
- RMAN backups

If you set `db_recovery_file_dest`, you must also set `db_recovery_file_dest_size`.

1. Show the FRA destination:

   ```sql
   SHOW PARAMETER db_recovery_file_dest;
   ```

2. Show the FRA size:

   ```sql
   SHOW PARAMETER db_recovery_file_dest_size;
   ```

   Typical output:

   ```text
   NAME                    TYPE   VALUE
   ----------------------- ------ -----------------------------------------
   db_recovery_file_dest   string /u01/app/oracle/fast_recovery_area
   db_recovery_file_dest_size big 14970M
   ```

   Notes:

   - `db_recovery_file_dest` points to the FRA root directory.
   - `db_recovery_file_dest_size` is a **hard limit** for total FRA space (in bytes; SQL*Plus formats it as MB/GB).

---

## 4. SGA Sizing: `SGA_TARGET` and `SGA_MAX_SIZE`

The **System Global Area (SGA)** holds shared memory structures. Two key parameters:

- `sga_target` – target size for the SGA
- `sga_max_size` – maximum SGA size

1. Show SGA‑related parameters:

   ```sql
   SHOW PARAMETER sga;
   ```

   In the lab you will see something like:

   ```text
   NAME         TYPE   VALUE
   ------------ ------ -------
   sga_max_size big   1920M
   sga_target   big   1920M
   ```

   Notes:

   - When `sga_target` is non‑zero, Automatic Shared Memory Management (ASMM) is enabled.
   - Oracle automatically distributes memory among buffer cache, shared pool, large pool, Java pool, Streams pool.
   - You can still set *minimum* sizes for some pools if an application needs them.

The log buffer, keep/recycle caches, and certain other structures are not part of ASMM and must be sized separately if you care.

---

## 5. Undo Configuration: `UNDO_TABLESPACE`

Undo records support:

- Transaction rollback
- Read consistency
- Some Flashback features

They live in the **undo tablespace**.

1. Show the undo tablespace parameter:

   ```sql
   SHOW PARAMETER undo_tablespace;
   ```

   Expected:

   ```text
   NAME            TYPE   VALUE
   --------------- ------ --------
   undo_tablespace string UNDOTBS1
   ```

   This tells you which undo tablespace is in use when the instance starts.

---

## 6. Compatibility: `COMPATIBLE`

`COMPATIBLE` controls which Oracle release features the database may use while guaranteeing a minimum ability to revert.

1. Show the `compatible` parameter:

   ```sql
   SHOW PARAMETER compatible;
   ```

   Typical output:

   ```text
   NAME        TYPE   VALUE
   ----------- ------ --------
   compatible  string 19.0.0
   ```

   Notes:

   - By default, `compatible` matches your installed database version.
   - For a CDB, this must be at least `12.0.0.0` to support multitenant features.

Changing this is **not** something you do casually.

---

## 7. Control Files: `CONTROL_FILES`

The `control_files` parameter lists one or more control files (up to 8), each with a full path.

Oracle strongly recommends multiplexing:

- Multiple control files on different disks or mount points

1. Show the parameter:

   ```sql
   SHOW PARAMETER control_files;
   ```

   Example of formatted output:

   ```text
   NAME          TYPE   VALUE
   ------------- ------ ----------------------------------------------
   control_files string /u01/app/oracle/oradata/ORCLCDB/control01.ctl,
                         /u01/app/oracle/oradata/ORCLCDB/control02.ctl
   ```

If you ever move control files, this parameter must be updated (it is static), and the instance must be restarted.

---

## 8. Workload Limits: `PROCESSES` and `SESSIONS`

These parameters define how many people and processes can be active at once.

1. Show `processes`:

   ```sql
   SHOW PARAMETER processes;
   ```

   Example:

   ```text
   NAME      TYPE   VALUE
   --------- ------ -----
   processes integer 300
   ```

   This is the maximum number of **OS user processes** that can connect simultaneously, including background processes.

2. Show `sessions`:

   ```sql
   SHOW PARAMETER sessions;
   ```

   Example:

   ```text
   NAME      TYPE   VALUE
   --------- ------ -----
   sessions  integer 472
   ```

   Notes:

   - `sessions` is derived from `processes` by default.
   - A rule of thumb: set `sessions` ≈ concurrent users + background processes + ~10% for recursive sessions.

If you increase `processes`, re‑evaluate whether `sessions` and `transactions` should be adjusted.

---

## 9. Wrap‑Up

1. Exit SQL*Plus:

   ```sql
   EXIT;
   ```

You have now:

- Used `SHOW PARAMETER` to inspect:
  - Global database name (`db_name`, `db_domain`)
  - Fast recovery area (`db_recovery_file_dest`, `db_recovery_file_dest_size`)
  - Memory (`sga_target`, `sga_max_size`)
  - Undo (`undo_tablespace`)
  - Compatibility (`compatible`)
  - Control file locations (`control_files`)
  - Workload sizing (`processes`, `sessions`)
- Tied each parameter to what it means for the running instance

In other words, you can now look at an instance and say more than “it’s up.” You can say *how* it’s configured, which is a key step on the road from “button pusher” to “actual DBA.”

