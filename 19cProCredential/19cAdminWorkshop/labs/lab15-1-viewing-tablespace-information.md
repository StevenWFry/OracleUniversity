# Lab 15-1 - Viewing Tablespace Information

Practice 2-1 focuses on exploring tablespace metadata in `ORCLPDB1` using both
SQL*Plus and SQL Developer.

---

## 1. Practice Goal

You will:

- query dictionary and performance views for tablespace details
- inspect tables, indexes, segments, and datafiles
- review the same storage details in SQL Developer

Assumption:

- logged in as `oracle` OS user

---

## 2. Start DB and Listener

Open terminal and run:

```bash
cd /home/oracle/labs
./dbstart.sh
```

Expected:

- listener and database open (or already running)

---

## 3. Set Environment and Grant DBA to `pdb_admin`

Open new terminal:

```bash
. oraenv
```

When prompted, enter:

```text
orclcdb
```

Connect to `ORCLPDB1` as SYSDBA:

```bash
sqlplus sys/cloud_4U@orclpdb1 as sysdba
```

Grant:

```sql
GRANT dba TO pdb_admin;
```

---

## 4. Connect as `pdb_admin` and Inspect Views

```sql
CONNECT pdb_admin@orclpdb1
```

Password:

```text
cloud_4U
```

List columns in tablespace view:

```sql
DESC dba_tablespaces;
```

List tablespaces in `ORCLPDB1`:

```sql
SELECT tablespace_name
FROM   dba_tablespaces;
```

Find HR schema tablespace:

```sql
SELECT DISTINCT tablespace_name
FROM   all_tables
WHERE  owner = 'HR';
```

Expected:

- `USERS`

Inspect `SYSAUX` properties:

```sql
SELECT status, contents, logging, plugged_in, bigfile, extent_management, allocation_type
FROM   dba_tablespaces
WHERE  tablespace_name = 'SYSAUX';
```

---

## 5. Use `V$TABLESPACE` for Control-File Runtime Info

```sql
DESC v$tablespace;
```

Then query `SYSAUX`:

```sql
SELECT name, included_in_database_backup, bigfile, flashback_on, encrypt_in_backup, con_id
FROM   v$tablespace
WHERE  name = 'SYSAUX';
```

Expected interpretation in lab:

- included in DB backup: `YES`
- bigfile: `NO` (smallfile)
- flashback participation: `YES`
- `CON_ID` reflects current PDB container id

---

## 6. HR Objects in `USERS` Tablespace

Tables:

```sql
SELECT table_name
FROM   dba_tables
WHERE  owner = 'HR'
AND    tablespace_name = 'USERS';
```

Indexes:

```sql
SELECT index_name
FROM   dba_indexes
WHERE  owner = 'HR'
AND    tablespace_name = 'USERS';
```

---

## 7. Datafile and Segment Analysis

Datafile view columns:

```sql
DESC dba_data_files;
```

`SYSAUX` datafile details:

```sql
SELECT file_name, autoextensible, bytes, maxbytes, user_bytes
FROM   dba_data_files
WHERE  tablespace_name = 'SYSAUX';
```

Count segments in `SYSAUX`:

```sql
SELECT COUNT(*)
FROM   dba_segments
WHERE  tablespace_name = 'SYSAUX';
```

Largest index segment in `SYSAUX`:

```sql
SELECT segment_name, bytes
FROM   dba_segments
WHERE  tablespace_name = 'SYSAUX'
AND    segment_type = 'INDEX'
ORDER  BY bytes DESC
FETCH FIRST 1 ROW ONLY;
```

Exit SQL*Plus:

```sql
EXIT
```

---

## 8. Review in SQL Developer

Launch SQL Developer and open DBA navigator for `PDB1` connection.

Path:

- `Storage` -> `Tablespaces`

Review tabs:

- `Space`
- `Files`
- `Free Space`

Check `SYSAUX` usage metrics:

- used %
- free space MB

Lab note:

- values vary by environment; transcript showed ~92-94% used.

Close SQL Developer when finished.

---

## 9. Lab Result

You inspected tablespace metadata, runtime tablespace status, HR object
placement, SYSAUX datafile/segment footprint, and cross-checked storage
utilization in SQL Developer.
