# Lab 12-2 - Cloning Remote PDBs in Hot Mode (Part 2)

Part 2 covers the live clone operation, refresh behavior (manual and automatic),
and cleanup.

---

## 1. Resume After Setup Completion

After `setup_hotclone.sh` completes (about 20+ minutes in the demo), continue
with step 5.

---

## 2. Grant Privilege for Hot Clone User

In `ORCLCDB` as SYSDBA:

```sql
GRANT CREATE PLUGGABLE DATABASE TO system CONTAINER=ALL;
```

Then:

```sql
EXIT
```

---

## 3. Start Source Transaction in `PDB_SOURCE_FOR_HOTCLONE`

Open terminal titled `session ORCLCDB`, then connect:

```bash
sqlplus system@pdb_source_for_hotclone
```

Password:

```text
cloud_4U
```

Set prompt and begin source-side activity (query/update as guided in lab).

---

## 4. Prepare `CDBTEST` Session and Directory

Open terminal titled `session CDBTEST`.

Create target directory:

```bash
mkdir -p /u01/app/oracle/oradata/CDBTEST/pdb_hotclone
```

Set environment and connect:

```bash
. oraenv
```

When prompted, enter:

```text
cdbtest
```

Then:

```bash
sqlplus / as sysdba
```

---

## 5. Create DB Link and Hot Clone (Manual Refresh Mode)

Cleanup old link (expected to fail if absent):

```sql
DROP PUBLIC DATABASE LINK link_pdb_source_for_hotclone;
```

Create link:

```sql
CREATE PUBLIC DATABASE LINK link_pdb_source_for_hotclone
CONNECT TO system IDENTIFIED BY cloud_4U
USING 'pdb_source_for_hotclone';
```

Set destination:

```sql
ALTER SESSION SET db_create_file_dest='/u01/app/oracle/oradata/CDBTEST/pdb_hotclone';
```

Create refreshable hot clone:

```sql
CREATE PLUGGABLE DATABASE pdb_hotclone
FROM pdb_source_for_hotclone@link_pdb_source_for_hotclone
FILE_NAME_CONVERT=(
  '/u01/app/oracle/oradata/ORCLCDB/pdb_source_for_hotclone',
  '/u01/app/oracle/oradata/CDBTEST/pdb_hotclone'
)
REFRESH MODE MANUAL;
```

Expected:

- `Pluggable database created.`

---

## 6. Open Clone Read-Only and Validate Data

```sql
ALTER PLUGGABLE DATABASE pdb_hotclone OPEN READ ONLY;
ALTER SESSION SET CONTAINER=pdb_hotclone;
SELECT DISTINCT label FROM source_user.bigtab;
SELECT COUNT(*) FROM source_user.bigtab;
```

Expected row count in demo:

- `10000`

---

## 7. Manual Refresh Workflow

After committing new source changes in `ORCLCDB`, refresh from `CDBTEST`.

If refresh is attempted while clone is open, you get the expected error that the
refreshable PDB must be closed.

Correct sequence:

```sql
ALTER PLUGGABLE DATABASE pdb_hotclone CLOSE;
ALTER PLUGGABLE DATABASE pdb_hotclone REFRESH;
ALTER PLUGGABLE DATABASE pdb_hotclone OPEN READ ONLY;
```

Then recheck:

```sql
SELECT DISTINCT label FROM source_user.bigtab;
```

---

## 8. Recreate Clone for Automatic Refresh

Close and return to root:

```sql
ALTER PLUGGABLE DATABASE pdb_hotclone CLOSE;
ALTER SESSION SET CONTAINER=CDB$ROOT;
```

Check status:

```sql
SELECT pdb_name, status FROM cdb_pdbs;
```

Drop and recreate with auto refresh interval:

```sql
DROP PLUGGABLE DATABASE pdb_hotclone INCLUDING DATAFILES;
```

```sql
CREATE PLUGGABLE DATABASE pdb_hotclone
FROM pdb_source_for_hotclone@link_pdb_source_for_hotclone
FILE_NAME_CONVERT=(
  '/u01/app/oracle/oradata/ORCLCDB/pdb_source_for_hotclone',
  '/u01/app/oracle/oradata/CDBTEST/pdb_hotclone'
)
REFRESH MODE EVERY 2 MINUTES;
```

Note from lab:

- refreshable copy must be closed for refresh to execute; otherwise refresh is deferred.

---

## 9. Validate Auto Refresh

In `ORCLCDB`, update and commit source data (example label changed to `DATA2`).

In `CDBTEST`, verify refreshed data after scheduled interval / closed-state refresh:

```sql
ALTER SESSION SET CONTAINER=pdb_hotclone;
ALTER PLUGGABLE DATABASE pdb_hotclone OPEN READ ONLY;
SELECT DISTINCT label FROM source_user.bigtab;
```

Transcript result:

- clone showed refreshed value (`DATA2`).

---

## 10. Cleanup

Run cleanup script:

```bash
cd /home/oracle/labs/DBMod_PDBs
./cleanup_hotclone.sh
```

This drops:

- `PDB_SOURCE_FOR_HOTCLONE` in `ORCLCDB`
- `PDB_HOTCLONE` in `CDBTEST`

Close all terminal windows.

---

## 11. Lab Result

You performed hot clone from remote source, validated manual refresh behavior,
recreated clone with automatic refresh, confirmed refreshed data propagation,
and cleaned up lab artifacts.
