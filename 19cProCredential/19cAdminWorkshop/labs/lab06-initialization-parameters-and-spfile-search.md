# Lab 6 – Initialization Parameters and the SPFILE/PFILE Search Game

And look, the instance does not start up because of “vibes.” It starts because it finds a parameter file and does what it is told. In this lab you will watch Oracle prove just how literal it is about that.

You will:

- See where the default server parameter file (SPFILE) lives
- Inspect the stock `init.ora` sample pfile
- Create a text pfile for `ORCLCDB` from the spfile
- Prove that Oracle falls back to the pfile when the spfile is “lost”
- Restore the spfile and confirm that startup behaviour changes back

---

## 1. Assumptions and Setup

- You are logged in as the `oracle` OS user.
- The CDB is named `ORCLCDB`.
- The CDB and listener are already running (started in earlier practices), or can be started with the provided `dbstart.sh` script if needed.

1. Open a terminal.
2. Set the environment with `oraenv`:

   ```bash
   . oraenv
   # When prompted, enter:
   ORACLE_SID = [ORCLCDB] ? ORCLCDB
   ```

3. Connect to the CDB root as SYSDBA:

   ```bash
   sqlplus / as sysdba
   ```

---

## 2. Locate the Default SPFILE

First, confirm which parameter file Oracle thinks it is using.

1. From SQL*Plus, show the `spfile` parameter:

   ```sql
   SHOW PARAMETER spfile;
   ```

   You should see something like:

   ```text
   NAME    TYPE   VALUE
   ------- ------ ------------------------------------------------
   spfile  string /u01/app/oracle/product/19.3.0/dbhome_1/dbs/spfileorclcdb.ora
   ```

   This tells you:

   - There **is** a spfile
   - It lives in `$ORACLE_HOME/dbs` as `spfileorclcdb.ora`

---

## 3. Inspect the Sample `init.ora` Text File

Now you will look at the generic sample pfile that ships with 19c.

1. Drop to the OS shell from SQL*Plus:

   ```sql
   HOST
   ```

   (In SQL*Plus, you can also use `!` as shorthand for `HOST`.)

2. Change to the parameter directory and list its contents:

   ```bash
   cd $ORACLE_HOME/dbs
   ls
   ```

   You should see:

   - `spfileorclcdb.ora`
   - `init.ora` (the sample text file)
   - Possibly spfiles/pfiles for other databases

3. Display the sample `init.ora`:

   ```bash
   cat init.ora
   ```

   This is the stock template DBCA copies from when it builds new instances.

4. Return to SQL*Plus:

   ```bash
   EXIT
   ```

   Then reconnect:

   ```bash
   sqlplus / as sysdba
   ```

---

## 4. Create a Text PFILE for `ORCLCDB` from the SPFILE

The lab now has you create a text pfile that mirrors the current spfile.

1. From SQL*Plus:

   ```sql
   CREATE PFILE = '$ORACLE_HOME/dbs/initorclcdb.ora'
   FROM SPFILE;
   ```

   (Adjust the exact filename to match the case used in your lab; the transcript used `initORCLCDB.ora`.)

   You should see:

   ```text
   File created.
   ```

2. Drop to the shell again:

   ```sql
   HOST
   ```

3. Confirm the new pfile exists:

   ```bash
   cd $ORACLE_HOME/dbs
   ls init*orclcdb*.ora
   cat initORCLCDB.ora
   ```

   You should see the full list of parameters that currently drive `ORCLCDB`.

4. Return to SQL*Plus:

   ```bash
   EXIT
   sqlplus / as sysdba
   ```

---

## 5. Prove That Oracle Falls Back to the PFILE

Now you will “lose” the spfile so that Oracle must use the pfile instead.

1. Shut down the database instance cleanly:

   ```sql
   SHUTDOWN IMMEDIATE;
   ```

   Wait for:

   ```text
   ORACLE instance shut down.
   ```

2. Drop to the shell:

   ```sql
   HOST
   ```

3. Rename the spfile so Oracle cannot find it:

   ```bash
   cd $ORACLE_HOME/dbs
   mv spfileorclcdb.ora orig_spfileorclcdb.ora
   ls spfileorclcdb*
   ```

   You should now **not** see `spfileorclcdb.ora` – only the renamed file.

4. Return to SQL*Plus:

   ```bash
   EXIT
   sqlplus / as sysdba
   ```

5. Start the database:

   ```sql
   STARTUP;
   ```

   The instance should start successfully. Since there is no spfile, Oracle will search for a pfile and use `initORCLCDB.ora` instead.

6. Confirm which parameter file Oracle thinks it is using:

   ```sql
   SHOW PARAMETER spfile;
   ```

   Expected:

   ```text
   NAME    TYPE   VALUE
   ------- ------ -----
   spfile  string
   ```

   An empty `VALUE` means the instance was started from a **pfile**, not an spfile.

---

## 6. Restore the SPFILE as the Startup Source

Now you will restore the spfile so the instance goes back to using it by default.

1. Shut down the database again:

   ```sql
   SHUTDOWN IMMEDIATE;
   ```

2. Drop to the shell:

   ```sql
   HOST
   ```

3. Rename the spfile back to its original name:

   ```bash
   cd $ORACLE_HOME/dbs
   mv orig_spfileorclcdb.ora spfileorclcdb.ora
   ls spfileorclcdb.ora
   ```

4. Return to SQL*Plus and start the database:

   ```bash
   EXIT
   sqlplus / as sysdba

   SQL> STARTUP;
   ```

5. Confirm the instance is now using the spfile again:

   ```sql
   SHOW PARAMETER spfile;
   ```

   You should once again see a path to `spfileorclcdb.ora` in `$ORACLE_HOME/dbs`.

6. Exit SQL*Plus:

   ```sql
   EXIT;
   ```

---

## 7. What You Just Learned the Hard Way

By the end of this lab you:

- Located the spfile for `ORCLCDB` and inspected the stock `init.ora` sample
- Created a text pfile for `ORCLCDB` from the existing spfile
- Shut down the instance and **temporarily removed** the spfile
- Verified that Oracle fell back to the pfile (`SHOW PARAMETER spfile` returned an empty value)
- Restored the spfile and confirmed that the instance once again used it on startup

In other words, you now have first‑hand evidence that Oracle’s parameter search logic does exactly what the documentation promises—no more, no less. Which is comforting, because “mystery behaviour at startup” is not a phrase anyone wants to hear in production.

