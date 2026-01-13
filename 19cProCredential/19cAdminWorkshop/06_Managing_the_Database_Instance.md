# 6 – Managing the Database Instance (Taming the Parameter Hydra)

And look, creating a database is fun in a terrifying, ritual‑summoning kind of way. But once it exists, your day‑to‑day job is managing the **instance** that runs it: its parameters, its diagnostics, and its performance views. This is where you stop being a database midwife and start being a database doctor.

This chapter covers:

- Parameter files: `spfile` vs `pfile`, how the instance uses them
- Changing parameters with `ALTER SYSTEM` and `ALTER SESSION`
- Dynamic vs static parameters and the `SCOPE` clause
- The Automatic Diagnostic Repository (ADR) layout and ADRCI
- Performance views (`V$`, `GV$`) and data dictionary views (`USER_`, `DBA_`, `CDB_`)

---

## 1. Parameter Files: Who Tells the Instance What To Do

When you start an instance, it needs instructions. Those instructions live in **initialization parameter files**:

- **spfile** – server parameter file, binary, database‑managed
  - Typically named `spfile<SID>.ora`
  - Lives in `$ORACLE_HOME/dbs` on Linux/UNIX
- **pfile** – plain text parameter file
  - Typically named `init<SID>.ora`
  - Also lives in `$ORACLE_HOME/dbs`

DBCA usually:

- Starts from a stock `init.ora` template
- Generates an `init<SID>.ora`/`spfile<SID>.ora` with the values you picked in the wizard

### 1.1 Required vs optional parameters

Technically, Oracle only *requires* one parameter in the initialization file:

```ini
db_name = orcl
```

Everything else can default. In practice:

- You will set many more: memory, file locations, compatibility, etc.
- Otherwise you are doing all your tuning from scratch on every new instance.

### 1.2 How the instance uses parameter files

On startup:

1. Oracle locates the parameter file (`spfile` first, then `pfile` if needed)
2. Reads the parameters
3. Stores them in **internal tables** in the SGA
4. Runs with those values until shutdown

To see what the instance is actually using, you query:

```sql
SELECT name, value
FROM   v$parameter;
```

Not the text file on disk.

---

## 2. Changing Parameters: `ALTER SYSTEM` and `ALTER SESSION`

You can change many parameters **while the database is running**.

### 2.1 `ALTER SYSTEM`

Changes the **instance‑level** value for all sessions (or for all sessions in a CDB container, depending on how you connect).

Basic form:

```sql
ALTER SYSTEM SET parameter_name = value [SCOPE = {MEMORY | SPFILE | BOTH}];
```

Examples:

```sql
ALTER SYSTEM SET open_cursors = 1000 SCOPE=BOTH;
ALTER SYSTEM SET db_recovery_file_dest_size = 50G SCOPE=SPFILE;
```

### 2.2 `ALTER SESSION`

Changes the value only for **your** session:

```sql
ALTER SESSION SET sql_trace = TRUE;
ALTER SESSION SET nls_date_format = 'DD-MON-YYYY HH24:MI:SS';
```

Useful for:

- Tracing a problematic session
- Tweaking NLS settings without affecting everyone else

### 2.3 Dynamic vs static parameters

Not all parameters are created equal:

- **Dynamic parameters** can be changed while the instance is running
  - Examples: `open_cursors`, some memory settings, tracing/logging settings
- **Static parameters** require a restart
  - Examples: `control_files`, `db_name`, often some memory structures

For static parameters:

1. Change them in the spfile:

   ```sql
   ALTER SYSTEM SET control_files =
     '/u02/oradata/ORCL/control01.ctl',
     '/u03/oradata/ORCL/control02.ctl'
   SCOPE=SPFILE;
   ```

2. Shut down the instance
3. Move/copy the files as needed
4. Start the instance again

### 2.4 The `SCOPE` clause

`SCOPE` tells Oracle *where* to apply the change:

- `SCOPE=MEMORY`
  - Change takes effect in the **running instance only**
  - Does **not** persist across restarts
