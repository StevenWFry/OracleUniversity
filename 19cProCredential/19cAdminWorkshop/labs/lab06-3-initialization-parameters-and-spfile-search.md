Lab 6-3 - Modifying Initialization Parameters with SQL*Plus
===========================================================

And look, the only thing more dangerous than not knowing what your instance
parameters are is changing them without understanding what they do.
This lab is about doing the former so you are less tempted to do the latter.

In this practice you will:

- Change a **session-level** NLS parameter and watch it evaporate when you disconnect.
- Change a **dynamic system-level** parameter and verify it survives a restart.
- Change a **static system-level** parameter via the SPFILE and prove that a restart is required.

Prerequisites
-------------

- Logged in to the VM as the `oracle` OS user.
- `ORCLCDB` is up and running.
- You know how to use `oraenv` and `sqlplus` (from the earlier labs).

Step 1 - Connect as SYSDBA
--------------------------

1. Open a terminal and point the environment at `orclcdb`:

   ```bash
   . oraenv
   ORACLE_SID = [orclcdb] ? orclcdb
   ```

2. Start SQL*Plus and connect as SYS with SYSDBA:

   ```bash
   sqlplus / as sysdba
   ```

Part A - Session-Level Parameter: `NLS_DATE_FORMAT`
---------------------------------------------------

First up: a harmless session-level parameter that lets you prove a point about scope.

### A.1 Inspect `NLS_DATE_FORMAT` in `V$PARAMETER`

3. Ask the instance what it *could* do with `NLS_DATE_FORMAT`:

   ```sql
   SELECT name,
          isses_modifiable,
          issys_modifiable,
          ispdb_modifiable,
          value
   FROM   v$parameter
   WHERE  name = 'nls_date_format';
   ```

   You should see something like:

   ```text
   NAME             ISSES_MODIFIABLE ISSYS_MODIFIABLE ISPDB_MODIFIABLE VALUE
   ---------------- ---------------- ---------------- ---------------- -----
   nls_date_format  TRUE             FALSE            TRUE
   ```

   Translation:

   - Session-level: **modifiable**.
   - System-level: **not** directly modifiable.
   - PDB-level: modifiable.

### A.2 Check the default date format

4. Check the territory, which drives the default date format:

   ```sql
   SELECT name, value
   FROM   v$parameter
   WHERE  name = 'nls_territory';
   ```

   You should see `AMERICA` (or similar).

5. Move into a PDB and see the current display:

   ```sql
   ALTER SESSION SET container = orclpdb1;

   SELECT last_name, hire_date
   FROM   hr.employees;
   ```

   Dates should appear in the default `DD-MON-RR` format.

### A.3 Change `NLS_DATE_FORMAT` for your session

6. Change only **your** session:

   ```sql
   ALTER SESSION SET nls_date_format = 'MON DD YYYY';
   ```

7. Rerun the query:

   ```sql
   SELECT last_name, hire_date
   FROM   hr.employees;
   ```

   Hire dates should now be formatted as `MON DD YYYY`.

8. Confirm with `SHOW PARAMETER`:

   ```sql
   SHOW PARAMETER nls_date_format
   ```

   You should see:

   ```text
   NAME             TYPE   VALUE
   ---------------- ------ ------------
   nls_date_format  string MON DD YYYY
   ```

### A.4 Disconnect and reconnect with Easy Connect

9. Disconnect:

   ```sql
   DISCONNECT
   ```

10. From the SQL*Plus prompt, grab the full host name:

    ```sql
    HOST hostname -f
    ```

    Make note of the output; call it `full.host.name`.

11. Check the listener port:

    ```sql
    HOST lsnrctl status
    ```

    You should see the standard `1521` port.

12. Find the PDB service name:

    ```sql
    CONNECT / AS SYSDBA

    SELECT name
    FROM   v$services;
    ```

    Look for the service that corresponds to `orclpdb1` (for example, `orclpdb1` or similar).

13. Connect as the `SYSTEM` user to `orclpdb1` using Easy Connect and the course password:

    ```sql
    CONNECT system/cloud_4U@full.host.name:1521/orclpdb1
    ```

14. Query the dates again:

    ```sql
    SELECT last_name, hire_date
    FROM   hr.employees;
    ```

    Back to `DD-MON-RR`. New session, new default format.

15. Check the parameter one more time:

    ```sql
    SHOW PARAMETER nls_date_format
    ```

    The `VALUE` column should be empty again. Session-level changes do **not** persist.

