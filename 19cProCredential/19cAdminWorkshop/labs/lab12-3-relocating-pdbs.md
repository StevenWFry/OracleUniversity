# Lab 12-3 - Relocating PDBs

Practice 2-2 demonstrates near-zero-downtime PDB relocation by moving
`ORCLPDB3` from `ORCLCDB` to `CDBTEST` in one operation.

---

## 1. Practice Goal

Relocate source PDB to target CDB while preserving workload continuity as much
as possible.

Assumptions:

- databases/listeners are running
- `CDBTEST` exists and is open

---

## 2. Prepare Source PDB3

Run setup script:

```bash
cd /home/oracle/labs/DBMod_PDBs
./setup_pdb3.sh
```

Password when prompted:

```text
cloud_4U
```

Script outcome:

- recreates `ORCLPDB3`
- adds `PDB3` service in `tnsnames.ora`

---

## 3. Validate Source and Data (Session `ORCLCDB1`)

Set environment and connect as SYSDBA:

```bash
. oraenv
sqlplus / as sysdba
```

Check local undo enabled:

```sql
SELECT property_name, property_value
FROM   database_properties
WHERE  property_name = 'LOCAL_UNDO_ENABLED';
```

Verify `TEST.BIGTAB` contents in `PDB3`:

```sql
CONNECT test@pdb3
SELECT label, COUNT(*)
FROM   test.bigtab
GROUP  BY label;
```

Expected:

- row count around `10000`

---

## 4. Source-Side DB Link and PDB State

Reconnect SYSDBA in `ORCLCDB1` and create link to `CDBTEST`:

```sql
DROP PUBLIC DATABASE LINK link_cdbtest;
CREATE PUBLIC DATABASE LINK link_cdbtest
CONNECT TO system IDENTIFIED BY cloud_4U
USING 'cdbtest';
```

Open and save PDB states:

```sql
ALTER PLUGGABLE DATABASE ALL OPEN;
ALTER PLUGGABLE DATABASE ALL SAVE STATE;
SHOW PDBS;
```

---

## 5. Target Setup (Session `CDBTEST`)

Set environment and connect:

```bash
. oraenv
sqlplus / as sysdba
```

Create link back to source:

```sql
DROP PUBLIC DATABASE LINK link_orclpdb3;
CREATE PUBLIC DATABASE LINK link_orclpdb3
CONNECT TO system IDENTIFIED BY cloud_4U
USING 'orclcdb';
```

Prepare destination directory:

```bash
mkdir -p /u01/app/oracle/oradata/CDBTEST/pdb_relocated
```

---

## 6. Relocate PDB

Initial relocate command may fail if privilege missing.

Grant required privilege on source (`ORCLCDB1`):

```sql
GRANT SYSOPER TO system CONTAINER=ALL;
```

Then run in `CDBTEST`:

```sql
CREATE PLUGGABLE DATABASE pdb_relocated
FROM orclpdb3@link_orclpdb3
FILE_NAME_CONVERT = (
  '/u01/app/oracle/oradata/ORCLCDB/orclpdb3',
  '/u01/app/oracle/oradata/CDBTEST/pdb_relocated'
);
```

Check status:

```sql
SELECT pdb_name, status FROM cdb_pdbs;
```

Expected intermediate status:

- `RELOCATING`

---

## 7. Run Concurrent Session Script During Relocation

In separate terminal/session (named in demo as `ORCLCDB2`), run:

```bash
cd /home/oracle/labs/DBMod_PDBs
./sessions.sh
```

Use `cloud_4U` when prompted.

Do not wait for script completion before continuing next steps.

---

## 8. Open Relocated PDB and Validate Data

In `CDBTEST`:

```sql
ALTER PLUGGABLE DATABASE pdb_relocated OPEN READ ONLY;
ALTER SESSION SET CONTAINER=pdb_relocated;
SELECT label, COUNT(*) FROM test.bigtab GROUP BY label;
```

If relocation finalization takes too long, transition using force mode:

```sql
ALTER SESSION SET CONTAINER=CDB$ROOT;
ALTER PLUGGABLE DATABASE pdb_relocated OPEN READ WRITE FORCE;
```

Then recheck:

```sql
ALTER SESSION SET CONTAINER=pdb_relocated;
SELECT label, COUNT(*) FROM test.bigtab GROUP BY label;
SELECT pdb_name, status FROM cdb_pdbs;
```

Expected final status:

- `NORMAL`

---

## 9. Verify Source PDB Removal

In `ORCLCDB1`:

```sql
SELECT pdb_name, status FROM cdb_pdbs;
```

Expected:

- source `ORCLPDB3` no longer present after relocation completion

---

## 10. Cleanup and Privilege Revoke

In `CDBTEST`:

```sql
ALTER SESSION SET CONTAINER=CDB$ROOT;
ALTER PLUGGABLE DATABASE pdb_relocated CLOSE;
DROP PLUGGABLE DATABASE pdb_relocated INCLUDING DATAFILES;
EXIT
```

In `ORCLCDB1`:

```sql
sqlplus / as sysdba
REVOKE SYSOPER FROM system CONTAINER=ALL;
EXIT
```

---

## 11. Lab Result

You relocated a PDB from `ORCLCDB` to `CDBTEST`, observed relocation states,
validated data continuity on target, finalized migration, and cleaned up both
objects and temporary elevated privileges.
