Lab 6-2 - Viewing Advanced Parameters and Parameter Views with SQL*Plus
======================================================================

And look, at some point in your DBA career you will stare at an Oracle
parameter name like `commit_logging` and think, "That sounds important
and I have absolutely no idea what it does."
This lab is about turning that feeling into something slightly less horrifying.

In Lab 6-1 you played the SPFILE/PFILE shell game and looked at the basic,
actually-useful parameters.
In this lab you poke at the so-called *advanced* parameters and the parameter
views Oracle hides all the fun in.

You will:

- Use `SHOW PARAMETER` on advanced parameters you rarely change on purpose.
- Explore `V$PARAMETER`, `V$SPPARAMETER`, `V$PARAMETER2`, and `V$SYSTEM_PARAMETER`.
- Learn how to tell which parameters are static vs dynamic without guessing.

Prerequisites
-------------

- Logged in to the VM as the `oracle` OS user.
- `ORCLCDB` container database is up and running.
- You know how to use `oraenv` and `sqlplus` (from Lab 6-1).

Step 1 - Set the environment and connect
----------------------------------------

1. Open a terminal and point the shell at your CDB:

   ```bash
   . oraenv
   ORACLE_SID = [orclcdb] ? orclcdb
   ```

2. Connect as SYS with SYSDBA:

   ```bash
   sqlplus / as sysdba
   ```

Step 2 - Inspect "advanced" parameters with SHOW PARAMETER
----------------------------------------------------------

These are the knobs Oracle does not expect you to touch often.
Which is exactly why we are going to look at them.

### 2.1 `transactions`

1. Show the transactions-related parameters:

   ```sql
   show parameter transactions
   ```

   You should see something like:

   ```text
   NAME                           TYPE    VALUE
   ------------------------------ ------- -----
   transactions                   integer 519
   transactions_per_rollback_segment integer 5
   ```

   Notes:

   - `transactions` is the maximum number of concurrent transactions.
   - `transactions_per_rollback_segment` (advanced, almost never changed)
     controls how many concurrent transactions a single rollback segment can
     host when `undo_management=MANUAL`.
   - With automatic undo (`undo_management=AUTO`), Oracle sizes this for you.

### 2.2 `db_files`

2. Show the configuration for the maximum number of datafiles:

   ```sql
   show parameter db_files
   ```

   - `db_files` is the upper limit on open database files.
   - Your value will look like `200` or similar and is bounded by the OS.

### 2.3 Commit-related parameters

3. Check how redo is batched:

   ```sql
   show parameter commit_logging
   ```

   - This controls how the log writer (LGWR) batches redo.
   - No value (blank) means "use the default behaviour."

4. Check when commit waits for redo flush:

   ```sql
   show parameter commit_wait
   ```

   - This controls when the redo for a command is flushed to the redo log.
   - Blank value again means defaults are in effect.

Both commit-related parameters can be set at the PDB level if you really
need to, but only after you understand the performance trade-offs.

### 2.4 Shared pool and block-related parameters

5. Show the shared pool size:

   ```sql
   show parameter shared_pool_size
   ```

   - `shared_pool_size` sets the size of the shared pool in bytes (if you are
     not letting `sga_target` auto-manage it).

6. Show the standard block size:

   ```sql
   show parameter db_block_size
   ```

   - This is the default block size, in bytes, for most tablespaces.
   - Set at database creation; cannot be changed afterward.

7. Show the default buffer cache size:

   ```sql
   show parameter db_cache_size
   ```

   - `db_cache_size` is the size of the default buffer cache.
   - With automatic shared memory management it may be `0` and derived from
     `sga_target`.

### 2.5 Undo and automatic memory management

8. Show how undo is managed:

   ```sql
   show parameter undo_management
   ```

   - `AUTO` means automatic undo management via an undo tablespace.
   - `MANUAL` means old-school rollback segments.

9. Show Automatic Memory Management (AMM) parameters:

   ```sql
   show parameter memory_target
   show parameter memory_max_target
   ```

   - `memory_target` is the target for combined SGA+PGA.
   - `memory_max_target` is the ceiling for `memory_target`.
   - In this lab environment both may be `0`, meaning AMM is not being used.

