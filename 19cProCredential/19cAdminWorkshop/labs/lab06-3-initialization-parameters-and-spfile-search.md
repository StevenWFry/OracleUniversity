# Lab 6-3 - Modifying Initialization Parameters with SQL*Plus

And look, the only thing more dangerous than not knowing what your instance parameters are is changing them without understanding what they do. This lab is about doing the former so you are less tempted to do the latter.

This aligns to **Practice 6-3: Modifying Initialization Parameters by Using SQL*Plus**.

---

## 1. Assumptions

- You are logged in as the `oracle` OS user.
- The Oracle environment is set to `orclcdb`.

---

## 2. Session-Level Parameter: `NLS_DATE_FORMAT`

1. Connect as SYSDBA:

   ```bash
   sqlplus / as sysdba
   ```

2. Inspect the parameter:

   ```sql
   COLUMN name FORMAT A18
   COLUMN value FORMAT A20
   SELECT name, isses_modifiable, issys_modifiable, ispdb_modifiable, value
   FROM   v$parameter
   WHERE  name = 'nls_date_format';
   ```

3. Check the territory default:

   ```sql
   SELECT name, value FROM v$parameter WHERE name = 'nls_territory';
   ```

4. Switch to `ORCLPDB1` and view dates:

   ```sql
   ALTER SESSION SET container = ORCLPDB1;
   SELECT last_name, hire_date FROM hr.employees;
   ```

5. Change the format for this session:

   ```sql
   ALTER SESSION SET nls_date_format = 'mon dd yyyy';
   ```

6. Query again and confirm the format changed:

   ```sql
   SELECT last_name, hire_date FROM hr.employees;
   SHOW PARAMETER nls_date_format;
   ```

7. Disconnect:

   ```sql
   DISCONNECT
   ```

8. Reconnect with Easy Connect as `system` and verify the default is back:

   ```sql
   HOST hostname -f
   HOST lsnrctl status | grep tcp | grep PORT

   CONNECT / AS SYSDBA
   COL name FORMAT a20
   COL network_name FORMAT a20
   SELECT name, network_name FROM v$services WHERE name = 'orclpdb1';

   CONNECT system/cloud_4U@//<full hostname>:1521/orclpdb1
   SELECT last_name, hire_date FROM hr.employees;
   SHOW PARAMETER nls_date_format;
   ```

---

## 3. Dynamic System-Level Parameter: `JOB_QUEUE_PROCESSES`

1. Exit and reconnect as SYSDBA to the root container:

   ```sql
   EXIT
   ```

   ```bash
   sqlplus / as sysdba
   ```

2. Inspect the parameter:

   ```sql
   COLUMN name FORMAT A20
   COLUMN value FORMAT A20
   SELECT name, isses_modifiable, issys_modifiable, value
   FROM   v$parameter
   WHERE  name = 'job_queue_processes';
   ```

3. Set the value and persist it:

   ```sql
   ALTER SYSTEM SET job_queue_processes = 15 SCOPE=BOTH;
   ```

4. Verify with a partial search:

   ```sql
   SHOW PARAMETER job
   ```

5. Restart and verify the change persisted:

   ```sql
   SHUTDOWN IMMEDIATE;
   STARTUP;
   SHOW PARAMETER job
   ```

---

## 4. Static System-Level Parameter: `SEC_MAX_FAILED_LOGIN_ATTEMPTS`

1. Inspect the parameter:

   ```sql
   COLUMN name FORMAT a30
   SELECT name, isses_modifiable, issys_modifiable, value
   FROM   v$parameter
   WHERE  name = 'sec_max_failed_login_attempts';
   ```

2. Attempt to change it with `SCOPE=BOTH` (expect error):

   ```sql
   ALTER SYSTEM SET sec_max_failed_login_attempts = 2 SCOPE=BOTH;
   ```

3. Change it in the SPFILE with a comment:

   ```sql
   ALTER SYSTEM SET sec_max_failed_login_attempts = 2
     COMMENT = 'Reduce for tighter security.'
     SCOPE = SPFILE;
   ```

4. Confirm it is not yet active:

   ```sql
   SHOW PARAMETER sec_max
   ```

5. Restart and confirm the new value:

   ```sql
   SHUTDOWN IMMEDIATE;
   STARTUP;
   SHOW PARAMETER sec_max
   ```

6. Verify the comment:

   ```sql
   COLUMN name FORMAT a30
   COLUMN update_comment FORMAT a30
   SELECT name, update_comment
   FROM   v$parameter
   WHERE  name = 'sec_max_failed_login_attempts';
   ```

7. Reset the parameter to its original value:

   ```sql
   ALTER SYSTEM SET sec_max_failed_login_attempts = 3
     COMMENT = ''
     SCOPE = SPFILE;
   ```

8. Exit SQL*Plus:

   ```sql
   EXIT
   ```

---

## 5. Wrap-Up

You now have hands-on proof that:

- Session-level changes vanish when the session ends
- Dynamic system-level changes can be applied immediately and persisted
- Static parameters require SPFILE changes plus a restart
