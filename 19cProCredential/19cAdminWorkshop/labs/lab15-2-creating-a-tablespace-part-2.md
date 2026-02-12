# Lab 15-2 - Creating a Tablespace (Part 2)

Part 2 fixes the `INVENTORY` space error in SQL Developer, adds a second data
file, reruns load, verifies table placement, then drops the tablespace.

---

## 1. Continue in SQL Developer (DBA View)

If `PDB1 / PDB_ADMIN` DBA connection is not present, use available PDB DBA
connection (for example `PDB1 / SYS`) to perform datafile management tasks.

Navigate:

- `PDB1` -> `Storage` -> `Tablespaces`
- open/double-click `INVENTORY`
- open `Data Files` tab

Expected:

- `inventory01.dbf` appears fully used (100%).

---

## 2. Resize `inventory01.dbf` to 40M

1. In DBA pane -> `Data Files`, double-click `inventory01.dbf`.
2. `Actions` -> `Edit`.
3. Set file size to:

```text
40M
```

4. Open `SQL` tab to view generated command.
5. Click `Apply`, then `OK` on success dialog.

Verify:

- file size now shows `40M`.

---

## 3. Add `inventory02.dbf` (30M)

1. In `INVENTORY` tablespace, choose:
 - `Actions` -> `Add Data File`
2. Enter:

- File name: `inventory02.dbf`
- Directory: `/u01/app/oracle/oradata/ORCLCDB/ORCLPDB1`
- Size: `30M`

3. Open `SQL` tab to review generated DDL.
4. Click `Apply`, then `OK`.
5. Refresh `Data Files` tab.

Expected:

- two datafiles listed for `INVENTORY`.

Close SQL Developer.

---

## 4. Rerun Load Script in SQL*Plus

Connect as `pdb_admin` to `ORCLPDB1`:

```bash
sqlplus pdb_admin@orclpdb1
```

Password:

```text
cloud_4U
```

Rerun table creation/load script:

```sql
@/home/oracle/labs/storage/Create_Table_X.sql
```

Expected this time:

- no extend error
- script completes with rows inserted/committed (transcript showed 2048 rows)

---

## 5. Verify Table `X` Is in `INVENTORY`

Reconnect if needed, then run:

```sql
SELECT table_name, tablespace_name
FROM   user_tables
WHERE  table_name = 'X';
```

Expected:

- `X` in `INVENTORY`.

---

## 6. Drop `INVENTORY` Tablespace

From SQL*Plus as privileged user in `ORCLPDB1`, drop tablespace per guide
instructions.

Typical form:

```sql
DROP TABLESPACE inventory INCLUDING CONTENTS AND DATAFILES;
```

---

## 7. Lab Result

You resolved `INVENTORY` space exhaustion by resizing and adding datafiles,
successfully populated table `X`, confirmed object placement, and completed
cleanup by dropping the tablespace.
