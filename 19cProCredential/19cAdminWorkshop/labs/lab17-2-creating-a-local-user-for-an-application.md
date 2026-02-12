# Lab 17-2 - Creating a Local User for an Application

Practice 19-2 creates a local application owner account in `ORCLPDB1` and
verifies least-privilege connectivity.

Source:

- Activity Guide (`D106546GC10_ag_unlocked.pdf`), Practice 19-2

---

## 1. Practice Goal

Create local user `INVENTORY` in `ORCLPDB1` for an application account that does
not represent a person, then verify login and granted privileges.

Assumption:

- logged in to compute node as `oracle` OS user

---

## 2. Catchup (Only If You Skipped Practice 19-1)

```bash
/home/oracle/labs/DBMod_UsersSec/setup_users_lab19.sh
```

If Practice 19-1 was completed, continue directly to step 3.

---

## 3. Connect as Local PDB Administrator

Open terminal and connect:

```bash
sqlplus pdb1_admin/cloud_4U@orclpdb1
```

---

## 4. Create Local Application User `INVENTORY`

Create user with default tablespace and quota:

```sql
CREATE USER inventory IDENTIFIED BY cloud_4U
  DEFAULT TABLESPACE users
  QUOTA UNLIMITED ON users;
```

Grant minimum login privilege:

```sql
GRANT create session TO inventory;
```

---

## 5. Verify Local User Exists in `ORCLPDB1`

```sql
SELECT DISTINCT username
FROM   dba_users
WHERE  common = 'NO'
ORDER  BY 1;
```

Expected:

- `INVENTORY` appears with other local users.

---

## 6. Connect as `INVENTORY` and Verify Privileges

Disconnect `PDB1_ADMIN`:

```sql
DISCONNECT
```

Connect as `INVENTORY`:

```sql
CONNECT inventory/cloud_4U@orclpdb1
```

Check session privileges:

```sql
SELECT *
FROM   session_privs
ORDER  BY 1;
```

Expected:

- `CREATE SESSION`

---

## 7. Exit

```sql
EXIT
```

Close terminal.

---

## 8. Lab Result

You created a local application account (`INVENTORY`) in `ORCLPDB1`, granted
only the required login privilege, and verified scope/privileges through
`DBA_USERS` and `SESSION_PRIVS`.