Part B - Dynamic System-Level Parameter: `JOB_QUEUE_PROCESSES`
--------------------------------------------------------------

Now we turn a knob that actually sticks across restarts, because why not make things interesting.

16. Exit and reconnect as SYSDBA to the root container:

    ```sql
    EXIT
    ```

    ```bash
    sqlplus / as sysdba
    ```

### B.1 Inspect `JOB_QUEUE_PROCESSES`

17. Check how modifiable it is and what it is currently set to:

    ```sql
    COLUMN name  FORMAT a20
    COLUMN value FORMAT a20

    SELECT name,
           isses_modifiable,
           issys_modifiable,
           value
    FROM   v$parameter
    WHERE  name = 'job_queue_processes';
    ```

    You should see:

    ```text
    ISSES_MODIFIABLE = FALSE
    ISSYS_MODIFIABLE = IMMEDIATE
    VALUE            = 40
    ```

### B.2 Change and persist the value

18. Change it and persist to memory and SPFILE:

    ```sql
    ALTER SYSTEM SET job_queue_processes = 15 SCOPE=BOTH;
    ```

19. Confirm the new value:

    ```sql
    SHOW PARAMETER job_queue_processes
    ```

### B.3 Restart to prove it stuck

20. Restart the instance:

    ```sql
    SHUTDOWN IMMEDIATE;
    STARTUP;
    ```

21. Confirm again:

    ```sql
    SHOW PARAMETER job_queue_processes
    ```

    Still `15`. The value was written into the SPFILE and survives restarts.

Part C - Static System-Level Parameter: `SEC_MAX_FAILED_LOGIN_ATTEMPTS`
-----------------------------------------------------------------------

Finally, a parameter that refuses to be changed at runtime and forces you to go through the SPFILE.

### C.1 Inspect the parameter

22. See how modifiable it is and what it is set to:

    ```sql
    SELECT name,
           isses_modifiable,
           issys_modifiable,
           value,
           update_comment
    FROM   v$parameter
    WHERE  name = 'sec_max_failed_login_attempts';
    ```

    You should see:

    ```text
    VALUE = 3
    ISSYS_MODIFIABLE = FALSE
    ```

### C.2 Try (and fail) to change it live

23. Attempt to change it with `SCOPE=BOTH`:

    ```sql
    ALTER SYSTEM SET sec_max_failed_login_attempts = 2
      SCOPE=BOTH;
    ```

    Expect an error along the lines of:

    ```text
    ORA-02095: specified initialization parameter cannot be modified
    ```

    This is Oracle politely saying, "No, you may not do brain surgery while the patient is awake."

### C.3 Change it in the SPFILE only

24. Change it in the SPFILE with an explanatory comment:

    ```sql
    ALTER SYSTEM SET sec_max_failed_login_attempts = 2
      COMMENT = 'Reduced for tighter security'
      SCOPE=SPFILE;
    ```

25. Show the parameter again:

    ```sql
    SHOW PARAMETER sec_max_failed_login_attempts
    ```

    It will still show `3` in the running instance, because you have not restarted yet.

### C.4 Restart to pick up the new value

26. Restart the instance:

    ```sql
    SHUTDOWN IMMEDIATE;
    STARTUP;
    ```

27. Confirm the new value and comment:

    ```sql
    SELECT name,
           value,
           update_comment
    FROM   v$parameter
    WHERE  name = 'sec_max_failed_login_attempts';
    ```

    You should see `2` and the comment `Reduced for tighter security`.

### C.5 Put it back (optional but polite)

28. Reset the parameter to its original value of `3` in the SPFILE:

    ```sql
    ALTER SYSTEM SET sec_max_failed_login_attempts = 3
      COMMENT = 'Reset to course default'
      SCOPE=SPFILE;
    ```

29. Optionally restart again so the running instance matches the file:

    ```sql
    SHUTDOWN IMMEDIATE;
    STARTUP;
    ```

Step 4 - Exit SQL*Plus
----------------------

30. When you are done:

    ```sql
    EXIT
    ```

You have now:

- Modified a **session-level** parameter and watched it vanish when you disconnected.
- Modified a **dynamic system-level** parameter that persisted across restarts.
- Modified a **static system-level** parameter correctly via the SPFILE and a restart.

Which puts you firmly ahead of anyone whose default instinct is still `STARTUP FORCE`.

