# Lab 15-2 - Creating a Tablespace (Part 1)

Practice 2-2 starts by creating a tablespace named `INVENTORY`, attempting to
populate table `X`, then intentionally hitting a space error before fixing it in
the next section.

---

## 1. Practice Goal

In Part 1, you will:

- create `INVENTORY` tablespace
- run table creation/load script for `X`
- reproduce expected space error (`unable to extend table`)
- launch SQL Developer to begin the resize fix

Assumption:

- logged in as `oracle` OS user

---

## 2. Open Terminal and Connect

1. Right-click desktop -> `Open Terminal`
2. Resize terminal for readability
3. Connect to SQL*Plus as `pdb_admin` (per activity guide connection details)

Example pattern:

```bash
sqlplus pdb_admin@orclpdb1
```

Password:

```text
cloud_4U
```

---

## 3. Run Tablespace Creation Script

Execute:

```sql
@/home/oracle/labs/DBMod_Storage/Create_Inventory_tablespace.sql
```

Expected:

- `INVENTORY` tablespace and initial datafile (for example `inventory01.dbf`)
  are created.

---

## 4. Run Table Build/Load Script for `X`

Execute:

```sql
@/home/oracle/labs/DBMod_Storage/Create_Table_X.sql
```

Expected near script end:

- error similar to:

```text
ORA-01653: unable to extend table PDB_ADMIN.X by 128 in tablespace INVENTORY
```

Interpretation:

- this is expected in lab flow
- `INVENTORY` datafile is too small for the load volume

---

## 5. Begin Resize Fix in SQL Developer

Next section starts in SQL Developer:

1. Open from menu:
 - `Applications` -> `Programming` -> `SQL Developer`
2. In DBA navigator, expand:
 - `PDB1` -> `PDB_ADMIN`

Part 1 stops here, right before performing the actual datafile resize steps.
