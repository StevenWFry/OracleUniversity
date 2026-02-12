# Lab 13-2 - Setting Parameter Values for PDBs

Practice 3-2 investigates how initialization parameter values behave at CDB root
and PDB scope, using `optimizer_use_sql_plan_baselines` as the example.

---

## 1. Practice Goal

Validate that:

- not all parameters are PDB-modifiable
- PDB-local parameter changes can diverge from root defaults
- values persist per-container behavior through PDB close/open and CDB restart

---

## 2. Connect as SYSDBA and Check Modifiability

```bash
sqlplus / as sysdba
```

Check whether parameter is PDB-modifiable:

```sql
SELECT ispdb_modifiable
FROM   v$parameter
WHERE  name = 'optimizer_use_sql_plan_baselines';
```

Also check with:

```sql
SHOW PARAMETER optimizer_use_sql_plan_baselines;
```

---

## 3. Check and Set Value in `PDB3_ORCL`

Connect:

```sql
CONNECT system@pdb3_orcl
```

Password:

```text
cloud_4U
```

Check current value:

```sql
SHOW PARAMETER optimizer_use_sql_plan_baselines;
```

If needed for lab consistency, set value in this PDB:

```sql
ALTER SYSTEM SET optimizer_use_sql_plan_baselines = TRUE SCOPE=BOTH;
```

Recheck:

```sql
SHOW PARAMETER optimizer_use_sql_plan_baselines;
```

---

## 4. Create `TEST` PDB and Compare Default

Create directory (per guide path):

```bash
mkdir -p /u01/app/oracle/oradata/ORCLCDB/test
```

Create/open PDB from root:

```sql
CONNECT / AS SYSDBA
CREATE PLUGGABLE DATABASE test
  ADMIN USER test_admin IDENTIFIED BY cloud_4U
  ROLES = (CONNECT)
  CREATE_FILE_DEST = '/u01/app/oracle/oradata/ORCLCDB/test';
ALTER PLUGGABLE DATABASE test OPEN;
```

Add TNS alias via lab script:

```bash
cd /home/oracle/labs/DBMod_PDBs
./add_testtns.sh
```

Connect to `TEST` and inspect parameter:

```sql
CONNECT sys@test AS SYSDBA
SHOW PARAMETER optimizer_use_sql_plan_baselines;
```

Expected in transcript:

- new `TEST` PDB shows root default (`TRUE`).

---

## 5. Verify `PDB3_ORCL` Retains Its Own Value

In `PDB3_ORCL`, close/open and recheck:

```sql
CONNECT / AS SYSDBA
ALTER PLUGGABLE DATABASE pdb3_orcl CLOSE IMMEDIATE;
ALTER PLUGGABLE DATABASE pdb3_orcl OPEN;
CONNECT system@pdb3_orcl
SHOW PARAMETER optimizer_use_sql_plan_baselines;
```

Transcript observation:

- `PDB3_ORCL` retained `FALSE` while root/new PDB default remained `TRUE`.

---

## 6. Restart CDB and Compare Container Values

From root:

```sql
CONNECT / AS SYSDBA
SHUTDOWN IMMEDIATE;
STARTUP;
SHOW PARAMETER optimizer_use_sql_plan_baselines;
ALTER PLUGGABLE DATABASE ALL OPEN;
SHOW PDBS;
```

Check values by container:

```sql
SELECT con_id, value
FROM   v$system_parameter
WHERE  name = 'optimizer_use_sql_plan_baselines'
ORDER  BY con_id;
```

Expected pattern:

- root container value `TRUE`
- `PDB3_ORCL` container value `FALSE`

---

## 7. Cleanup (`TEST` and `PDB3_ORCL`)

Close PDBs:

```sql
ALTER PLUGGABLE DATABASE test CLOSE;
ALTER PLUGGABLE DATABASE pdb3_orcl CLOSE;
```

Drop:

```sql
DROP PLUGGABLE DATABASE test INCLUDING DATAFILES;
DROP PLUGGABLE DATABASE pdb3_orcl INCLUDING DATAFILES;
```

Remove directories:

```bash
rm -rf /u01/app/oracle/oradata/ORCLCDB/test
rm -rf /u01/app/oracle/oradata/ORCLCDB/orclpdb3
```

Exit:

```sql
EXIT
```

---

## 8. Lab Result

You validated container-specific parameter behavior, created a comparison PDB,
confirmed persistence across close/open and restart, and cleaned up lab objects.
