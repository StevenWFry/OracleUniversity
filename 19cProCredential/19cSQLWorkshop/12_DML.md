# Data Manipulation Language (DML)

## Table of Contents
1. [Overview](#overview)
2. [How to run the examples](#how-to-run-the-examples)
3. [INSERT](#insert)
4. [INSERT ... SELECT](#insert--select)
5. [UPDATE](#update)
6. [DELETE](#delete)
7. [TRUNCATE vs DELETE](#truncate-vs-delete)
8. [MERGE (UPSERT)](#merge-upsert)
9. [Transaction Control (COMMIT / ROLLBACK / SAVEPOINT)](#transaction-control-commit--rollback--savepoint)
10. [Auto-commit Behavior](#auto-commit-behavior)
11. [Row Locks and Read Consistency](#row-locks-and-read-consistency)
12. [RETURNING / Affected-row info](#returning--affected-row-info)
13. [Oracle vs MySQL — DML differences](#oracle-vs-mysql--dml-differences)
14. [Summary Examples](#summary-examples)
15. [Best Practices](#best-practices)

---

### Overview
DML statements change table data: INSERT, UPDATE, DELETE, MERGE. Use transactions to control persistence. Be cautious with WHERE clauses to avoid unintentional mass changes.

---

### How to run the examples
- Open two separate client sessions (Session A and Session B) when examples reference blocking/locking behavior.
- For Oracle use SQL*Plus / SQLcl / SQL Developer; for MySQL use the mysql client or MySQL Workbench.
- Ensure you run statements in the order shown and include semicolons where required.
- Disable client auto-commit when you want explicit transaction control (examples show how).

---

### INSERT

- Purpose: Add new rows to a table.
- Common usage: Single-row INSERT for ad-hoc data entry; multi-row INSERT (VALUES list or INSERT ALL) for bulk loads; INSERT ... SELECT for copying/archiving data.

```sql
INSERT INTO employees (emp_id, first_name, last_name, hire_date, salary, department_id)
VALUES (3001, 'Liam', 'Nguyen', TO_DATE('2025-11-01','YYYY-MM-DD'), 68000, 10);
```

Multi-row (Oracle):
```sql
INSERT ALL
  INTO employees (emp_id, first_name, last_name) VALUES (employees_seq.NEXTVAL, 'C1', 'One')
  INTO employees (emp_id, first_name, last_name) VALUES (employees_seq.NEXTVAL, 'C2', 'Two')
SELECT * FROM dual;
```

Multi-row (MySQL):
```sql
INSERT INTO employees (first_name, last_name)
VALUES ('C1','One'), ('C2','Two');
```

---

### INSERT ... SELECT

- Purpose: Populate a table from the result of a query.
- Common usage: Archive rows before delete, populate staging tables, transform and load data in ETL flows.

```sql
INSERT INTO employees_archive (emp_id, first_name, last_name, end_date)
SELECT emp_id, first_name, last_name, SYSDATE
FROM employees
WHERE termination_date <= SYSDATE;
```

---

### UPDATE

- Purpose: Modify existing rows.
- Common usage: Apply business-rule changes (price/salary adjustments), correct data, propagate derived values. Use WHERE to limit scope; use RETURNING to capture changed values where supported.

```sql
-- Give 5% raise to department 30
UPDATE employees
SET salary = ROUND(salary * 1.05, 2)
WHERE department_id = 30;
```

Limit rows (MySQL):
```sql
UPDATE employees
SET salary = salary + 100
WHERE department_id = 10
LIMIT 10;
```

Limit rows (Oracle):
```sql
UPDATE employees
SET salary = salary + 100
WHERE emp_id IN (
  SELECT emp_id FROM (
    SELECT emp_id FROM employees WHERE department_id = 10 ORDER BY emp_id
  ) WHERE ROWNUM <= 10
);
```

---

### DELETE

- Purpose: Remove specific rows.
- Common usage: Remove test data, purge logical records, delete rows after archiving. Always verify with SELECT first.

```sql
DELETE FROM employees WHERE created_by = 'TEST_USER';
```

Delete by join (MySQL):
```sql
DELETE e
FROM employees e
JOIN temp_remove tr ON e.emp_id = tr.emp_id;
```

Delete by existence (Oracle / ANSI):
```sql
DELETE FROM employees e
WHERE EXISTS (
  SELECT 1 FROM temp_remove tr WHERE tr.emp_id = e.emp_id
);
```

---

### TRUNCATE vs DELETE

- Purpose: TRUNCATE removes all rows quickly; DELETE removes rows selectively.
- Common usage: TRUNCATE for clearing staging/temp tables; DELETE for selective removal and when you need the ability to rollback.

```sql
TRUNCATE TABLE employee_temp;
-- vs
DELETE FROM employee_temp WHERE run_id = 20251121;
```

Note: TRUNCATE is DDL in many DBs (Oracle/MySQL) and issues an implicit COMMIT — it cannot be rolled back in Oracle.

---

### MERGE (UPSERT)

- Purpose: Insert new rows or update existing rows in a single statement.
- Common usage: Synchronize dimension/lookup tables, perform upsert in ETL or sync jobs.

Oracle MERGE:
```sql
MERGE INTO employees tgt
USING (SELECT 301 AS emp_id, 'Dana' AS first_name, 'Soto' AS last_name, 65000 AS salary FROM dual) src
ON (tgt.emp_id = src.emp_id)
WHEN MATCHED THEN
  UPDATE SET tgt.salary = src.salary
WHEN NOT MATCHED THEN
  INSERT (emp_id, first_name, last_name, salary) VALUES (src.emp_id, src.first_name, src.last_name, src.salary);
```

MySQL upsert:
```sql
INSERT INTO employees (emp_id, first_name, last_name, salary)
VALUES (301, 'Dana', 'Soto', 65000)
ON DUPLICATE KEY UPDATE salary = VALUES(salary), last_name = VALUES(last_name);
```

---

### Transaction Control (COMMIT / ROLLBACK / SAVEPOINT)

- Purpose: Control transaction boundaries and allow partial/full undo.
- Common usage: Group multiple DML statements into an atomic unit; use SAVEPOINT for partial rollback during complex operations.

```sql
INSERT INTO employees (emp_id, first_name, last_name) VALUES (1001, 'Amy', 'Wong');

SAVEPOINT sp_after_start;

UPDATE employees SET salary = salary + 500 WHERE department_id = 20;

SAVEPOINT sp_mid;

UPDATE employees SET salary = salary + 300 WHERE emp_id = 101;

-- Partial undo
ROLLBACK TO SAVEPOINT sp_mid;  -- undoes last update only

-- Full undo
ROLLBACK;  -- undoes all uncommitted changes

-- Make permanent
COMMIT;
```

Notes:
- Transactions begin implicitly on first DML in Oracle.
- DDL/DCL statements cause implicit commits.

---

### Auto-commit Behavior

- Purpose: Control whether each statement is committed automatically.
- Common usage: Keep auto-commit ON for simple ad-hoc queries; disable for multi-statement transactions that require rollback capability.

Oracle SQL Developer: Tools > Preferences > Database > Advanced > Enable Auto Commit.

MySQL:
```sql
-- Check current auto-commit status
SELECT @@autocommit;
-- Enable auto-commit
SET autocommit = 1;
-- Disable auto-commit
SET autocommit = 0;
```

PostgreSQL:
- Clients perform implicit auto-commit by default; use explicit BEGIN/COMMIT when grouping statements.

---

### Row Locks and Read Consistency

- Purpose: Lock rows for safe concurrent DML and illustrate MVCC read-consistent behavior.
- Common usage: Use SELECT ... FOR UPDATE to reserve a row for processing; use SKIP LOCKED for worker queues.

Oracle — locking example (Session A / Session B)
```sql
-- Session A: acquire lock (starts transaction implicitly)
SELECT * FROM employees WHERE emp_id = 101 FOR UPDATE;

-- Session B: this will block until Session A commits/rolls back
UPDATE employees SET salary = salary + 500 WHERE emp_id = 101;

-- To raise an error immediately instead of blocking:
SELECT * FROM employees WHERE emp_id = 101 FOR UPDATE NOWAIT;

-- Non-locking read in Session B (read-consistent view; won't see A's uncommitted change)
SELECT emp_id, salary FROM employees WHERE emp_id = 101;

-- Session A completes
COMMIT;
```

MySQL / InnoDB — locking example (Session A / Session B)
```sql
-- Session A:
SET autocommit = 0;
START TRANSACTION;
SELECT * FROM employees WHERE emp_id = 101 FOR UPDATE;

-- Session B:
SELECT * FROM employees WHERE emp_id = 101 FOR UPDATE; -- blocks until A commits

-- Session A:
COMMIT;
```

---

### RETURNING / Affected-row info

- Purpose: Capture generated keys or confirm how many rows were affected by DML.
- Common usage: Use RETURNING in Oracle PL/SQL to get new PKs; use LAST_INSERT_ID()/ROW_COUNT() in MySQL for application logic.

Oracle RETURNING:
```sql
DECLARE
  v_new_id employees.emp_id%TYPE;
BEGIN
  INSERT INTO employees (emp_id, first_name, last_name)
  VALUES (employees_seq.NEXTVAL, 'Eve', 'Kim')
  RETURNING emp_id INTO v_new_id;
  DBMS_OUTPUT.PUT_LINE('Inserted id: ' || v_new_id);
END;
/
```

MySQL last id / affected rows:
```sql
INSERT INTO employees (first_name, last_name) VALUES ('Ben', 'Stone');
SELECT LAST_INSERT_ID() AS new_id;

UPDATE employees SET salary = salary * 1.05 WHERE emp_id = 101;
SELECT ROW_COUNT() AS rows_affected;
```

---

### Oracle vs MySQL — DML differences

- Purpose: Highlight practical differences developers must account for when moving SQL between databases.
- Common usage: Pick the DB-specific pattern that fits your environment (sequences vs AUTO_INCREMENT, MERGE vs ON DUPLICATE KEY, etc).

Examples:
```sql
-- Oracle sequence
INSERT INTO employees (emp_id, first_name) VALUES (employees_seq.NEXTVAL, 'Ana');

-- MySQL auto_increment
INSERT INTO employees (first_name, last_name) VALUES ('Ana','Diaz');
SELECT LAST_INSERT_ID();
```

---

### Summary Examples
(Short examples for the lesson summary items)

```sql
-- INSERT
INSERT INTO employees (emp_id, first_name, last_name) VALUES (4001, 'Zed', 'Smith');

-- UPDATE
UPDATE employees SET salary = salary * 1.05 WHERE department_id = 10;

-- DELETE
DELETE FROM employees WHERE emp_id = 4001;

-- TRUNCATE
TRUNCATE TABLE employee_temp;

-- COMMIT
COMMIT;

-- SAVEPOINT
SAVEPOINT sp_a;

-- ROLLBACK (to savepoint)
ROLLBACK TO SAVEPOINT sp_a;

-- Full ROLLBACK
ROLLBACK;

-- FOR UPDATE (locks)
SELECT * FROM employees WHERE emp_id = 101 FOR UPDATE;
```

---

### Best Practices
- Always test SELECT with the same WHERE before running UPDATE/DELETE.
- Use explicit column lists in INSERT.
- Use transactions (COMMIT/ROLLBACK) for grouped changes.
- Prefer JOINs over correlated subqueries for performance where appropriate.
- Document DDL/DML that causes implicit commits.

---

**Course**: Oracle 19c SQL Workshop  
**Last Updated**: November 21, 2025

---