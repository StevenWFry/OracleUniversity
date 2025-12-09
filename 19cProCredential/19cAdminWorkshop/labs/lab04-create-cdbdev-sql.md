# Lab 4 – Creating `CDBDEV` with Raw SQL (No DBCA Safety Net)

And look, this is the part of the course where we politely take DBCA away and see whether you can still build a database without panicking. The good news is that the process is entirely predictable once you know the steps.

In this lab you will create a **container database** called `CDBDEV` **using the `CREATE DATABASE` SQL command**, then verify that Oracle really built what you asked for.

---

## 1. Lab Goal and Characteristics

You will create a new CDB named `CDBDEV` with these characteristics:

- Same passwords for `SYS` and `SYSTEM` as in the base CDB:

  ```text
  cloud_4U
  ```

- Oracle Managed Files (OMF) for **datafiles** and **redo logs**:
  - `DB_CREATE_FILE_DEST` → `/u01/app/oracle/oradata`
- CDB root contains:
  - Default TEMP tablespace: `TEMP`
  - Default permanent tablespace: `USERS`
  - Undo tablespace: `UNDOTBS`
- EM Express HTTPS port target: `5501`
  - Note: manual `CREATE DATABASE` does **not** set this automatically; you will see that at the end.
- The CDB is created with **no PDBs** except the **PDB seed**.

Assumptions:

- You are `oracle` on the lab VM.
- ORACLE_HOME and base CDB are already installed and working.

---

## 2. Make Sure `CDBDEV` Does Not Already Exist

1. Open a terminal (right‑click desktop → **Open Terminal**).
2. Check the data directory to make sure there is no leftover `CDBDEV` from earlier runs:

   ```bash
   ls /u01/app/oracle/oradata
   ```

   You should **not** see a `CDBDEV` directory. If you do, follow your lab guide’s cleanup instructions before continuing.

---

## 3. Prepare the `CREATE DATABASE` Script

The lab gives you a template SQL script with a full `CREATE DATABASE` command. You just need to plug in the correct passwords.

1. Change to the directory that contains the script:

   ```bash
   cd /home/oracle/labs/DBMod_CreateDB
   ls
   ```

   Look for the file:

   ```text
   CrCDBDEV.sql
   ```

2. Open it in an editor (the guide uses `gedit`):

   ```bash
   gedit CrCDBDEV.sql &
   ```

3. In the script, find **all** password placeholders (anything that literally says `password` or an obvious placeholder) for:

   - `SYS`
   - `SYSTEM`
   - PDB admin, if present

4. Replace each placeholder with the lab password:

   ```text
   cloud_4U
   ```

   For example, the clauses should end up looking like:

   ```sql
   USER SYS IDENTIFIED BY "cloud_4U"
   USER SYSTEM IDENTIFIED BY "cloud_4U"
   ```

5. Save the file and close the editor.

---

## 4. Create the Initialization Parameter File `initCDBDEV.ora`

Next you will prepare a pfile for the new instance by copying the sample `init.ora` and editing it.

1. Set the SID for the new database:

   ```bash
   export ORACLE_SID=CDBDEV
   ```

2. Change into the parameter directory:

   ```bash
   cd $ORACLE_HOME/dbs
   ls
   ```

   Confirm that a generic `init.ora` exists.

3. Copy the generic init file to a new pfile for `CDBDEV`:

   ```bash
   cp init.ora initCDBDEV.ora
   ls initCDBDEV.ora
   ```

4. Edit `initCDBDEV.ora`:

   ```bash
   gedit initCDBDEV.ora &
   ```

5. **Add** the following parameters (you can put them near the top or bottom):

   ```ini
   db_create_file_dest = '/u01/app/oracle/oradata'
   enable_pluggable_database = true
   ```

6. **Change** the following parameters to match the lab:

   - Database name:

     ```ini
     db_name = CDBDEV
     ```

   - Audit file destination:

     ```ini
     audit_file_dest = '/u01/app/oracle/admin/CDBDEV/adump'
     ```

   - Recovery area:

     ```ini
     db_recovery_file_dest = '/u01/app/oracle/fast_recovery_area'
     ```

   - Diagnostic destination:

     ```ini
     diagnostic_dest = '/u01/app/oracle'
     ```

   - Dispatchers (service name updated for this DB):

     ```ini
     dispatchers = '(PROTOCOL=TCP)(SERVICE=CDBDEVXDB)'
     ```

   - Control file names (copy the template from the lab book, then adjust):

     ```ini
     control_files =
       '/u01/app/oracle/oradata/CDBDEV/control01.ctl',
       '/u01/app/oracle/oradata/CDBDEV/control02.ctl'
     ```

   - `compatible` to match 19c:

     ```ini
     compatible = '19.0.0.0.0'
     ```

7. Follow the note from the lab:

   - If you see any `ORACLE_BASE` placeholders, replace them with the actual path you are using.

8. Do a quick scan:

   - Make sure there are no stray characters from editing (for example partial words/typos).
   - Confirm `db_domain` is either set to your desired domain or empty, as per the template.

9. Save `initCDBDEV.ora` and close the editor.

---

## 5. Create Required Directories

Now make sure the directories referenced in the pfile actually exist.

Run:

```bash
mkdir -p /u01/app/oracle/admin/CDBDEV/adump
ls      /u01/app/oracle/fast_recovery_area
ls      /u01/app/oracle/oradata
```

