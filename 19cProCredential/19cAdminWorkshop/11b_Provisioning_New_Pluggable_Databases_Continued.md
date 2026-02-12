# 11b - Provisioning New Pluggable Databases Continued (More Ways To Build, Move, and Confuse Future You)

This is the missing continuation right after creating PDBs from seed: the
practical menu of provisioning and migration methods, plus a live SQL demo of
seed-based creation and cleanup, with all the ways one quote mark can ruin your afternoon.

---

## 1. Provisioning Method Matrix

Ways to provision or move PDB workloads:

- create empty PDB from `PDB$SEED`
- convert non-CDB into PDB and plug into an existing CDB
- clone an existing PDB (with data)
- unplug/plug between CDBs (or same CDB under another name)
- relocate PDB using DB link
- create proxy PDB pointing to remote PDB via DB link

Key clarification from lecture:

- unplug/plug is not automatic source destruction
- relocation is the workflow that removes source after successful move

---

## 2. Tooling Options

All methods can be done using:

- SQL (`CREATE PLUGGABLE DATABASE ...`)
- SQL Developer GUI
- Enterprise Manager / Cloud Control
- DBCA

Version nuance called out:

- 12c DBCA: mainly empty PDB from seed workflows
- 18c+: DBCA can replicate existing populated PDBs too

---

## 3. Seed Content and Naming Notes

`PDB$SEED` includes base system structures needed for empty PDB creation
(`SYSTEM`, `SYSAUX`, baseline metadata framework, temp/undo behavior per config).

Operational guidance:

- keep seed naming/conventions standard (`PDB$SEED`) for compatibility with
 tooling and patch behavior.

---

## 4. File Placement Strategies for New PDBs

Main options discussed:

- `FILE_NAME_CONVERT` in create statement
- `PDB_FILE_NAME_CONVERT` parameter mapping full source/target paths
- pre-set `DB_CREATE_FILE_DEST` then create without explicit file mapping
- explicit `CREATE_FILE_DEST` in command for direct destination placement

Rule of thumb:

- explicit destination in command is simplest for one-off provisioning.
- parameter-driven defaults help when provisioning repeatedly.

---

## 5. Demo Flow: Create `PDBRON` from Seed

The live walkthrough sequence:

1. Connect as SYSDBA to CDB root:

```sql
sqlplus / as sysdba
SHOW PDBS;
```

2. Inspect seed datafile paths (using `CON_ID = 2` for `PDB$SEED`):

```sql
SELECT name, con_id
FROM   v$datafile
WHERE  con_id = 2;
```

3. Create new PDB with admin user and file-name conversion:

```sql
CREATE PLUGGABLE DATABASE pdbron
  ADMIN USER ron IDENTIFIED BY ron
  ROLES = (CONNECT)
  FILE_NAME_CONVERT = ('pdbseed','pdbron');
```

4. Open and verify:

```sql
SHOW PDBS;
ALTER PLUGGABLE DATABASE pdbron OPEN;
SHOW PDBS;
```

Expected:

- `PDBRON` appears as `MOUNTED`, then `READ WRITE` after open.

---

## 6. Demo Flow: Drop `PDBRON` Correctly

Close before drop:

```sql
ALTER PLUGGABLE DATABASE pdbron CLOSE IMMEDIATE;
```

Drop behavior:

- plain drop fails if datafiles remain referenced

Correct command:

```sql
DROP PLUGGABLE DATABASE pdbron INCLUDING DATAFILES;
```

Then verify:

```sql
SHOW PDBS;
```

`PDBRON` no longer listed. Vanished. Gone. Like an uncommitted change after `CLOSE ABORT`.

---

## 7. Control File and Identity Notes

Lecture emphasized identifier context:

- each PDB gets identity tracking in control-file metadata
- `CON_ID` and container-specific IDs drive observability/management views

Example check:

```sql
SELECT con_dbid, con_id
FROM   v$database;
```

And broader metadata/state details come from dynamic views (`v$` / `cdb_`).

---

## 8. Operational Takeaways

- hot operations (clone/relocate) depend on DB link workflows and local undo support.
- always validate target before removing source in manual migration patterns.
- be precise with quoting/path syntax in `CREATE PLUGGABLE DATABASE`; most failures
 are statement hygiene, not Oracle magic, and definitely not "a weird database mood."
