# Lab 5 – Startup, Shutdown, and PDB Open Modes (a.k.a. Please Don’t Kill the Whole CDB)

And look, theory about NOMOUNT / MOUNT / OPEN is nice, but until you’ve actually shut the database down and brought it back up without nuking all the PDBs, it’s just words.

In this lab you will:

- Use `oraenv` to set your environment for `ORCLCDB`
- Create a new pluggable database `ORCLPDB3` from a helper script
- Shut down and start up the CDB in different phases
- Inspect PDB states and saved states
- Reset saved state behaviour
- Shut down and drop a single PDB without touching the rest of the CDB

---

## 1. Lab Assumptions

- You are logged in as the `oracle` OS user.
- The base CDB is `ORCLCDB`.
- The lab image provides helper scripts:
  - `dbstart.sh` to start the CDB + listener (if needed)
  - `setup_pdb3.sh` to create/configure `ORCLPDB3`

If your paths or names differ slightly, follow your activity guide’s values.

---

## 2. Set the Environment and Create `ORCLPDB3`

1. Open a terminal (right‑click desktop → **Open Terminal**).
2. Set the environment for `ORCLCDB` using `oraenv`:

   ```bash
   . oraenv
   # When prompted for ORACLE_SID, enter:
   ORCLCDB
   ```

3. Move to the directory that contains the `setup_pdb3.sh` script and list the files:

   ```bash
   cd /home/oracle/labs       # adjust if your guide uses a different path
   ls
   ```

   Confirm that `setup_pdb3.sh` exists.

4. Execute the script:

   ```bash
   ./setup_pdb3.sh
   ```

   You should see output including:

   - `ALTER PLUGGABLE DATABASE ORCLPDB3 OPEN;`
   - Creation of users/privileges for that PDB

When the script finishes, `ORCLPDB3` exists and is ready to be experimented on.

---

## 3. Connect as SYSDBA

Start SQL*Plus and connect as a privileged user:

```bash
sqlplus / as sysdba
```

You should see `Connected to:` and a reference to `ORCLCDB`.

---

## 4. Shutdown `ORCLCDB` in Normal Mode

You’ll begin with a **clean** shutdown.

1. Issue a normal shutdown:

   ```sql
   SHUTDOWN;
   ```

   Notes:

   - `SHUTDOWN` with no modifier means `SHUTDOWN NORMAL`.
   - New connections are refused.
   - The command waits for all sessions to disconnect voluntarily.

   In the lab environment there should be no other active sessions, so shutdown should complete quickly with messages similar to:

   - `Database closed.`
   - `Database dismounted.`
   - `ORACLE instance shut down.`

2. Check who SQL*Plus thinks you are (even though the DB is down):

   ```sql
   SHOW USER;
   ```

   Expected:

   ```text
   USER is "SYS"
   ```

3. Show the current container:

   ```sql
   SHOW CON_NAME;
   ```

   With the database shut down, this should return an error like “Oracle not available” – which is exactly the point: certain features require the database to be open.

---

## 5. Start Up in Phases: NOMOUNT → MOUNT → OPEN

Now you’ll bring the instance and database back up in stages.

1. Start the instance in **NOMOUNT** mode:

   ```sql
   STARTUP NOMOUNT;
   ```

   At this stage Oracle:

   - Reads the parameter file (spfile/pfile)
   - Allocates the SGA
   - Starts background processes
   - Opens alert and trace files

   No control files or datafiles are opened yet, and user data remains inaccessible.

2. Mount the database:

   ```sql
   ALTER DATABASE MOUNT;
   ```

   Now:

   - Control files are opened
   - Oracle reads file metadata and checks whether clean shutdown or crash recovery is required

3. Open the database:

   ```sql
   ALTER DATABASE OPEN;
   ```

   At this point:

   - Datafiles and redo logs are opened
   - The CDB root is available for normal use

4. Verify your container and user:

   ```sql
   SHOW CON_NAME;
   SHOW USER;
   ```

   Expected:

   ```text
   CON_NAME
   ---------
   CDB$ROOT

   USER is "SYS"
   ```

---

## 6. Inspect PDB Open Modes

Now check the current state of the PDBs.

1. Query `V$PDBS`:

   ```sql
   COLUMN name FORMAT A15
   SELECT name,
          open_mode
   FROM   v$pdbs
   ORDER  BY name;
   ```

   You should see entries for:

   - `PDB$SEED` (usually `READ ONLY`)
   - `ORCLPDB1`
   - `ORCLPDB2`
   - `ORCLPDB3`

   In this practice, after startup, `ORCLPDB3` typically shows as `MOUNTED` while some others may be `READ WRITE`.