10. Show the PGA aggregate target:

    ```sql
    show parameter pga_aggregate_target
    ```

    - This is the target amount of PGA memory available to all server processes.
    - Default is the greater of 10 MB or 20% of the SGA.

Step 3 - Find all "parameter" views in the data dictionary
---------------------------------------------------------

Now that we have admired a bunch of individual knobs, let us see where Oracle
hides the metadata about them.

11. Make the output sane for long lists:

    ```sql
    set pagesize 100
    ```

12. List all dictionary views whose names contain the word `PARAMETER`:

    ```sql
    select table_name
    from   dictionary
    where  table_name like '%PARAMETER%';
    ```

    You should see about 66 rows, including:

    - `V$PARAMETER`
    - `V$SPPARAMETER`
    - `V$PARAMETER2`
    - `V$SYSTEM_PARAMETER`

Step 4 - Explore `V$PARAMETER`
------------------------------

13. Describe the view:

    ```sql
    desc v$parameter
    ```

    Pay particular attention to the `ISSYS_MODIFIABLE` column:

    - `FALSE`    -> static parameter (requires restart).
    - `IMMEDIATE` -> dynamic, can be changed at runtime.
    - `DEFERRED`  -> applies to new sessions.

14. Optionally, enable paging between screens:

    ```sql
    set pause on
    column name  format a35
    column value format a20
    ```

15. Query a subset of columns ordered by parameter name:

    ```sql
    select name,
           issys_modifiable,
           value
    from   v$parameter
    order  by name;
    ```

    - Expect 400+ rows.
    - Look for:
      - `transactions` with `ISSYS_MODIFIABLE = FALSE` (static).
      - `plsql_warnings` with `ISSYS_MODIFIABLE = IMMEDIATE` (dynamic).

16. Filter parameters to those that contain the string `pool`:

    ```sql
    select name, value
    from   v$parameter
    where  name like '%pool%';
    ```

    You should see parameters such as `shared_pool_size`, `java_pool_size`,
    `large_pool_size`, and friends.

17. When you are done paging:

    ```sql
    set pause off
    ```

Step 5 - Explore `V$SPPARAMETER`
--------------------------------

`V$SPPARAMETER` tells you what is in the server parameter file (SPFILE), as
opposed to what is currently in use.

18. Describe the view:

    ```sql
    desc v$spparameter
    ```

19. Query name and value:

    ```sql
    select name, value
    from   v$spparameter
    order  by name;
    ```

    - If the instance was not started with an SPFILE, the `ISSPECIFIED` column
      will be `FALSE` for all rows.

Step 6 - Explore `V$PARAMETER2`
-------------------------------

`V$PARAMETER2` is like `V$PARAMETER`, but multi-valued parameters (such as
`control_files`) get one row per value.

20. Describe the view:

    ```sql
    desc v$parameter2
    ```

21. Query name and value:

    ```sql
    select name, value
    from   v$parameter2
    order  by name, ordinal;
    ```

    - Expect around 450 rows.
    - For parameters with multiple values, inspect the different `ORDINAL`
      values to see each element.

Step 7 - Explore `V$SYSTEM_PARAMETER`
-------------------------------------

Finally, zoom out to instance-level defaults.

22. Describe the view:

    ```sql
    desc v$system_parameter
    ```

23. Query name and value:

    ```sql
    select name, value
    from   v$system_parameter
    order  by name;
    ```

    - These are the parameter values in effect for the instance as a whole.
    - New sessions inherit from here, unless you override things at the session
      or PDB level.

Step 8 - Clean exit
-------------------

24. When you are finished spelunking through parameters:

    ```sql
    exit
    ```

You have now:

- Looked at advanced instance parameters without immediately changing them.
- Learned where Oracle hides the metadata that tells you what is static
  versus dynamic.
- Practiced querying the `V$` and data dictionary views you will use
  constantly as a DBA.

Which means next time someone suggests "just set commit_logging to something cool,"
you can, very calmly, ask them *why*.

