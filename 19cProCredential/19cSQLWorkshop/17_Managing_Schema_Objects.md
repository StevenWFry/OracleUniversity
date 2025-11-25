# Managing Schema Objects (Chapter 15)

## Table of Contents
- [Overview](#overview)
- [Managing Constraints](#managing-constraints)  
  - [Add / Drop Constraints](#add--drop-constraints)  
  - [Enable / Disable / Cascade](#enable--disable--cascade)  
  - [ONLINE clause](#online-clause)  
  - [ON DELETE actions](#on-delete-actions)  
  - [ALTER TABLE ... RENAME](#alter-table--rename)  
  - [Validation Modes (VALIDATE / NOVALIDATE)](#validation-modes-validate--novalidate)  
  - [Deferrable Constraints / Deferred vs Immediate](#deferrable-constraints--deferred-vs-immediate)
- [Temporary Tables](#temporary-tables)  
  - [Global Temporary Tables (GTT)](#global-temporary-tables-gtt)  
  - [Private Temporary Tables (PTT)](#private-temporary-tables-ptt)  
  - [ON COMMIT behavior](#on-commit-behavior)
- [External Tables](#external-tables)  
  - [Directory objects & privileges](#directory-objects--privileges)  
  - [Oracle Loader external table example](#oracle-loader-external-table-example)  
  - [Oracle Data Pump external table example](#oracle-data-pump-external-table-example)
- [Recycle Bin & PURGE](#recycle-bin--purge)
- [Practical Tips](#practical-tips)

## Overview
This lesson covers altering and managing constraints, creating/using temporary tables (global & private), and creating/using external tables (OS files accessed as tables). Examples are Oracle-centric.

---

## Managing Constraints

### Add / Drop Constraints
- Use ALTER TABLE ... ADD CONSTRAINT for all constraints except NOT NULL (use MODIFY).
- Drop constraint by name with ALTER TABLE ... DROP CONSTRAINT.

Example — add FK (table-level named constraint):
```sql
ALTER TABLE emp2
  ADD CONSTRAINT fk_emp2_mgr
  FOREIGN KEY (manager_id) REFERENCES emp2(employee_id);
```

Add NOT NULL (column-level):
```sql
ALTER TABLE emp2 MODIFY (email VARCHAR2(100) NOT NULL);
```

Drop constraint:
```sql
ALTER TABLE emp2 DROP CONSTRAINT fk_emp2_mgr;
```

### Enable / Disable / Cascade
- Disable constraints to speed bulk loads or relax validation order; re-enable later.
- DROP PRIMARY KEY CASCADE removes dependent FKs.
- DISABLE ... CASCADE disables constraint and dependents.

Disable FK:
```sql
ALTER TABLE emp2 DISABLE CONSTRAINT fk_emp2_mgr;
```

Enable constraint:
```sql
ALTER TABLE emp2 ENABLE CONSTRAINT fk_emp2_mgr;
```

Disable PK and dependent FKs:
```sql
ALTER TABLE parent_tab DISABLE CONSTRAINT parent_pk CASCADE;
```

### ONLINE clause
- Some ALTER operations support ONLINE to allow concurrent DML while DDL runs.
- Example: drop constraint online (if supported):
```sql
ALTER TABLE emp2 DROP CONSTRAINT parent_pk ONLINE;
```
-- Note: DROP CONSTRAINT ... ONLINE is version-dependent. If your Oracle release does not support ONLINE for DROP CONSTRAINT, use the standard DROP and perform during maintenance, or use DBMS_REDEFINITION / disabling strategies to minimize blocking.
-- Safe example:
ALTER TABLE emp2 DROP CONSTRAINT parent_pk;
-- Or disable first, then drop during a maintenance window:
ALTER TABLE emp2 DISABLE CONSTRAINT parent_pk;
ALTER TABLE emp2 DROP CONSTRAINT parent_pk;

### ON DELETE actions
- Define behavior for deleting parent rows.

Cascade delete:
```sql
ALTER TABLE emp2
  ADD CONSTRAINT fk_emp_dept
  FOREIGN KEY (department_id) REFERENCES departments(department_id)
  ON DELETE CASCADE;
```

Set NULL:
```sql
... ON DELETE SET NULL;
```

### ALTER TABLE ... RENAME
Rename table, column or constraint:
```sql
ALTER TABLE marketing RENAME TO new_marketing;
ALTER TABLE new_marketing RENAME COLUMN team_id TO id;
ALTER TABLE new_marketing RENAME CONSTRAINT marketing_pk TO new_marketing_pk;
```

### Validation Modes (VALIDATE / NOVALIDATE)
- Control whether enabling a constraint checks existing data.
- ENABLE VALIDATE: enable and validate all existing rows.
- ENABLE NOVALIDATE: enable constraint for new DML but do not validate existing rows.
- DISABLE VALIDATE / DISABLE NOVALIDATE also available.

Example:
```sql
ALTER TABLE dept2 ENABLE NOVALIDATE PRIMARY KEY;
```

### Deferrable Constraints — Deferred vs Immediate
- DEFERRABLE constraints can be deferred to transaction commit.
- INITIALLY DEFERRED / INITIALLY IMMEDIATE controls default behavior.
- alter session to control checking mode.

-- Recommended: define deferrable PK as a table-level constraint
CREATE TABLE demo (
  id NUMBER,
  name VARCHAR2(225),
  CONSTRAINT demo_pk PRIMARY KEY (id) DEFERRABLE INITIALLY DEFERRED
);

Session control:
```sql
-- Note: ALTER SESSION syntax to switch constraint checking can vary. Verify on your Oracle version.
-- Example shown in transcript (verify in your environment):
-- ALTER SESSION SET CONSTRAINTS = DEFERRED;
-- ALTER SESSION SET CONSTRAINTS = IMMEDIATE;
-- If unsupported, control deferral at constraint creation (DEFERRABLE / INITIALLY DEFERRED) and use transaction boundaries to test behavior.
-- check at commit
ALTER SESSION SET CONSTRAINTS = DEFERRED;

-- check immediately on each DML
ALTER SESSION SET CONSTRAINTS = IMMEDIATE;
```

Behavior:
- When deferred, constraint checks run at commit; violating transaction is rolled back at commit.
- When immediate, violations error at the DML statement time.

---

## Temporary Tables

### Overview
- Temp tables provide session/transaction-scoped storage for intermediate data (e.g., shopping cart).
- Structure (definition) visible to users (global) or only session (private); data is session/private.

### Global Temporary Tables (GTT)
- Created persistently; data is session- or transaction-private.
- ON COMMIT DELETE ROWS: rows cleared at commit (transaction-scoped).
- ON COMMIT PRESERVE ROWS: rows preserved until session end (session-scoped).

Examples:
```sql
-- transaction-scoped
CREATE GLOBAL TEMPORARY TABLE cart_items (
  session_id VARCHAR2(50),
  item_id    NUMBER,
  qty        NUMBER
) ON COMMIT DELETE ROWS;

-- session-scoped
CREATE GLOBAL TEMPORARY TABLE cart_saved (
  session_id VARCHAR2(50),
  item_id    NUMBER,
  qty        NUMBER
) ON COMMIT PRESERVE ROWS;
```

### Private Temporary Tables (PTT)
- Session-private definition and data; name must start with ORA$PTT_.
- Two variants: ON COMMIT DROP DEFINITION (transaction-specific) or PRESERVE DEFINITION (session-specific).

Examples:
```sql
-- private temporary table (transaction)
CREATE PRIVATE TEMPORARY TABLE ORA$PTT_myptt (id NUMBER) ON COMMIT DROP DEFINITION;

-- private temporary table (session)
CREATE PRIVATE TEMPORARY TABLE ORA$PTT_myptt2 (id NUMBER) ON COMMIT PRESERVE DEFINITION;
```

Notes:
- Global temp table definitions are stored in the data dictionary; private temp tables exist only for the session.
- Use temp tables for staged data, session carts, ETL intermediate steps.

---

## External Tables

### Overview
- External tables let Oracle read OS files (text, CSV, custom formats) as read-only tables (or via Data Pump exports).
- Require a DIRECTORY object pointing to the OS path and privileges (READ/WRITE) granted to users.

Create directory & grant:
```sql
CREATE DIRECTORY emp_dir AS '/home/oracle/labs/sql2/emptor';
GRANT READ ON DIRECTORY emp_dir TO app_user;
-- or GRANT READ, WRITE ON DIRECTORY emp_dir TO app_user;
```

### Oracle Loader external table example
- Use ORACLE_LOADER access driver and access parameters to map fields.

Example:
```sql
CREATE TABLE ext_items (
  category_id NUMBER,
  book_id     VARCHAR2(50),
  price       NUMBER(10,2),
  qty         NUMBER
)
ORGANIZATION EXTERNAL (
  TYPE ORACLE_LOADER
  DEFAULT DIRECTORY emp_dir
  ACCESS PARAMETERS (
    RECORDS DELIMITED BY NEWLINE
    FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
    ( category_id, book_id, price, qty )
  )
  LOCATION ('library.items.dat')
)
REJECT LIMIT UNLIMITED;
```

Select from external table:
```sql
SELECT * FROM ext_items;
```

### Oracle Data Pump external table example
- Use to export query results into binary export files and read them as an external table.

Create external using Data Pump files (example):
```sql
CREATE TABLE dept_add_external (
  dept_id   NUMBER,
  dept_name VARCHAR2(100),
  mgr_id    NUMBER,
  loc_id    NUMBER,
  created_on DATE
)
ORGANIZATION EXTERNAL (
  TYPE ORACLE_DATAPUMP
  DEFAULT DIRECTORY emp_dir
  LOCATION ('ORA_EMP4_EXP.DAT','ORA_EMP5_EXP.DAT')
)
REJECT LIMIT UNLIMITED;
```

Notes:
- DBA creates directory and grants access. External table read is subject to file format and access driver.
- External tables are read-only for regular SQL (unless using specific features).

---

## Recycle Bin & PURGE
- Dropped tables go to Recycle Bin (10g+). Space still consumed until purged.
- Recover with FLASHBACK TABLE ... TO BEFORE DROP.
- Permanently remove & free space:
```sql
DROP TABLE big_table PURGE;
```

---

## Practical Tips
- Name constraints explicitly for easier maintenance and readable error messages.
- Use DISABLE/ENABLE with care — disabled constraints can cause inconsistent data if not re-validated.
- For large schema changes, prefer SET UNUSED / DROP UNUSED (where available) or ONLINE operations to minimize blocking.
- Use temporary tables for session-scoped work and ETL staging; prefer GTT with ON COMMIT behavior matching requirements.
- Use external tables for bulk read-only access to OS files or Data Pump exports; grant directory privileges securely.
- Test deferrable vs immediate behavior in transactions before relying on deferred constraints in production.

---

## Exercises (recommended)
1. Add, disable, enable and drop a foreign key constraint; observe behavior on inserts.
2. Create a DEFERRABLE constraint and demonstrate deferred vs immediate checks across a transaction with duplicate keys.
3. Create a global temporary table for a shopping cart; show ON COMMIT DELETE ROWS vs PRESERVE ROWS behavior.
4. Create a DIRECTORY, grant READ, create an external ORACLE_LOADER table for a CSV file and run a SELECT.
5. Drop a large table and recover space by issuing DROP ... PURGE; test flashback recovery (if enabled).

**Course**: Oracle 19c SQL Workshop  
**Last Updated**: November 25, 2025