# Lab 13-1 - Managing PDBs: Renaming and Open Modes

Practice 3-1 focuses on renaming a PDB and using the correct open mode
(`RESTRICTED`) for that operation.

---

## 1. Practice Goal

Rename `ORCLPDB3` to `PDB3_ORCL` and validate:

- required restricted mode behavior
- normal reopen behavior
- client connectivity via updated TNS service entry

Assumption:

- database and listener are running

---

## 2. Recreate Source PDB for Practice

Run setup script:

```bash
cd /home/oracle/labs/DBMod_PDBs
./setup_pdb3.sh
```

Password when prompted:

```text
cloud_4U
```

---

## 3. Connect to Root and List PDBs

Set environment and connect:

```bash
. oraenv
```

When prompted, enter:

```text
orclcdb
```

Then:

```bash
sqlplus / as sysdba
SHOW PDBS;
```

---

## 4. Attempt Rename and Observe Required Context

From root, this fails by design:

```sql
ALTER PLUGGABLE DATABASE orclpdb3 RENAME GLOBAL_NAME TO pdb3_orcl;
```

Error indicates operation must be performed from within the PDB context.

Connect to target PDB:

```sql
CONNECT system@orclpdb3
```

Password:

```text
cloud_4U
```

Retry rename in normal mode and observe restricted-mode requirement.

---

## 5. Open in Restricted Mode and Rename

Run:

```sql
ALTER PLUGGABLE DATABASE CLOSE;
ALTER PLUGGABLE DATABASE OPEN RESTRICTED;
```

Verify restricted status:

```sql
SELECT name, open_mode, restricted
FROM   v$pdbs;
```

Rename:

```sql
ALTER PLUGGABLE DATABASE orclpdb3 RENAME GLOBAL_NAME TO pdb3_orcl;
```

Recheck:

```sql
SELECT name, open_mode, restricted
FROM   v$pdbs;
```

Expected:

- name now reflects `PDB3_ORCL`

---

## 6. Return to Normal Open Mode

```sql
ALTER PLUGGABLE DATABASE CLOSE;
ALTER PLUGGABLE DATABASE OPEN;
```

Verify read-write/non-restricted state:

```sql
SELECT name, open_mode, restricted
FROM   v$pdbs;
```

Exit SQL*Plus when done:

```sql
EXIT
```

---

## 7. Update TNS Alias and Test Connectivity

Add/refresh service entry via provided script:

```bash
cd /home/oracle/labs/DBMod_PDBs
./add_pdb3_tns.sh
```

Test connect:

```bash
sqlplus system@pdb3_orcl
```

Password:

```text
cloud_4U
```

Expected:

- successful connection to renamed PDB service

Exit:

```sql
EXIT
```

---

## 8. Lab Result

You renamed `ORCLPDB3` to `PDB3_ORCL` using required restricted mode workflow,
reopened the PDB normally, updated client naming, and verified successful login.
