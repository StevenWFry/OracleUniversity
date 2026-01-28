# Lab 3 - Creating `CDBTEST` with DBCA Silent Mode

And look, you *could* type out a multipage `CREATE DATABASE` command by hand, carefully placing every comma like you are defusing a bomb. Or you can let DBCA do the heavy lifting while you supervise with coffee and a vague sense of superiority.

This lab walks through creating a new **container database** called `CDBTEST` using **DBCA in silent mode**, driven by a helper shell script.

---

## 1. Lab Goal and Requirements

You will:

- Format your shell/SQL*Plus output for readability
- Edit a DBCA helper script so all passwords use the lab value `cloud_4U`
- Run DBCA in **silent mode** to create `CDBTEST`
- Verify database characteristics and EM Express port
- Clean up with the provided drop script

Assumptions:

- You are logged in to the lab VM as the `oracle` OS user
- The base CDB (for example `orclcdb`) is already configured on the host
- The course "Practice Environment Security Credentials" page says the common password is:

  ```text
  cloud_4U
  ```

---

## 2. Format Output and Start the CDB (if needed)

1. Open a terminal on the lab desktop (right-click desktop **Open Terminal**).

2. Run the provided formatting script:

   ```bash
   $HOME/labs/DBMod_CreateDB/glogin.sh
   ```

3. Check whether the base CDB is already running, for example:

   ```bash
   pgrep -lf orclcdb
   ```

   - If you see nothing, the instance is down.

4. Start the CDB and listener with the helper script (path may vary slightly by image; use the one given in your guide):

   ```bash
   cd /home/oracle/labs
   ./dbstart.sh
   ```

5. Optionally verify from SQL*Plus:

   ```bash
   sqlplus / as sysdba

   SQL> SELECT name, open_mode, cdb FROM v$database;
   SQL> EXIT;
   ```

---

## 3. Confirm `CDBTEST` Is Not Already Registered

The `oratab` file tells `oraenv`/`dbstart` which databases exist on the host.

1. From the terminal, display `/etc/oratab`:

   ```bash
   cat /etc/oratab
   ```

2. Check for a line containing `CDBTEST`:

   - If **no** `CDBTEST` entry appears, you are good.
   - If you **do** see an entry for `CDBTEST` left over from a previous run, follow your activity guide instructions to remove or comment out that line before continuing.

---

## 4. Edit the DBCA Silent Script

The lab provides a shell script that wraps a long `dbca -silent` command. Your job is to make it use the correct passwords.

1. Change to the directory that contains the script:

   ```bash
   cd /home/oracle/labs/DBMod_CreateDB
   ls
   ```

   Look for:

   ```text
   CrCDBTEST.sh
   ```

2. Open the script in a text editor (the guide uses `gedit`):

   ```bash
   gedit CrCDBTEST.sh &
   ```

3. In the script, locate **all** password parameters (for example `-sysPassword`, `-systemPassword`, `-pdbAdminPassword`, `-dbsnmpPassword`).

4. Replace each placeholder with the lab password:

   ```text
   cloud_4U
   ```

5. Save the file and close the editor.

6. Back in the terminal, confirm the changes:

   ```bash
   cat CrCDBTEST.sh
   ```

   Verify that every password value is now `cloud_4U`.

7. Note from the activity guide: the dashes in the script are **flags**, not line continuations. Do not delete or reflow them when editing.

---

## 5. Make the Script Executable and Run DBCA

1. Mark the script as executable:

   ```bash
   chmod 755 CrCDBTEST.sh
   ```

2. Run the script:

   ```bash
   ./CrCDBTEST.sh
   ```

3. Wait for completion. Expect 10-15 minutes. DBCA reports progress and logs to:

   ```text
   /u01/app/oracle/cfgtoollogs/dbca/CDBTEST
   ```

---

## 6. Verify the New CDB `CDBTEST`

1. Verify the instance is running:

   ```bash
   pgrep -lf cdbtest
   ```

   You should see an `ora_pmon_CDBTEST` (or similar) process.

2. Connect as SYSDBA:

   ```bash
   . oraenv
   # When prompted, enter:
   CDBTEST

   sqlplus / as sysdba
   ```

3. Confirm database properties:

   ```sql
   SELECT cdb FROM v$database;

   SELECT pdb_name, open_mode
   FROM   v$pdbs
   ORDER  BY pdb_id;
   ```

   You should see:

   - `CDB = YES`
   - Root open in `READ WRITE`
   - Only the seed PDB (`PDB$SEED`) present

---

## 7. Verify Files, Tablespaces, and EM Express Port

1. Verify datafile locations:

   ```sql
   COLUMN name FORMAT A58
   SELECT name
   FROM   v$datafile
   ORDER  BY 1;
   ```

   You should see datafiles under:

   ```text
   /u01/app/oracle/oradata/CDBTEST
   ```

2. Verify tablespaces:

   ```sql
   COLUMN tablespace_name FORMAT A15
   COLUMN contents FORMAT A15
   SELECT tablespace_name, contents
   FROM   dba_tablespaces;
   ```

3. Verify the EM Express HTTPS port:

   ```sql
   SELECT dbms_xdb_config.gethttpsport() FROM dual;
   ```

   Expected:

   ```text
   5502
   ```

4. Exit SQL*Plus:

   ```sql
   EXIT;
   ```

---

## 8. Confirm `/etc/oratab` Entry

DBCA adds a new entry to `/etc/oratab`. Confirm it exists:

```bash
cat /etc/oratab
```

You should see:

```text
CDBTEST:/u01/app/oracle/product/19.3.0/dbhome_1:N
```

---

## 9. Clean Up (per the Activity Guide)

When instructed, remove `CDBTEST`:

```bash
/home/oracle/labs/DBMod_CreateDB/dropCDBTEST.sh
```

---

## 10. What You Just Achieved (Without Typing `CREATE DATABASE`)

By the end of this lab you:

- Ran DBCA in silent mode to create `CDBTEST`
- Standardized all system passwords on `cloud_4U`
- Verified CDB status, tablespaces, files, and EM Express port
- Confirmed the `/etc/oratab` entry and cleaned up with `dropCDBTEST.sh`

In other words, you now have **two** databases on the host and still have not typed a single multiscreen `CREATE DATABASE` statement. That is exactly how it should be.
