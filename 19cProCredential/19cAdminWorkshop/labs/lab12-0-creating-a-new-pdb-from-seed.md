# Lab 12-0 - Creating a New PDB from the PDB Seed

Practice 1 of the pluggable database module: create `ORCLPDB3` from
`PDB$SEED`, wire up a client service, and verify files/services landed where
they should.

---

## 1. Find the Activity Guide and Credentials

From desktop:

1. Open `Home`
2. Open `eKit`
3. Open the unzipped course folder
4. Open the `*_ag.pdf` file (Activity Guide)

Credential reference:

- Security Credentials section: around pages 5-6
- Lab password used in this course image:

```text
cloud_4U
```

---

## 2. Verify Database and Listener State

Check running processes:

```bash
pgrep -lf smon
pgrep -lf tns
```

If not running, start with:

```bash
cd /home/oracle/labs
./dbstart.sh
```

Transcript note:

- if already running, `dbstart.sh` reports startup not needed.

---

## 3. Set Environment and Create PDB

Set CDB environment:

```bash
. oraenv
```

When prompted, enter:

```text
orclcdb
```

Connect as SYSDBA:

```bash
sqlplus / as sysdba
```

Create the new PDB:

```sql
CREATE PLUGGABLE DATABASE orclpdb3
  ADMIN USER pdb3_admin IDENTIFIED BY cloud_4U
  ROLES = (CONNECT)
  CREATE_FILE_DEST = '/u01/app/oracle/oradata';
```

Expected:

- `Pluggable database created.`

---

## 4. Check and Open the New PDB

Inspect status:

```sql
COLUMN pdb_name FORMAT A20
SELECT pdb_id, pdb_name, status
FROM   cdb_pdbs;
```

Expected initial status for new PDB:

- `NEW`

Open it:

```sql
ALTER PLUGGABLE DATABASE orclpdb3 OPEN;
```

Recheck:

```sql
SELECT pdb_id, pdb_name, status
FROM   cdb_pdbs;
```

Expected:

- `ORCLPDB3` status becomes `NORMAL`

Exit SQL*Plus.

---

## 5. Create Net Service Alias `PDB3`

Launch Net Manager:

```bash
netmgr
```

In UI:

1. Expand `Local` -> `Service Naming`
2. Click `+`
3. Net Service Name: `PDB3`
4. Protocol: `TCP/IP`
5. Hostname: `localhost`
6. Port: `1521`
7. Service Name: `ORCLPDB3`
8. Click `Finish`
9. `File` -> `Save Network Configuration`
10. Exit Net Manager

---

## 6. Connect to `PDB3` and Verify Datafiles

Connect:

```bash
sqlplus system@pdb3
```

Password:

```text
cloud_4U
```

Check datafile location:

```sql
SET LINESIZE 80
COLUMN name FORMAT A78
SELECT name FROM v$datafile;
```

Expected:

- datafiles under OMF-style path rooted at
  `/u01/app/oracle/oradata/...`

---

## 7. Verify Service Registration

In the same session:

```sql
SELECT name FROM v$services;
```

Expected:

- `ORCLPDB3` service is present

Exit SQL*Plus:

```sql
EXIT
```

---

## 8. Lab Result

You created `ORCLPDB3` from seed, opened it, configured client alias `PDB3`,
validated OMF datafile placement, and confirmed service visibility.
