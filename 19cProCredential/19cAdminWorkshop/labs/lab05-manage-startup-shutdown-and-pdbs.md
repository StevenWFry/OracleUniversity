# Lab 5 - Startup, Shutdown, and PDB Open Modes (a.k.a. Please Don't Kill the Whole CDB)

And look, theory about NOMOUNT / MOUNT / OPEN is nice, but until you've actually shut the database down and brought it back up without nuking all the PDBs, it's just words.

In this lab you will:

- Set your environment for `ORCLCDB`
- Create `ORCLPDB3` from a helper script
- Walk through shutdown and startup phases
- Inspect PDB open modes and saved state behavior
- Drop a single PDB without touching the rest of the CDB

---

## 1. Assumptions

- You are logged in as the `oracle` OS user.
- The base CDB is `ORCLCDB`.
- The lab provides helper scripts in `/home/oracle/labs/DBMod_CreateDB`.

---

## 2. Create `ORCLPDB3`

1. Set the environment:

   ```bash
   . oraenv
   # When prompted:
   ORACLE_SID = [CDBDEV] ? orclcdb
   ```

2. Run the setup script:

   ```bash
   /home/oracle/labs/DBMod_CreateDB/setup_pdb3.sh
   ```

   Note: The script may show errors about dropping objects that do not exist. The activity guide says to ignore those.

3. The script creates and opens `ORCLPDB3`, then creates a `TEST` user and a `TEST.BIGTAB` table with rows (used later for shutdown behavior).

---

## 3. Connect as SYSDBA

```bash
sqlplus / as sysdba
```

---

## 4. Shut Down in IMMEDIATE Mode

```sql
SHUTDOWN IMMEDIATE;
```

This closes datafiles and redo logs, dismounts the database, and stops the instance.

Check the current user (still `SYS`):

```sql
SHOW USER;
```

Show container name (expect error because the database is down):

```sql
SHOW CON_NAME;
```

---

## 5. Start Up in Phases

1. Start the instance in `NOMOUNT`:

   ```sql
   STARTUP NOMOUNT;
   ```

2. Mount the database:

   ```sql
   ALTER DATABASE MOUNT;
   ```

3. Open the database:

   ```sql
   ALTER DATABASE OPEN;
   ```

4. Confirm container and user:

   ```sql
   SHOW CON_NAME;
   SHOW USER;
   ```

---

## 6. Inspect PDB Open Modes

1. Query `V$PDBS`:

   ```sql
   COLUMN name FORMAT A10
   SELECT con_id, name, open_mode FROM v$pdbs;
   ```

   Expect `ORCLPDB3` to be `MOUNTED` while `ORCLPDB1` and `ORCLPDB2` may be `READ WRITE`.

2. Check saved PDB states:

   ```sql
   COLUMN con_name FORMAT A16
   SELECT con_id, con_name, state FROM dba_pdb_saved_states;
   ```

---

## 7. Clear Saved States and Observe Behavior

1. Close all PDBs:

   ```sql
   ALTER PLUGGABLE DATABASE ALL CLOSE;
   ```

2. Save the closed state:

   ```sql
   ALTER PLUGGABLE DATABASE ALL SAVE STATE;
   ```

3. Verify saved states are cleared:

   ```sql
   SELECT con_id, con_name, state FROM dba_pdb_saved_states;
   ```

4. Shut down and restart:

   ```sql
   SHUTDOWN IMMEDIATE;
   STARTUP;
   ```

5. Confirm all PDBs are now `MOUNTED`:

   ```sql
   SELECT con_id, name, open_mode FROM v$pdbs;
   ```

---

## 8. Reset Saved States to OPEN

1. Open all PDBs:

   ```sql
   ALTER PLUGGABLE DATABASE ALL OPEN;
   ```

2. Save the open state:

   ```sql
   ALTER PLUGGABLE DATABASE ALL SAVE STATE;
   ```

3. Confirm saved states:

   ```sql
   SELECT con_id, con_name, state FROM dba_pdb_saved_states;
   ```

---

## 9. Close and Drop `ORCLPDB3` Only

1. Close `ORCLPDB3`:

   ```sql
   ALTER PLUGGABLE DATABASE orclpdb3 CLOSE;
   ```

2. Drop it (including datafiles):

   ```sql
   DROP PLUGGABLE DATABASE orclpdb3 INCLUDING DATAFILES;
   ```

3. Exit SQL*Plus:

   ```sql
   EXIT;
   ```

---

## 10. What You Practiced (And Didn't Blow Up)

By the end of this lab you:

- Created a new PDB with a helper script
- Ran through `NOMOUNT`, `MOUNT`, and `OPEN` phases
- Managed PDB saved states and verified their impact
- Dropped a single PDB without stopping the whole CDB

In short: you can now control startup behavior at the PDB level without collateral damage.
