# Views

## Table of Contents
- [Overview](#overview)
- [Key Concepts](#key-concepts)
- [Creating Views](#creating-views)
- [DML Rules & Limitations](#dml-rules--limitations)
- [WITH CHECK OPTION / WITH READ ONLY](#with-check-option--with-read-only)
- [Viewing View Metadata](#viewing-view-metadata)
- [Dropping Views](#dropping-views)
- [Practical Notes & Best Practices](#practical-notes--best-practices)
- [Exercises (recommended practice)](#exercises-recommended-practice)

## Overview
- Purpose: Provide a logical, named query that presents a subset or transformation of data stored in underlying tables.
- Common usage: Restrict columns/rows for security, simplify complex joins/expressions, present alternative representations (annual vs monthly salary), and provide data independence.

---

## Key Concepts
- A view does not store data; it presents rows from the base table(s).
- Simple view: based on a single table, contains no aggregates/groups, no DISTINCT, no expressions — DML often allowed.
- Complex view: can reference multiple tables, use functions, GROUP BY, aggregates — DML typically not allowed.
- CREATE OR REPLACE VIEW updates a view definition without revoking existing grants.
- View column aliases may be defined either inline or positionally in the CREATE statement.

---

## Creating Views

Simple view (annual salary example):
```sql
CREATE VIEW empsalview AS
SELECT employee_id,
       last_name,
       salary * 12 AS ann_sal
FROM employees;
```

Create view with explicit column aliases:
```sql
CREATE VIEW salvu50 (id_number, name, ann_salary)
AS
SELECT employee_id,
       first_name || ' ' || last_name,
       salary * 12
FROM employees
WHERE department_id = 50;
```

Create or replace view (preserve grants):
```sql
CREATE OR REPLACE VIEW empsalview AS
SELECT employee_id, last_name, salary * 12 AS ann_sal
FROM employees;
```

Complex view with join and aggregates (DML-limited):
```sql
CREATE VIEW dept_salary_stats AS
SELECT d.department_id,
       d.department_name,
       MIN(e.salary) AS min_sal,
       MAX(e.salary) AS max_sal,
       AVG(e.salary) AS avg_sal
FROM employees e
JOIN departments d ON e.department_id = d.department_id
GROUP BY d.department_id, d.department_name;
```

---

## DML Rules & Limitations
- You can usually perform INSERT/UPDATE/DELETE through simple views that map directly to base table columns.
- You cannot perform DML on views that:
  - contain GROUP BY, aggregates, or GROUP functions;
  - use DISTINCT;
  - use ROWNUM (or similar pseudocolumn limiting constructs);
  - include expressions or computed/virtual columns (those columns cannot be the target of updates);
  - omit NOT NULL base-table columns that lack defaults (prevents inserts).
- Updating a virtual column (e.g., ANN_SAL) will fail because it does not exist in the base table.

Example: attempting to update a virtual column (will error)
```sql
UPDATE empsalview SET ann_sal = 100000 WHERE department_id = 90;
-- ORA-xxxxx: virtual column not allowed here
```

Example: updating a base-column via a simple view (works)
```sql
UPDATE empsalview SET department_id = 10
WHERE last_name = 'Kochhar';
-- Changes applied to EMPLOYEES table
```

---

## WITH CHECK OPTION / WITH READ ONLY

WITH CHECK OPTION:
- Purpose: Prevent DML through the view that would produce rows that do not satisfy the view's WHERE clause.
- Named constraint allows identification.

Example (restrict to departments 10 & 90):
```sql
CREATE OR REPLACE VIEW empsal_rw AS
SELECT employee_id, last_name, department_id, salary
FROM employees
WHERE department_id IN (10, 90)
WITH CHECK OPTION;  -- or: WITH CHECK OPTION CONSTRAINT chk_empsal_rw
```

Behavior:
- UPDATE that moves a row outside the allowed department list is rejected:
  "WITH CHECK OPTION where-clause violation"

WITH READ ONLY:
- Purpose: disallow all DML through the view.
```sql
CREATE VIEW empvu10 AS
SELECT * FROM employees WHERE department_id = 10
WITH READ ONLY;
```

---

## Viewing View Metadata
- user_views (Oracle) contains the view text/definition and other metadata.

Examples:
```sql
-- Show the view definition
SELECT view_name, text
FROM user_views
WHERE view_name = 'EMPSALVIEW';

-- List views that reference a table (simple text search)
SELECT view_name FROM user_views WHERE UPPER(text) LIKE '%EMPLOYEES%';
```

---

## Dropping Views
- DROP VIEW view_name;  -- does not affect base table data
- DROP will remove the view object and any grants on it.

Example:
```sql
DROP VIEW empsalview;
```

---

## Practical Notes & Best Practices
- Use views to hide sensitive columns (phone, address, salary) from general users.
- Use views to encapsulate complex joins and calculations so application SQL is simpler.
- Prefer CREATE OR REPLACE VIEW to evolve view definitions without revoking grants.
- Name CHECK OPTION constraints for easier maintenance and error identification.
- Test DML through views in a safe environment to confirm allowed operations.
- Query dictionary views (user_views, all_views, dba_views) to audit view definitions and dependencies.

---

## Exercises (recommended practice)
1. Create a simple view showing employee id, name, and annual salary.
2. Create a complex view that shows department salary statistics (min/max/avg).
3. Create a view with WITH CHECK OPTION that restricts rows and attempt an update that violates the restriction.
4. Query USER_VIEWS to retrieve the text used to create your views.
5. Drop one view and confirm base table data is unchanged.

--- 

**Course**: Oracle 19c SQL Workshop  
**Last Updated**: November 25, 2025