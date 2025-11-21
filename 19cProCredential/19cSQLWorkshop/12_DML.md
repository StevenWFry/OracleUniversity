# Data Manipulation Language (DML)

## Table of Contents
1. [Overview](#overview)
2. [INSERT](#insert)
3. [INSERT ... SELECT](#insert--select)
4. [UPDATE](#update)
5. [DELETE](#delete)
6. [MERGE (UPSERT)](#merge-upsert)
7. [TRUNCATE vs DELETE](#truncate-vs-delete)
8. [Transaction Control (COMMIT / ROLLBACK / SAVEPOINT)](#transaction-control-commit--rollback--savepoint)
9. [Returning Clauses / Affected Row Info](#returning-clauses--affected-row-info)
10. [Examples and Patterns](#examples-and-patterns)
11. [Best Practices](#best-practices)

---

### Overview
DML statements change table data: INSERT, UPDATE, DELETE, MERGE. Use transactions to control persistence. Be cautious with WHERE clauses to avoid unintentional mass changes.

---

### INSERT

Basic single-row insert:

```sql
INSERT INTO employees (emp_id, first_name, last_name, hire_date, salary, department_id)
VALUES (201, 'Alice', 'Jones', TO_DATE('2024-06-01','YYYY-MM-DD'), 75000, 10);
```

Insert using explicit column list (recommended). Multi-row insert (ANSI):

```sql
INSERT ALL
  INTO employees (emp_id, first_name, last_name, salary, department_id) VALUES (202, 'Bob', 'Lee', 62000, 20)
  INTO employees (emp_id, first_name, last_name, salary, department_id) VALUES (203, 'Cara', 'Ng', 58000, 20)
SELECT * FROM dual;  -- Oracle
```

MySQL multi-row:

```sql
INSERT INTO employees (emp_id, first_name, last_name, salary, department_id)
VALUES (202, 'Bob', 'Lee', 62000, 20),
       (203, 'Cara', 'Ng', 58000, 20);
```

---

### INSERT ... SELECT

Populate a table from query results:

```sql
INSERT INTO employees_archive (emp_id, first_name, last_name, salary, end_date)
SELECT emp_id, first_name, last_name, salary, SYSDATE
FROM employees
WHERE termination_date <= SYSDATE;
```

---

### UPDATE

Update single or multiple rows:

```sql
-- Give 5% raise to department 30
UPDATE employees
SET salary = ROUND(salary * 1.05, 2)
WHERE department_id = 30;
```

Update with subquery:

```sql
-- Set department name from lookup id (example pattern)
UPDATE employees e
SET e.manager_id = (
  SELECT m.emp_id FROM employees m
  WHERE m.first_name = 'John' AND m.last_name = 'Smith'
)
WHERE e.department_id = 40;
```

Use RETURNING (Oracle) to capture changed values (see Returning Clauses).

---

### DELETE

Delete rows safely:

```sql
-- Remove temporary test users
DELETE FROM employees WHERE created_by = 'TEST_USER';
```

Delete with join pattern (Oracle):

```sql
DELETE FROM employees e
WHERE EXISTS (
  SELECT 1 FROM temp_remove tr WHERE tr.emp_id = e.emp_id
);
```

MySQL DELETE with JOIN:

```sql
DELETE e
FROM employees e
JOIN temp_remove tr ON e.emp_id = tr.emp_id;
```

---

### MERGE (UPSERT)

Oracle MERGE for insert-or-update:

```sql
MERGE INTO employees tgt
USING (SELECT 301 AS emp_id, 'Dana' AS first_name, 'Soto' AS last_name, 65000 AS salary FROM dual) src
ON (tgt.emp_id = src.emp_id)
WHEN MATCHED THEN
  UPDATE SET tgt.salary = src.salary
WHEN NOT MATCHED THEN
  INSERT (emp_id, first_name, last_name, salary) VALUES (src.emp_id, src.first_name, src.last_name, src.salary);
```

MySQL equivalent (INSERT ... ON DUPLICATE KEY UPDATE):

```sql
INSERT INTO employees (emp_id, first_name, last_name, salary)
VALUES (301, 'Dana', 'Soto', 65000)
ON DUPLICATE KEY UPDATE salary = VALUES(salary), last_name = VALUES(last_name);
```

Postgres uses INSERT ... ON CONFLICT DO UPDATE.

---

### TRUNCATE vs DELETE

- TRUNCATE TABLE table_name: fast, non-logged (or minimally), removes all rows, cannot be rolled back in some DBs (depends on DB).
- DELETE FROM table_name: row-by-row, logged, can have WHERE clause, can be rolled back.

Example:

```sql
TRUNCATE TABLE employee_temp;
-- vs
DELETE FROM employee_temp WHERE run_id = 20251121;
```

---

### Transaction Control (COMMIT / ROLLBACK / SAVEPOINT)

```sql
-- Start changes (implicit in many clients)
UPDATE accounts SET balance = balance - 100 WHERE acc_id = 101;
UPDATE accounts SET balance = balance + 100 WHERE acc_id = 202;

-- Save partial progress
SAVEPOINT transfer_step1;

-- If all ok
COMMIT;

-- If error
ROLLBACK TO SAVEPOINT transfer_step1;  -- undo to savepoint
ROLLBACK;  -- undo entire transaction
```

In many clients auto-commit is ON by default (MySQL). Use explicit COMMIT when needed.

---

### Returning Clauses / Affected Row Info

Oracle RETURNING INTO (capture values into variables):

```sql
-- Oracle PL/SQL example
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

Oracle UPDATE ... RETURNING:

```sql
UPDATE employees
SET salary = salary * 1.05
WHERE emp_id = 101
RETURNING salary INTO :new_salary;
```

MySQL: use ROW_COUNT() to check number of affected rows; LAST_INSERT_ID() returns auto-generated id.

---

### Examples and Patterns

1. Safe update pattern — verify before commit:

```sql
SELECT COUNT(*) FROM employees WHERE department_id = 99;

UPDATE employees SET department_id = 100 WHERE department_id = 99;

-- Verify then commit
SELECT COUNT(*) FROM employees WHERE department_id = 100;
COMMIT;
```

2. Archiving then deleting:

```sql
INSERT INTO employees_archive (emp_id, first_name, last_name, end_date)
SELECT emp_id, first_name, last_name, SYSDATE
FROM employees
WHERE termination_date IS NOT NULL;

DELETE FROM employees WHERE termination_date IS NOT NULL;
COMMIT;
```

3. Bulk load (Oracle PL/SQL FORALL) — mention only (implementation in PL/SQL):

- Use FORALL for bulk DML in PL/SQL to reduce context switches.

4. Prevent accidental mass updates:

```sql
-- Always run SELECT first
SELECT emp_id, first_name FROM employees WHERE salary < 30000;

-- Then apply UPDATE with same WHERE
UPDATE employees SET salary = 30000 WHERE salary < 30000;
```

---

### Best Practices

- Always include WHERE for UPDATE/DELETE unless you intend to change all rows.
- Test DML with SELECT first.
- Use transactions: COMMIT when ready, ROLLBACK on error.
- Prefer explicit column lists in INSERT.
- Use MERGE or INSERT ... ON DUPLICATE KEY / ON CONFLICT for upsert logic.
- Backup or archive before destructive operations.
- Use RETURNING / LAST_INSERT_ID / ROW_COUNT to verify outcomes.

---

**Course**: Oracle 19c SQL Workshop  
**Last Updated**: November 21, 2025