- `SCOPE=SPFILE`
  - Change is written to the **spfile only**
  - Takes effect **next time** you start the instance
- `SCOPE=BOTH`
  - Change is applied immediately **and** persisted in the spfile

Rules:

- `MEMORY` and `BOTH` only make sense for **dynamic** parameters
- For **static** parameters, you must use `SCOPE=SPFILE` and restart

---

## 3. `SHOW PARAMETER` and `V$PARAMETER`

SQL*Plus provides a convenient shortcut:

```sql
SHOW PARAMETER db_name;
SHOW PARAMETER file;
```

Behind the scenes this is just a `SELECT` from `V$PARAMETER` with some formatting.

If you want more control, query it directly:

```sql
SELECT name,
       value,
       isses_modifiable,
       issys_modifiable
FROM   v$parameter
WHERE  name LIKE 'db_%'
ORDER  BY name;
```

In a multitenant environment, remember there are **CDB‑level** and **PDB‑level** parameter views as well; the lab later goes deeper into that.

---

## 4. The Automatic Diagnostic Repository (ADR)

When something goes wrong, you will inevitably be told to “check the alert log.” In 19c, the alert log and its friends live in the **Automatic Diagnostic Repository** (ADR).

ADR is:

- A directory structure under `ORACLE_BASE`
- With subdirectories based on product type and instance name

General layout:

```text
$ORACLE_BASE/diag/rdbms/<db_name>/<instance_name>/
  alert/
    log.xml        -- XML alert log
  cdump/           -- core dumps
  incident/        -- incident data per ORA error
  hm/              -- Health Monitor reports
  trace/
    alert_<inst>.log   -- text alert log
    *.trc, *.trm       -- background & session trace files
  ...
```

There is also:

```text
$ORACLE_BASE/diag/.../log/
  ddl/             -- DDL logging (if enabled)
  debug/           -- debug logging (if enabled)
```

Alerts capture:

- Startup/shutdown events
- Major structural changes (tablespace create/drop, file offline/online)
- Significant ORA‑ errors

Trace files capture:

- Session‑level tracing information
- Execution plans
- Wait events / timing data

---

## 5. ADRCI – Command‑Line Access to ADR

`ADRCI` (Automatic Diagnostic Repository Command Interface) is your CLI into ADR:

```bash
adrci
```

From the ADRCI prompt you can:

- See the current ADR home(s)
- View alert logs:

  ```text
  adrci> show alert
  ```

- Inspect incidents and problems:

  ```text
  adrci> show problem
  adrci> show incident
  ```

- Package diagnostic data for Oracle Support (SRs)

The earlier chapter showed a simple example:

- `SHOW ALERT`
- Navigating to a specific entry to see startup/shutdown messages and errors

---

## 6. Logging: DDL and Debug Logs

Besides alerts and trace files, you can enable **logging** for:

- DDL statements
- Debug/diagnostic output at various levels

When enabled, they write to:

```text
.../diag/.../log/ddl/
.../diag/.../log/debug/
```

Common use cases:

- You are doing a tuning session with many DDL changes:
  - Enable DDL logging
  - Later mine the log file for the exact sequence of DDL to replay or clean up
- You are reproducing a tricky ORA error for Oracle Support:
  - Enable debug logging at a higher level
  - Capture extra detail for analysis

And yes, it is entirely possible to turn on so much logging that you fill the filesystem, so use it like hot sauce: sparingly and intentionally.

---

## 7. Performance Views: `V$` and `GV$`

Oracle maintains a vast collection of **performance views** to tell you what the instance is doing. Under the hood:

- There is an internal “fixed tables” schema (`X$` tables)
- On startup, these are created and populated in memory
- When the instance shuts down, many are dropped; some information is pushed to persistent tables

You usually do **not** query `X$` directly (that’s where dragons live). Instead you use:

- `V$` views – single‑instance perspective
- `GV$` views – global (cluster‑wide) view in RAC

Examples:

- `V$DATABASE`

  ```sql
  DESC v$database;

  SELECT name,
         cdb,
         open_mode
  FROM   v$database;
  ```

  Shows:

  - Database name
  - Whether it is a CDB
  - Current open mode
  - Control file locations, logging configuration, and more

- `V$SESSION`

  ```sql
  DESC v$session;
  ```

  Holds:

  - One row per session
  - Session state, wait events, SQL being executed, module names, etc.

These are the raw materials for almost every performance and monitoring tool you will use.

---

## 8. Data Dictionary Views: USER_, ALL_, DBA_, CDB_

Beyond performance, you also need metadata: tables, indexes, users, privileges, etc. That’s the **data dictionary**.

View families:

- `USER_...`
  - Objects owned by *you*
  - Example: `USER_TABLES` – tables you own
- `ALL_...`
  - Objects you can *see* (because you own them or have privileges)
  - Example: `ALL_TABLES`
- `DBA_...`
  - All objects in the current container, for DBAs
  - Example: `DBA_TABLESPACES`, `DBA_TABLES`
- `CDB_...` (CDB only)
  - All objects across all containers (CDB root + PDBs)
  - Includes a `CON_ID` column to identify the container

Examples from the demo:

- Tablespaces in the current container:

  ```sql
  DESC dba_tablespaces;

  SELECT tablespace_name
  FROM   dba_tablespaces
  ORDER  BY tablespace_name;
  ```

- Tables across all containers:

  ```sql
  SELECT tablespace_name,
         con_id
  FROM   cdb_tablespaces
  ORDER  BY con_id, tablespace_name;
  ```

Note:

- Using `DBA_` views and trying to reference `CON_ID` fails, because that column only exists in the `CDB_` views.

---

## 9. A Few Handy CLI Tricks (SQL*Plus)

The demo also showed some practical SQL*Plus techniques:

- Setting the default editor:

  ```sql
  DEFINE _EDITOR = gedit
  ```

  Then:

  ```sql
  ED
  ```

  Opens your last command in the editor, you tweak it, save, and then:

  ```sql
  /
  ```

  Re‑executes the edited command.

- Searching for parameters:

  ```sql
  SHOW PARAMETER file;
  ```

  Returns all parameters whose names contain the string `file` – such as `db_recovery_file_dest` and its size.

These do not change the database, but they do change how quickly you can find out what is going on.

---

## 10. Summary – Keeping the Instance on a Short Leash

By now you should be able to:

- Explain the difference between `spfile` and `pfile`, and when each is used
- Use `ALTER SYSTEM` and `ALTER SESSION` to change parameters safely
- Understand dynamic vs static parameters and use the `SCOPE` clause correctly
- Navigate the ADR directory structure and know where:
  - Alert logs
  - Trace files
  - Incident and Health Monitor reports
  - DDL/debug logs
  live
- Use `ADRCI` to inspect alerts and incidents
- Query `V$` / `GV$` performance views to see what the instance is doing
- Use data dictionary views (`USER_`, `ALL_`, `DBA_`, `CDB_`) to inspect metadata

In short, you now have enough insight to stop treating the instance as a mysterious black box and start treating it like a system you can interrogate, tune, and, when necessary, gently shame into behaving better.

---

## Lab 6 Links

When you are ready to practice this in the lab environment, use:

- [Lab 6-1 - Basic Initialization Parameters with SQL*Plus](labs/lab06-1-view-parameters-with-sqlplus.md)
- [Lab 6-2 Part 1 - Viewing Parameters with SQL*Plus](labs/lab06-2-part-1-view-parameters-with-sqlplus.md)
- [Lab 6-2 Part 2 - Initialization Parameters and the SPFILE/PFILE Search Game](labs/lab06-2-part-2-initialization-parameters-and-spfile-search.md)


- [Lab 6-3 - Modifying Initialization Parameters with SQL*Plus](labs/lab06-3-initialization-parameters-and-spfile-search.md)
- [Lab 6-4 - Viewing Diagnostic Information (ADR, Alert Logs, and DDL Logging)](labs/lab06-4-viewing-diagnostic-information.md)
