# Lab 3 – Creating `CDBTEST` with DBCA Silent Mode

And look, you *could* type out a multi‑page `CREATE DATABASE` command by hand, carefully placing every comma like you are defusing a bomb. Or you can let DBCA do the heavy lifting while you supervise with coffee and a vague sense of superiority.  

This lab walks through creating a new **container database** called `CDBTEST` using **DBCA in silent mode**, driven by a helper shell script.

---

## 1. Lab Goal and Requirements

You will:

- Start the existing CDB instance and listener (if they are not already up)
- Edit a DBCA helper script so that all passwords use the lab value `cloud_4U`
- Run DBCA in **silent mode** via that script to create `CDBTEST`
- Verify that the new CDB is present and open

Assumptions:

- You are logged in to the lab VM as the `oracle` OS user
- The base CDB (for example `orclcdb`) is already configured on the host
- The course “Practice Environment Security Credentials” page says the common password is:

  ```text
  cloud_4U
  ```

---

## 2. Start the CDB and Listener (if needed)

1. Open a terminal on the lab desktop (right‑click desktop → **Open Terminal**).
2. Check whether the CDB is already running, for example:

   ```bash
   pgrep -lf orclcdb
   ```

   - If you see nothing, the instance is down.
3. Start the CDB and listener with the provided helper script (path may vary slightly by image; use the one given in your guide), for example:

   ```bash
   cd /home/oracle/labs
   ./dbstart.sh
   ```

   Watch for messages like:

   - Listener started
   - Instance started, database mounted, database opened

4. Optionally verify from SQL*Plus:

   ```bash
   sqlplus / as sysdba

   SQL> SELECT name, open_mode, cdb FROM v$database;
   SQL> EXIT;
   ```

---

## 3. Make Sure `CDBTEST` Is Not Already Registered

The `oratab` file tells `oraenv`/`dbstart` which databases exist on the host.

1. From the terminal, display `/etc/oratab`:

   ```bash
   cat /etc/oratab
   ```

2. Check for a line containing `CDBTEST` (or whatever exact name your lab uses):

   - If **no** `CDBTEST` entry appears, you are good.
   - If you **do** see an entry for `CDBTEST` left over from a previous run, follow your activity guide instructions to remove or comment out that line before continuing.

---

## 4. Edit the DBCA Silent Script

The lab provides a shell script that wraps a long `dbca -silent` command. Your job is to make it use the correct passwords.

1. Change to the directory that contains the script (match the exact path from your activity guide), for example:

   ```bash
   cd /home/oracle/labs/dbmod/createdb
   ls
   ```

   Look for a file named something like:

   ```text
   CDBTEST.sh
   ```

2. Open the script in a text editor (the guide uses `gedit`):

   ```bash
   gedit CDBTEST.sh &
   ```

3. In the script, locate **all** password parameters. Typical ones are:

   ```bash
   -sysPassword <something>
   -systemPassword <something>
   -pdbAdminPassword <something>
   # sometimes additional admin passwords are also present in italics in the guide
   ```

4. Replace every italicised / placeholder password with the lab password:

   ```text
   cloud_4U
   ```

   When you are done, the lines should look roughly like:

   ```bash
   -sysPassword cloud_4U \
   -systemPassword cloud_4U \
   -pdbAdminPassword cloud_4U \
   ```

5. Save the file and close the editor.

6. Back in the terminal, confirm the changes:

   ```bash
   cat CDBTEST.sh
   ```

   Verify that every password value is now `cloud_4U`.

---

## 5. Make the Script Executable and Run DBCA

1. Mark the script as executable:

   ```bash
   chmod +x CDBTEST.sh
   ```

2. Run the script:

   ```bash
   ./CDBTEST.sh
   ```

3. Now wait. And this is the part where DBCA does a lot of very boring but very important work:

   - Allocates memory for the new instance  
   - Creates control files, datafiles, and redo logs as Oracle Managed Files  
   - Builds the root CDB dictionary and the seed PDB  
   - Configures the default tablespaces:
     - `TEMP` (temporary)
     - `USERS` (default permanent)
     - `UNDOTBS` (undo)
   - Configures EM Express on port `5502`

   The script may take several minutes. This is normal. If something is wrong, you will see an error; otherwise, be patient.

---

## 6. Verify the New CDB `CDBTEST`

Once the script completes successfully:

1. Verify the instance is running:

   ```bash
   pgrep -lf cdbtest
   ```

   You should see an `ora_pmon_CDBTEST` (or similar) process.

2. Connect as SYSDBA:

   ```bash
   sqlplus sys/cloud_4U@CDBTEST as sysdba
   ```

   (If your lab uses a different service name, use that instead of `CDBTEST`.)

3. Confirm database properties:

   ```sql
   SELECT name, cdb, open_mode FROM v$database;

   SELECT pdb_name, open_mode
   FROM   v$pdbs
   ORDER BY pdb_id;
   ```

   You should see:

   - A CDB with `CDB` = `YES`
   - The root open in `READ WRITE`
   - Only the seed PDB (`PDB$SEED`) present (no additional user PDBs yet)

4. Leave the session open for some deeper checks in the next section, or exit now if you prefer and reconnect later.

---

## 7. Deep‑Dive Verification (Files, Tablespaces, EM Express Port)

The lab guide also walks you through checking that DBCA put things **exactly** where you asked it to. This is where we confirm that.

> If you exited SQL*Plus in the previous step, reconnect first:
>
> ```bash
> . oraenv    # when prompted, enter CDBTEST (case‑sensitive)
> sqlplus / as sysdba
> ```

1. **Confirm again that `CDBTEST` is a container database**

   ```sql
   SELECT cdb FROM v$database;
   ```

   You should see:

   ```text
   CDB
   ---
   YES
   ```

2. **Verify datafile locations**

   Set a wider display and list the datafiles:

   ```sql
   COLUMN name FORMAT A80
   SELECT name
   FROM   v$datafile
   ORDER  BY 1;
   ```

   Check that:

   - The paths match the Oracle Managed Files location from the lab instructions
   - You see the expected number of files (for example, 7 datafiles as shown in the guide)

3. **Verify tablespaces**

   ```sql
   SELECT tablespace_name,
          contents
   FROM   dba_tablespaces
   ORDER  BY tablespace_name;
   ```

   You should see at least:

   - `SYSTEM`
   - `SYSAUX`
   - `UNDOTBS` (or similar undo tablespace name from the script)
   - `TEMP`
   - `USERS`

4. **Verify the EM Express HTTPS port**

   The script configured EM Express on port `5502`. Confirm with:

   ```sql
   SELECT DBMS_XDB_CONFIG.gethttpsport AS https_port
   FROM   dual;
   ```

   You should see:

   ```text
   HTTPS_PORT
   ----------
        5502
   ```

5. Now exit SQL*Plus:

   ```sql
   EXIT;
   ```

---

## 8. What You Just Achieved (Without Typing `CREATE DATABASE`)

By the end of this lab you:

- Started the base CDB and listener if they were down
- Used a **DBCA silent‑mode script** to create a brand‑new CDB called `CDBTEST`
- Centralised all system passwords on the lab‑standard `cloud_4U` value
- Verified that:
  - The instance is running
  - The database is a proper CDB
  - Only the root and seed PDB exist (no extra PDBs yet)

In other words, you now have **two** databases on the host and still have not typed a single multi‑screen `CREATE DATABASE` statement. That is exactly how it should be.
