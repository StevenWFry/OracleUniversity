# Lab 17-1 - Creating Common and Local Users

Practice 1-1 covers user creation scope in multitenant: common user in CDB root
vs local user in `ORCLPDB1`, plus privilege behavior with and without
`SYSDBA`.

---

## 1. Practice Goal

You will:

- create common admin user `C##CDB_ADMIN1`
- grant common privileges/roles across containers
- compare privilege sets when connecting with and without `SYSDBA`
- create local admin user `PDB1ADMIN` in `ORCLPDB1`
- validate container scope restrictions

---

## 2. Locate Guide and Credentials

From desktop:

1. `Home` -> `eKit`
2. Open unzipped course folder
3. Open `*_ag.pdf` (Activity Guide)
4. Check security credentials (page 6 in transcript flow)

Lab password values used:

```text
cloud_4U
```

---

## 3. Start/Reset Lab State

Open terminal:

```bash
cd /home/oracle/labs
./dbstart.sh
```

Set environment:

```bash
. oraenv
```

When prompted:

```text
orclcdb
```

Reset `ORCLPDB1`:

```bash
/home/oracle/labs/DBMod_Users/reset_orclpdb1.sh
```

Reset users/roles:

```bash
/home/oracle/labs/DBMod_Users/reset_userroles.sh
```

Ignore object-not-exist reset errors when noted by guide.

---

## 4. Create Common User in Root

Connect as SYSDBA:

```bash
sqlplus / as sysdba
```

Create common user:

```sql
CREATE USER c##cdb_admin1 IDENTIFIED BY cloud_4U
  DEFAULT TABLESPACE users
  TEMPORARY TABLESPACE temp
  ACCOUNT UNLOCK
  CONTAINER=ALL;
```

Grant privileges/role commonly:

```sql
GRANT create session, dba, sysdba
TO c##cdb_admin1
CONTAINER=ALL;
```

Verify common users:

```sql
SELECT DISTINCT username
FROM   dba_users
WHERE  common = 'YES'
ORDER  BY username;
```

---

## 5. Compare SYSDBA vs Regular Connection

Disconnect and show user:

```sql
DISCONNECT
SHOW USER
```

Connect exercising `SYSDBA`:

```sql
CONNECT anything/anything AS SYSDBA
SHOW CON_NAME
SHOW USER
SELECT * FROM session_privs;
```

Transcript note:

- session user shown as `SYS` when connected with `SYSDBA`
- privileges list is larger (sample showed 253 rows)

Disconnect and connect as regular common user:

```sql
DISCONNECT
CONNECT c##cdb_admin1/cloud_4U
SHOW USER
SELECT * FROM session_privs;
```

Transcript comparison:

- fewer privileges than SYSDBA mode (sample showed 237 rows)

---

## 6. Create Local User in `ORCLPDB1`

Switch container:

```sql
ALTER SESSION SET CONTAINER=orclpdb1;
SHOW CON_NAME;
```

Create local user (no `CONTAINER=ALL`):

```sql
CREATE USER pdb1admin IDENTIFIED BY cloud_4U
  DEFAULT TABLESPACE users
  TEMPORARY TABLESPACE temp
  ACCOUNT UNLOCK;
```

Grant local privileges:

```sql
GRANT create session, dba TO pdb1admin;
```

List local users:

```sql
SELECT username
FROM   dba_users
WHERE  common = 'NO'
ORDER  BY username;
```

Expected:

- includes `PDB1ADMIN` and existing local accounts like `PDB_ADMIN`

---

## 7. Verify Local User Scope

Connect as `PDB1ADMIN`:

```sql
CONNECT pdb1admin/cloud_4U@orclpdb1
SHOW USER
SELECT * FROM session_privs;
```

Try moving to root:

```sql
ALTER SESSION SET CONTAINER=CDB$ROOT;
```

Expected:

- insufficient privileges / not allowed due to local-user scope and missing
 container-level privilege.

---

## 8. Exit

```sql
EXIT
```

---

## 9. Lab Result

You created and validated both common and local administrative users, confirmed
`CONTAINER=ALL` behavior, compared SYSDBA vs regular privilege footprints, and
demonstrated that local PDB users cannot operate at root scope by default.