2. Inspect **saved states** for PDBs:

   ```sql
   COLUMN con_name FORMAT A16
   SELECT con_id,
          con_name,
          state
   FROM   dba_pdb_saved_states
   ORDER  BY con_id;
   ```

   Expectation:

   - You may see `ORCLPDB1` and `ORCLPDB2` with `STATE = OPEN`
   - `ORCLPDB3` likely does **not** appear yet (no saved state)

This is the mechanism Oracle uses to remember how PDBs should come up next time.

---

## 7. Clear All Saved States and Observe the Effect

Now you’ll remove saved states and verify what happens on restart.

1. Close **all** PDBs:

   ```sql
   ALTER PLUGGABLE DATABASE ALL CLOSE;
   ```

2. Immediately save the new (closed) state:

   ```sql
   ALTER PLUGGABLE DATABASE ALL SAVE STATE;
   ```

   At this moment, you have effectively overwritten any previously saved “OPEN” states with “CLOSED.”

3. Re‑query saved states:

   ```sql
   SELECT con_id,
          con_name,
          state
   FROM   dba_pdb_saved_states;
   ```

   You should see:

   ```text
   no rows selected
   ```

   (Any previous saved state entries have been cleared.)

4. Shut down the CDB (this time using the more realistic mode):

   ```sql
   SHUTDOWN IMMEDIATE;
   ```

5. Start it back up:

   ```sql
   STARTUP;
   ```

6. Inspect PDB states again:

   ```sql
   SELECT name,
          open_mode
   FROM   v$pdbs
   ORDER  BY name;
   ```

   Now all PDBs (including `ORCLPDB1`, `ORCLPDB2`, and `ORCLPDB3`) should show as `MOUNTED`.

---

## 8. Reset PDB Saved States to OPEN

You’ll now configure all PDBs to auto‑open with the CDB.

1. Open all PDBs:

   ```sql
   ALTER PLUGGABLE DATABASE ALL OPEN;
   ```

2. Save the state for all of them:

   ```sql
   ALTER PLUGGABLE DATABASE ALL SAVE STATE;
   ```

3. Re‑query `DBA_PDB_SAVED_STATES`:

   ```sql
   SELECT con_id,
          con_name,
          state
   FROM   dba_pdb_saved_states
   ORDER  BY con_id;
   ```

   You should now see entries for the PDBs with `STATE = OPEN`.

On the next restart, the CDB will automatically bring those PDBs back to the `OPEN` state once the root is open.

---

## 9. Close and Drop a Single PDB (`ORCLPDB3`)

Now you’ll practice shutting down and removing **just** one PDB.

1. Close `ORCLPDB3`:

   ```sql
   ALTER PLUGGABLE DATABASE orclpdb3 CLOSE;
   ```

2. Check the states:

   ```sql
   SELECT name,
          open_mode
   FROM   v$pdbs
   ORDER  BY name;
   ```

   Expectation:

   - `ORCLPDB3` shows as `MOUNTED`
   - Other PDBs (for example `ORCLPDB1`, `ORCLPDB2`) can still be `READ WRITE`

3. Drop `ORCLPDB3` **including its datafiles**:

   ```sql
   DROP PLUGGABLE DATABASE orclpdb3 INCLUDING DATAFILES;
   ```

   This removes:

   - The dictionary definition of the PDB
   - Its datafiles from disk

4. Confirm it’s gone:

   ```sql
   SELECT name,
          open_mode
   FROM   v$pdbs
   ORDER  BY name;
   ```

   `ORCLPDB3` should no longer be listed.

5. Exit SQL*Plus:

   ```sql
   EXIT;
   ```

---

## 10. What You Practiced (And Didn’t Blow Up)

By the end of this lab you:

- Used `oraenv` to target `ORCLCDB`
- Created a new PDB (`ORCLPDB3`) via a helper script
- Performed a **Normal** shutdown and a phased startup (`NOMOUNT` → `MOUNT` → `OPEN`)
- Verified:
  - Your container (`CDB$ROOT`)
  - Your current user (`SYS`)
- Inspected PDB states via `V$PDBS`
- Examined and reset `DBA_PDB_SAVED_STATES` to control PDB auto‑open behaviour
- Shut down and restarted the CDB with different saved state configurations
- Safely closed and dropped **only** `ORCLPDB3` while leaving other PDBs intact

You have now seen, hands‑on, how easy it is to manage PDBs precisely with `ALTER PLUGGABLE DATABASE`, and how catastrophically easy it would be to get it wrong with an unqualified `SHUTDOWN`. Stick with the former.

