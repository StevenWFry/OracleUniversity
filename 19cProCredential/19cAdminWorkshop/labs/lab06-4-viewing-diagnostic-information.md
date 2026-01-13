Lab 6-4 - Viewing Diagnostic Information (ADR, Alert Logs, and DDL Logging)
===========================================================================

And look, when Oracle says, "Check the alert log," what it really means is,
"Go spelunking in a filing cabinet of XML and trace files until you find
something that looks incriminating."

This lab is about making that slightly less awful.

You will:

- Locate the ADR directories from inside the database.
- View the alert log as both XML and plain text.
- Use `adrci` to search the alert log like a grown-up.
- Enable DDL logging and prove that your DDL really is being recorded.

Prerequisites
-------------

- Logged in as the `oracle` OS user.
- Environment set for the `orclcdb` instance.
- You know how to connect with `sqlplus` as SYSDBA.

Part A - Locate ADR Directories with `V$DIAG_INFO`
--------------------------------------------------

1. Open a terminal and connect as SYSDBA:

   ```bash
   . oraenv
   ORACLE_SID = [orclcdb] ? orclcdb

   sqlplus / as sysdba
   ```

2. Ask Oracle where it hides diagnostic files:

   ```sql
   SELECT name, value
   FROM   v$diag_info;
   ```

   Pay attention to at least these rows:

   - `Diag Alert`  -> directory containing the XML alert log (`log.xml`)
   - `Diag Trace`  -> directory containing the text alert log and trace files

3. Note the full paths for those two entries; you will cd into them shortly.

4. Exit SQL*Plus:

   ```sql
   EXIT
   ```

Part B - View the XML Alert Log
-------------------------------

5. Change to the *alert* directory reported for `Diag Alert`:

   ```bash
   cd /u01/app/oracle/diag/rdbms/orclcdb/orclcdb/alert
   ls
   ```

   You should see:

   ```text
   log.xml
   ```

6. View the XML alert log with your preferred tool:

   ```bash
   more log.xml
   ```

   or, if you like GUIs:

   ```bash
   gedit log.xml &
   ```

   As you scroll you will see startup/shutdown messages, DDL events, and errors
   in XML form.

Part C - View the Text Alert Log
--------------------------------

7. Change to the *trace* directory reported for `Diag Trace`:

   ```bash
   cd /u01/app/oracle/diag/rdbms/orclcdb/orclcdb/trace
   ls alert*.log
   ```

   You should see something like:

   ```text
   alert_orclcdb.log
   ```

8. Look at the last few lines:

   ```bash
   tail alert_orclcdb.log
   ```

   This is the plain-text version of the alert log, showing recent database
   activity and errors.

9. Optionally, list all trace files:

   ```bash
   ls *.trc
   ```

   Each server and background process can write its own trace file when it
   detects internal errors.

Part D - Use ADRCI to View and Search the Alert Log
---------------------------------------------------

10. Start the ADR command interpreter:

    ```bash
    adrci
    ```

    If `ORACLE_HOME/bin` is not on your `PATH`, you can set it with:

    ```bash
    export PATH=$PATH:$ORACLE_HOME/bin
    adrci
    ```

11. At the `adrci>` prompt, show the alert log:

    ```text
    adrci> show alert
    ```

    - Use `G` (uppercase) to jump to the bottom.
    - You should see the same last entries you saw in the text alert log.

12. Search from the bottom for the most recent startup:

    ```text
    /starting Oracle
    ```

    Then press `N` (uppercase) to search backwards for the previous match.

13. Search for `ALTER` commands:

    ```text
    /alter
    ```

    Press `N` repeatedly to jump through successive `ALTER DATABASE` entries
    (for example, `MOUNT`, `OPEN`).

14. When you are finished:

    ```text
    :q
    adrci> exit
    ```

Part E - Enable DDL Logging and Generate Some DDL
-------------------------------------------------

15. Start SQL*Plus again as SYSDBA:

    ```bash
    sqlplus / as sysdba
    ```

16. Switch your session to a PDB (as in the practice, `orclpdb1`):

    ```sql
    ALTER SESSION SET container = orclpdb1;
    ```

17. Check whether DDL logging is enabled:

    ```sql
    SHOW PARAMETER enable_ddl_logging
    ```

    - On Oracle Cloud services this may default to `TRUE`.
    - In the course VM it is normally `FALSE`.

18. If it is `FALSE`, enable it for **this session only**:

    ```sql
    ALTER SESSION SET enable_ddl_logging = TRUE;
    ```

19. Run a couple of DDL statements to give Oracle something to log:

    ```sql
    CREATE TABLE test_ddl_log (
      id  NUMBER,
      txt VARCHAR2(30)
    );

    DROP TABLE test_ddl_log;
    ```

20. Exit SQL*Plus:

    ```sql
    EXIT
    ```

Part F - Inspect the DDL Log Files
----------------------------------

21. Change to the `log/ddl` directory under your ADR home. For example:

    ```bash
    cd /u01/app/oracle/diag/rdbms/orclcdb/orclcdb/log/ddl
    ls
    ```

    You should see a file like:

    ```text
    ddl_orclcdb.log
    ```

22. View the text DDL log:

    ```bash
    more ddl_orclcdb.log
    ```

    Look for entries corresponding to:

    - `CREATE TABLE TEST_DDL_LOG`
    - `DROP TABLE TEST_DDL_LOG`

23. For completeness, inspect the XML version in the `log/ddl` directory:

    ```bash
    ls *.xml
    ```

    and, if you like:

    ```bash
    gedit ddl_orclcdb.xml &
    ```

Step 5 - Clean Up
-----------------

24. Close any remaining terminals or just leave them for future labs.

You have now:

- Located the ADR structure from inside the database.
- Viewed the alert log as XML and as plain text.
- Used `adrci` to search the alert log without manually scrolling for hours.
- Enabled DDL logging and confirmed that your DDL really does get recorded.

Which means next time someone says, "Nothing changed, it just broke," you can
smile, open the DDL log, and say, "Funny story about that..."

