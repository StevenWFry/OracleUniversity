# Lab 18-1 - Granting Local DBA Role to `PDBADMIN`

Practice 2-1 explores existing privilege/role grants for `PDBADMIN` in
`ORCLPDB1`, then grants local `DBA` role so later labs can create users, roles,
and profiles.

---

## 1. Practice Goal

You will:

- inspect system privileges granted directly to `PDBADMIN`
- inspect roles granted to `PDBADMIN`
- inspect role composition (`PDB_DBA`, `CONNECT`)
- grant local `DBA` role to `PDBADMIN`
- verify resulting role grants

Assumption:

- logged in as `oracle` OS user
- database/listener are running

Optional checks:

```bash
pgrep -lf smon
pgrep -lf tns
```

If needed:

```bash
cd /home/oracle/labs
./dbstart.sh
```

---

## 2. Connect as SYSDBA

```bash
sqlplus / as sysdba
```

Switch to target PDB:

```sql
ALTER SESSION SET CONTAINER = orclpdb1;
```

---

## 3. Explore Existing Grants for `PDBADMIN`

Direct system privileges:

```sql
SELECT grantee, privilege, admin_option
FROM   dba_sys_privs
WHERE  grantee = 'PDBADMIN'
ORDER  BY privilege;
```

Expected in lab:

- no direct system privileges returned

Roles granted to `PDBADMIN`:

```sql
SELECT grantee, granted_role, admin_option
FROM   cdb_role_privs
WHERE  grantee = 'PDBADMIN'
ORDER  BY granted_role;
```

Expected:

- `PDB_DBA` present
- `ADMIN_OPTION = YES` (per guide output)

---

## 4. Explore Role Composition

System privileges in `PDB_DBA`:

```sql
SELECT role, privilege
FROM   role_sys_privs
WHERE  role = 'PDB_DBA'
ORDER  BY privilege;
```

Expected:

- `CREATE SESSION`
- `CREATE PLUGGABLE DATABASE`

Roles granted to `PDB_DBA`:

```sql
SELECT grantee, granted_role
FROM   dba_role_privs
WHERE  grantee = 'PDB_DBA'
ORDER  BY granted_role;
```

Expected:

- `CONNECT`

System privileges in `CONNECT` role:

```sql
SELECT role, privilege
FROM   role_sys_privs
WHERE  role = 'CONNECT'
ORDER  BY privilege;
```

Expected:

- `CREATE SESSION`
- `SET CONTAINER`

---

## 5. Grant Local `DBA` Role to `PDBADMIN`

```sql
GRANT dba TO pdbadmin;
```

Verify role grants:

```sql
SELECT grantee, granted_role, admin_option
FROM   dba_role_privs
WHERE  grantee = 'PDBADMIN'
ORDER  BY granted_role;
```

Expected:

- both `DBA` and `PDB_DBA` listed

---

## 6. Exit

```sql
EXIT
```

Close terminal.

---

## 7. Lab Result

You validated the default role model for `PDBADMIN`, analyzed role inheritance
(`PDB_DBA` -> `CONNECT`), granted local `DBA`, and confirmed the updated role
set required for later administration practices.
