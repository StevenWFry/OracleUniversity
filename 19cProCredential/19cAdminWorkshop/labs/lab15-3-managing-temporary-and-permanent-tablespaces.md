# Lab 15-3 - Managing Temporary and Permanent Tablespaces

Practice 2-3 manages permanent and temporary tablespaces at both CDB root and
PDB level, then verifies default assignments for common and local users.

---

## 1. Practice Goal

You will:

- inspect current default permanent/temp tablespaces
- create permanent and temporary tablespaces in root and `ORCLPDB1`
- set new defaults at container level
- verify user-level tablespace assignments and update local user temp assignment

Assumption:

- `ORCLPDB1` exists and is open

---

## 2. Run Lab Formatting Script

```bash
/home/oracle/labs/storage/gLogin6.sh
```

---

## 3. Inspect Defaults in `ORCLCDB` Root

Connect:

```bash
sqlplus / as sysdba
```

Check defaults:

```sql
SELECT property_name, property_value
FROM   database_properties
WHERE  property_name LIKE 'DEFAULT%';
```

List tablespaces and container ids:

```sql
SELECT con_id, tablespace_name
FROM   cdb_tablespaces
ORDER  BY con_id, tablespace_name;
```

Filter temporary tablespaces:

```sql
SELECT con_id, tablespace_name
FROM   cdb_tablespaces
WHERE  contents = 'TEMPORARY'
ORDER  BY con_id, tablespace_name;
```

---

## 4. Create Root Permanent Tablespace `CDATA`

```sql
CREATE TABLESPACE cdata
  DATAFILE '/u01/app/oracle/oradata/ORCLCDB/cdata01.dbf'
  SIZE 10M;
```

Verify:

```sql
SELECT con_id, tablespace_name
FROM   cdb_tablespaces
WHERE  tablespace_name = 'CDATA';
```

Set root default permanent tablespace:

```sql
ALTER DATABASE DEFAULT TABLESPACE cdata;
```

Recheck defaults:

```sql
SELECT property_name, property_value
FROM   database_properties
WHERE  property_name LIKE 'DEFAULT%';
```

---

## 5. Create `LDATA` in `ORCLPDB1` and Set Default

Connect:

```sql
CONNECT system@orclpdb1
```

Password:

```text
cloud_4U
```

Create tablespace:

```sql
CREATE TABLESPACE ldata
  DATAFILE '/u01/app/oracle/oradata/ORCLCDB/orclpdb1/ldata01.dbf'
  SIZE 10M;
```

Set default permanent:

```sql
ALTER DATABASE DEFAULT TABLESPACE ldata;
```

Verify:

```sql
SELECT property_name, property_value
FROM   database_properties
WHERE  property_name LIKE 'DEFAULT%';
```

---

## 6. Create Root Temp Tablespace `TEMP_ROOT`

Return to root:

```sql
CONNECT / AS SYSDBA
```

Create temp tablespace:

```sql
CREATE TEMPORARY TABLESPACE temp_root
  TEMPFILE '/u01/app/oracle/oradata/ORCLCDB/temp_root01.dbf'
  SIZE 500M;
```

Set root default temp:

```sql
ALTER DATABASE DEFAULT TEMPORARY TABLESPACE temp_root;
```

Verify defaults:

```sql
SELECT property_name, property_value
FROM   database_properties
WHERE  property_name LIKE 'DEFAULT%';
```

---

## 7. Create `TEMP_PDB1` and `MY_TEMP` in `ORCLPDB1`

Connect:

```sql
CONNECT system@orclpdb1
```

Create `TEMP_PDB1`:

```sql
CREATE TEMPORARY TABLESPACE temp_pdb1
  TEMPFILE '/u01/app/oracle/oradata/ORCLCDB/orclpdb1/temp_pdb101.dbf'
  SIZE 100M;
```

Set as default temp for this PDB:

```sql
ALTER DATABASE DEFAULT TEMPORARY TABLESPACE temp_pdb1;
```

Verify:

```sql
SELECT property_name, property_value
FROM   database_properties
WHERE  property_name LIKE 'DEFAULT%';
```

Create additional temp tablespace:

```sql
CREATE TEMPORARY TABLESPACE my_temp
  TEMPFILE '/u01/app/oracle/oradata/ORCLCDB/orclpdb1/my_temp01.dbf'
  SIZE 10M;
```

---

## 8. Validate Defaults in Another PDB via Script

Run:

```bash
@/home/oracle/labs/DBMod_Storage/setup_newpdb.sql
```

Script behavior (per guide):

- creates test PDB
- queries defaults
- drops test PDB

---

## 9. Manage User-Level Defaults

Create common user (root):

```sql
CONNECT system@orclcdb
CREATE USER c##cu IDENTIFIED BY cloud_4U;
```

Check common user assignment across containers:

```sql
SELECT con_id, username, default_tablespace, temporary_tablespace
FROM   cdb_users
WHERE  username = 'C##CU'
ORDER  BY con_id;
```

Create local user in `ORCLPDB1`:

```sql
CONNECT system@orclpdb1
CREATE USER lu IDENTIFIED BY cloud_4U;
```

Check local defaults:

```sql
SELECT username, default_tablespace, temporary_tablespace
FROM   dba_users
WHERE  username = 'LU';
```

Change local user temp tablespace:

```sql
ALTER USER lu TEMPORARY TABLESPACE my_temp;
```

Verify:

```sql
SELECT username, default_tablespace, temporary_tablespace
FROM   dba_users
WHERE  username = 'LU';
```

---

## 10. Exit

```sql
EXIT
```

---

## 11. Lab Result

You created and assigned permanent/temp tablespaces in root and PDB contexts,
validated container defaults, and confirmed user-level temp tablespace behavior
for common and local users.