Notes:

- `mkdir -p` creates the directory if needed and does *not* complain if it already exists.
- The `ls` commands should show existing subdirectories (like `CDBTEST`, `ORCLCDB`, `RCATCDB`); it is fine if `CDBDEV` is not there yet.

---

## 6. Start the Instance in NOMOUNT and Run `CREATE DATABASE`

Time to actually use that parameter file.

1. Connect to SQL*Plus as SYSDBA:

   ```bash
   sqlplus / as sysdba
   ```

   You should see `Connected to an idle instance`.

2. Start the instance using `initCDBDEV.ora`:

   ```sql
   STARTUP NOMOUNT PFILE='$ORACLE_HOME/dbs/initCDBDEV.ora';
   ```

   Wait for:

   ```text
   ORACLE instance started.
   ```

3. Execute the `CREATE DATABASE` script:

   ```sql
   @/home/oracle/labs/DBMod_CreateDB/CrCDBDEV.sql
   ```

   This may take several minutes. If all goes well you will see confirmation that the database was created.

4. If you receive errors:

   - Follow the lab guide’s advice:

     ```sql
     SHUTDOWN ABORT;
     ```

   - Fix any typos in `initCDBDEV.ora` or `CrCDBDEV.sql`
   - Return to step 2 in this section (`STARTUP NOMOUNT ...`) and try again

---

## 7. Run the Data Dictionary Scripts (`catalog.sql` and `catproc.sql`)

After `CREATE DATABASE` succeeds, you need the catalog and PL/SQL packages.

From the same SYSDBA session:

1. Run the catalog script (about 3 minutes):

   ```sql
   @?/rdbms/admin/catalog.sql
   ```

2. Run the PL/SQL packages script (about 40 minutes):

   ```sql
   @?/rdbms/admin/catproc.sql
   ```

   The instructor paused the recording here for a reason. This is not fast.

3. When you see the end of `catproc.sql` (for example the final remarks), exit SQL*Plus:

   ```sql
   EXIT;
   ```

You now have a fully bootstrapped CDB.

---

## 8. Add `CDBDEV` to `/etc/oratab`

To make your new database known to tools like `oraenv` and `dbstart`, add an entry to `/etc/oratab`.

1. In the terminal, as `oracle`, append the line (exact home path may vary slightly; follow the value shown in your book):

   ```bash
   echo "CDBDEV:/u01/app/oracle/product/19.3.0/dbhome_1:N" | sudo tee -a /etc/oratab
   ```

   - The `N` at the end means “do not autostart with `dbstart`” (the lab uses that value).

2. Verify the new entry:

   ```bash
   grep CDBDEV /etc/oratab
   ```

   You should see the line you just added.

---

## 9. Verify the New Database Characteristics

Now we check that `CDBDEV` looks the way the exercise described.

1. Set the environment and connect:

   ```bash
   . oraenv      # when prompted, enter CDBDEV (case‑sensitive)
   sqlplus / as sysdba
   ```

2. Confirm that `CDBDEV` is a container database:

   ```sql
   SELECT cdb
   FROM   v$database;
   ```

   Expected:

   ```text
   CDB
   ---
   YES
   ```

3. Check datafile locations:

   ```sql
   COLUMN name FORMAT A80
   SELECT name
   FROM   v$datafile
   ORDER  BY 1;
   ```

   - You should see **7** datafiles (as shown in the guide)
   - The paths should be under `/u01/app/oracle/oradata` thanks to `db_create_file_dest`
   - PDB seed files will show GUID‑style names because of OMF

4. Verify the root tablespaces:

   ```sql
   SELECT tablespace_name,
          contents
   FROM   dba_tablespaces
   ORDER  BY tablespace_name;
   ```

   Confirm that you see at least:

   - `SYSTEM`
   - `SYSAUX`
   - The undo tablespace (for example `UNDOTBS`)
   - `TEMP`
   - `USERS`

5. Check the EM Express HTTPS port:

   ```sql
   SELECT DBMS_XDB_CONFIG.gethttpsport AS https_port
   FROM   dual;
   ```

   For a database created **manually** with SQL, this commonly returns `0` (not set).

   The lab explicitly calls this out: when you build the database with `CREATE DATABASE`, EM Express port **is not set automatically**. You must configure it later if you actually want to use EM Express on, say, port `5501`.

6. Exit SQL*Plus:

   ```sql
   EXIT;
   ```

---

## 10. What You Just Survived

By the end of this lab you:

- Confirmed there was no stray `CDBDEV` cluttering your data directories
- Edited a full `CREATE DATABASE` script to use the correct lab password
- Built an `initCDBDEV.ora` pfile from the stock `init.ora`
- Created all necessary directories for audit, data, and recovery files
- Started an instance in `NOMOUNT` and ran `CREATE DATABASE` by hand
- Executed `catalog.sql` and `catproc.sql` to finish the dictionary and PL/SQL environment
- Registered `CDBDEV` in `/etc/oratab`
- Verified:
  - It is a CDB (`CDB = YES`)
  - Datafiles live under the correct OMF directory
  - Required tablespaces (`SYSTEM`, `SYSAUX`, undo, `TEMP`, `USERS`) exist
  - EM Express port still needs to be set manually

In other words, you have now created a 19c CDB **entirely via SQL** and lived to tell the tale. You may now appreciate DBCA even more, which is exactly the point.

