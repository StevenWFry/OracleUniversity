# Displaying Data from Multiple Tables

## Table of Contents
1. [Introduction to Joins](#introduction-to-joins)
2. [INNER JOIN](#inner-join)
3. [LEFT (OUTER) JOIN](#left-outer-join)
4. [RIGHT (OUTER) JOIN](#right-outer-join)
5. [FULL (OUTER) JOIN](#full-outer-join)
6. [CROSS JOIN](#cross-join)
7. [Self Joins](#self-joins)
8. [Join Conditions and Examples](#join-conditions-and-examples)
9. [Multiple Joins](#multiple-joins)
10. [Join Performance Considerations](#join-performance-considerations)
11. [Best Practices](#best-practices)

---

### Introduction to Joins

**Purpose**: Combine rows from two or more tables based on related columns (foreign keys).

**Key Concepts**:
- **Primary Key**: Unique identifier in the "parent" table
- **Foreign Key**: Column that references a primary key in another table
- **Join Condition**: `ON` clause that specifies how tables are related
- **Types of Joins**: INNER, LEFT, RIGHT, FULL, CROSS, SELF

**Syntax**:
```sql
SELECT columns
FROM table1
[INNER | LEFT | RIGHT | FULL] JOIN table2
ON table1.key_column = table2.key_column
[WHERE conditions]
[ORDER BY ...];
```

**Join Types Comparison**:

| Join Type | Returns | NULL Rows |
|-----------|---------|-----------|
| INNER JOIN | Matching rows only | No |
| LEFT JOIN | All left + matching right | Right NULLs |
| RIGHT JOIN | All right + matching left | Left NULLs |
| FULL OUTER JOIN | All from both tables | Both sides NULLs |
| CROSS

---

### INNER JOIN

**Purpose**: Returns only rows that have matching values in both tables.

**Syntax**:
```sql
SELECT columns
FROM table1
INNER JOIN table2 ON table1.key = table2.key;
-- or (traditional syntax)
SELECT columns
FROM table1, table2
WHERE table1.key = table2.key;
```

**Examples**:

```sql
-- Basic INNER JOIN: employees and departments
SELECT 
  e.emp_id,
  e.first_name,
  e.last_name,
  e.department_id,
  d.department_name,
  d.location
FROM employees e
INNER JOIN departments d
ON e.department_id = d.department_id
ORDER BY e.last_name;
```

```sql
-- INNER JOIN with WHERE clause
SELECT 
  e.emp_id,
  e.first_name,
  e.last_name,
  d.department_name,
  e.salary
FROM employees e
INNER JOIN departments d
ON e.department_id = d.department_id
WHERE e.salary > 60000
ORDER BY e.salary DESC;
```

```sql
-- INNER JOIN with aggregation
SELECT 
  d.department_name,
  COUNT(e.emp_id) AS employee_count,
  ROUND(AVG(e.salary), 2) AS avg_salary
FROM departments d
INNER JOIN employees e
ON d.department_id = e.department_id
GROUP BY d.department_name
ORDER BY employee_count DESC;
```

```sql
-- INNER JOIN with calculations
SELECT 
  e.first_name,
  e.last_name,
  d.department_name,
  e.salary,
  ROUND(e.salary * 1.1, 2) AS salary_with_10pct_raise
FROM employees e
INNER JOIN departments d
ON e.department_id = d.department_id
WHERE e.salary > 50000;
```

---

### LEFT (OUTER) JOIN

**Purpose**: Returns all rows from the left table and matching rows from the right table. Non-matching rows show NULL for right table columns.

**Syntax**:
```sql
SELECT columns
FROM table1
LEFT JOIN table2 ON table1.key = table2.key;
-- or (Oracle syntax)
SELECT columns
FROM table1, table2
WHERE table1.key = table2.key(+);
```

**Examples**:

```sql
-- LEFT JOIN: all employees with their departments (even if no department assigned)
SELECT 
  e.emp_id,
  e.first_name,
  e.last_name,
  d.department_name
FROM employees e
LEFT JOIN departments d
ON e.department_id = d.department_id
ORDER BY e.last_name;
```

```sql
-- LEFT JOIN: find employees without department assignments
SELECT 
  e.emp_id,
  e.first_name,
  e.last_name,
  d.department_name
FROM employees e
LEFT JOIN departments d
ON e.department_id = d.department_id
WHERE d.department_id IS NULL
ORDER BY e.last_name;
```

```sql
-- LEFT JOIN: all departments with employee counts (including empty departments)
SELECT 
  d.department_name,
  COUNT(e.emp_id) AS employee_count,
  COALESCE(ROUND(AVG(e.salary), 2), 0) AS avg_salary
FROM departments d
LEFT JOIN employees e
ON d.department_id = e.department_id
GROUP BY d.department_name
ORDER BY employee_count DESC;
```

```sql
-- LEFT JOIN with multiple conditions
SELECT 
  e.first_name,
  e.last_name,
  j.job_title,
  h.hire_date
FROM employees e
LEFT JOIN jobs j
ON e.job_id = j.job_id
LEFT JOIN job_history h
ON e.emp_id = h.emp_id
ORDER BY e.last_name;
```

```sql
-- LEFT JOIN: show employee with or without commission
SELECT 
  e.emp_id,
  e.first_name,
  e.last_name,
  e.salary,
  COALESCE(c.commission_rate, 0) AS commission_rate,
  ROUND(e.salary * COALESCE(c.commission_rate, 0), 2) AS commission_amount
FROM employees e
LEFT JOIN commissions c
ON e.emp_id = c.emp_id
ORDER BY e.last_name;
```

---

### RIGHT (OUTER) JOIN

**Purpose**: Returns all rows from the right table and matching rows from the left table. Non-matching rows show NULL for left table columns.

**Syntax**:
```sql
SELECT columns
FROM table1
RIGHT JOIN table2 ON table1.key = table2.key;
-- or (Oracle syntax)
SELECT columns
FROM table1, table2
WHERE table1.key(+) = table2.key;
```

**Examples**:

```sql
-- RIGHT JOIN: all departments with their employees
SELECT 
  d.department_name,
  e.first_name,
  e.last_name,
  e.salary
FROM employees e
RIGHT JOIN departments d
ON e.department_id = d.department_id
ORDER BY d.department_name, e.last_name;
```

```sql
-- RIGHT JOIN: find departments with no employees
SELECT 
  d.department_name,
  COUNT(e.emp_id) AS employee_count
FROM employees e
RIGHT JOIN departments d
ON e.department_id = d.department_id
GROUP BY d.department_name
HAVING COUNT(e.emp_id) = 0;
```

```sql
-- RIGHT JOIN: all locations with department information
SELECT 
  l.city,
  l.country,
  d.department_name,
  COUNT(e.emp_id) AS employees_in_location
FROM employees e
RIGHT JOIN departments d
ON e.department_id = d.department_id
RIGHT JOIN locations l
ON d.location_id = l.location_id
GROUP BY l.city, l.country, d.department_name
ORDER BY l.country, l.city;
```

---

### FULL (OUTER) JOIN

**Purpose**: Returns all rows from both tables. Unmatched rows show NULL for columns from the table without the match.

**Syntax**:
```sql
SELECT columns
FROM table1
FULL OUTER JOIN table2 ON table1.key = table2.key;
-- Note: Not supported in MySQL; use UNION of LEFT and RIGHT JOINs
```

**Examples (Oracle/PostgreSQL)**:

```sql
-- FULL OUTER JOIN: all employees and all departments
SELECT 
  e.emp_id,
  e.first_name,
  d.department_id,
  d.department_name
FROM employees e
FULL OUTER JOIN departments d
ON e.department_id = d.department_id
ORDER BY d.department_id, e.last_name;
```

```sql
-- FULL OUTER JOIN: find unmatched employees or departments
SELECT 
  e.emp_id,
  e.first_name,
  d.department_id,
  d.department_name
FROM employees e
FULL OUTER JOIN departments d
ON e.department_id = d.department_id
WHERE e.emp_id IS NULL OR d.department_id IS NULL
ORDER BY COALESCE(e.emp_id, d.department_id);
```

**MySQL Equivalent (using UNION)**:

```sql
-- MySQL: simulate FULL OUTER JOIN
SELECT 
  e.emp_id,
  e.first_name,
  d.department_id,
  d.department_name
FROM employees e
LEFT JOIN departments d
ON e.department_id = d.department_id
UNION
SELECT 
  e.emp_id,
  e.first_name,
  d.department_id,
  d.department_name
FROM employees e
RIGHT JOIN departments d
ON e.department_id = d.department_id;
```

---

### CROSS JOIN

**Purpose**: Produces the Cartesian product (all possible combinations) of two tables.

**Syntax**:
```sql
SELECT columns
FROM table1
CROSS JOIN table2;
-- or (traditional syntax)
SELECT columns
FROM table1, table2;
```

**⚠️ WARNING**: CROSS JOIN produces many rows! (rows in table1 × rows in table2)

**Examples**:

```sql
-- Generate all possible combinations of employees and projects
SELECT 
  e.first_name,
  e.last_name,
  p.project_name
FROM employees e
CROSS JOIN projects p
ORDER BY e.last_name, p.project_name;
```

```sql
-- Create a combination matrix of departments and years/quarters
SELECT 
  d.department_name,
  y.year,
  y.quarter
FROM departments d
CROSS JOIN (
  SELECT 2023 AS year, 'Q1' AS quarter
  UNION ALL
  SELECT 2023, 'Q2'
  UNION ALL
  SELECT 2023, 'Q3'
  UNION ALL
  SELECT 2023, 'Q4'
) y
ORDER BY d.department_name, y.year, y.quarter;
```

```sql
-- Calculate row count from CROSS JOIN
-- 100 employees × 50 projects = 5,000 rows
SELECT COUNT(*) AS potential_assignments
FROM employees e
CROSS JOIN projects p;
```

---


### NATURAL JOIN

**Purpose**: Automatically joins two tables using all columns with the same names in both tables. It can simplify queries but may produce unintended results if tables share multiple column names.

**Syntax**:
```sql
SELECT columns
FROM table1
NATURAL [INNER | LEFT | RIGHT | FULL] JOIN table2;
```

**Notes**:
- NATURAL JOIN finds matching column names and uses them for the join condition.
- It is equivalent to an INNER JOIN on all identically named columns.
- Use with caution: unexpected columns with the same name (e.g., CREATED_BY, UPDATED_BY) can change join behavior.
- Prefer explicit JOIN ... USING(...) or JOIN ... ON ... for clarity in production code.

**Examples**:

```sql
-- Basic NATURAL JOIN: employees and departments share department_id
SELECT 
  emp_id,
  first_name,
  last_name,
  department_name
FROM employees
NATURAL JOIN departments;

-- NATURAL LEFT JOIN: keep all employees, attach department data when available
SELECT 
  emp_id,
  first_name,
  last_name,
  department_name
FROM employees
NATURAL LEFT JOIN departments;

-- NATURAL JOIN between three tables (joins pairwise on common column names)
SELECT 
  emp_id,
  first_name,
  last_name,
  department_name,
  location_id
FROM employees
NATURAL JOIN departments
NATURAL JOIN locations;

-- Equivalent explicit USING for clarity (recommended)
SELECT 
  e.emp_id,
  e.first_name,
  e.last_name,
  d.department_name
FROM employees e
JOIN departments d USING (department_id);

-- Example showing potential pitfall:
-- If both tables share additional common column names, NATURAL JOIN will use them all,
-- possibly filtering out rows unexpectedly.
-- Safer to use USING or ON to be explicit.
```

---

### Self Joins

**Purpose**: Join a table to itself to compare rows or find relationships within the same table.

**Common Uses**:
- Find employees and their managers
- Compare data within the same table
- Find related items in a hierarchy
- Find duplicates or similar records

**Examples**:

```sql
-- Find employees and their managers
SELECT 
  e.emp_id,
  e.first_name AS employee_name,
  e.manager_id,
  m.first_name AS manager_name,
  m.last_name AS manager_last_name
FROM employees e
LEFT JOIN employees m
ON e.manager_id = m.emp_id
ORDER BY e.last_name;
```

```sql
-- Find organizational hierarchy (3 levels)
SELECT 
  e.first_name AS employee,
  m1.first_name AS direct_manager,
  m2.first_name AS director,
  m3.first_name AS VP
FROM employees e
LEFT JOIN employees m1
ON e.manager_id = m1.emp_id
LEFT JOIN employees m2
ON m1.manager_id = m2.emp_id
LEFT JOIN employees m3
ON m2.manager_id = m3.emp_id
ORDER BY m3.first_name, m2.first_name, m1.first_name, e.first_name;
```

```sql
-- Find employees with the same salary
SELECT 
  e1.emp_id,
  e1.first_name,
  e1.salary,
  e2.emp_id,
  e2.first_name,
  e2.salary
FROM employees e1
INNER JOIN employees e2
ON e1.salary = e2.salary
AND e1.emp_id < e2.emp_id  -- Avoid self-join and duplicate pairs
ORDER BY e1.salary DESC, e1.first_name;
```

```sql
-- Find employees hired on the same date
SELECT 
  e1.first_name,
  e1.last_name,
  e1.hire_date,
  e2.first_name,
  e2.last_name
FROM employees e1
INNER JOIN employees e2
ON e1.hire_date = e2.hire_date
AND e1.emp_id < e2.emp_id
ORDER BY e1.hire_date DESC;
```

```sql
-- Find employees in the same department with salary within range
SELECT 
  e1.first_name AS emp1,
  e1.salary AS salary1,
  e2.first_name AS emp2,
  e2.salary AS salary2,
  ABS(e1.salary - e2.salary) AS salary_diff
FROM employees e1
INNER JOIN employees e2
ON e1.department_id = e2.department_id
AND e1.emp_id < e2.emp_id
AND ABS(e1.salary - e2.salary) < 5000
ORDER BY e1.department_id, salary_diff;
```

---

### Join Conditions and Examples

#### Using ON vs WHERE

```sql
-- ON clause: join condition (executed during join)
SELECT e.first_name, d.department_name
FROM employees e
INNER JOIN departments d
ON e.department_id = d.department_id;

-- WHERE clause: filter results (executed after join)
SELECT e.first_name, d.department_name, e.salary
FROM employees e
INNER JOIN departments d
ON e.department_id = d.department_id
WHERE e.salary > 60000;

-- Important: In OUTER JOINs, use ON for join condition, WHERE to exclude rows
SELECT e.first_name, d.department_name
FROM employees e
LEFT JOIN departments d
ON e.department_id = d.department_id
WHERE d.department_id IS NOT NULL;  -- Converts to INNER JOIN behavior
```

#### Composite Join Conditions

```sql
-- Join on multiple columns
SELECT 
  o.order_id,
  o.customer_id,
  o.branch_id,
  c.customer_name,
  c.city
FROM orders o
INNER JOIN customers c
ON o.customer_id = c.customer_id
AND o.branch_id = c.branch_id;
```

#### Join with Functions

```sql
-- Join using UPPER() for case-insensitive match
SELECT 
  e.first_name,
  s.state_name
FROM employees e
INNER JOIN states s
ON UPPER(e.state) = UPPER(s.state_code)
ORDER BY e.last_name;

-- Join using date functions
SELECT 
  e.first_name,
  e.hire_date,
  b.bonus_date
FROM employees e
INNER JOIN bonuses b
ON e.emp_id = b.emp_id
AND YEAR(e.hire_date) = YEAR(b.bonus_date)
ORDER BY e.last_name;
```

#### Join with Expressions

```sql
-- Join on calculated values
SELECT 
  e.first_name,
  e.salary,
  s.salary_grade,
  s.min_salary,
  s.max_salary
FROM employees e
INNER JOIN salary_grades s
ON e.salary BETWEEN s.min_salary AND s.max_salary
ORDER BY e.last_name;
```

---

### Multiple Joins

**Purpose**: Combine data from 3 or more tables to create comprehensive reports.

**Best Practices**:
- Use table aliases to avoid ambiguity
- Join on primary/foreign keys
- Place most restrictive conditions in WHERE clause
- Use INNER JOIN when possible (more efficient than OUTER)

**Examples**:

```sql
-- Three-table JOIN: employees, departments, and locations
SELECT 
  e.emp_id,
  e.first_name,
  e.last_name,
  d.department_name,
  l.city,
  l.country
FROM employees e
INNER JOIN departments d
ON e.department_id = d.department_id
INNER JOIN locations l
ON d.location_id = l.location_id
WHERE l.country = 'USA'
ORDER BY l.city, e.last_name;
```

```sql
-- Four-table JOIN: employees, jobs, departments, and locations
SELECT 
  e.first_name,
  e.last_name,
  j.job_title,
  d.department_name,
  l.city,
  e.salary
FROM employees e
INNER JOIN jobs j
ON e.job_id = j.job_id
INNER JOIN departments d
ON e.department_id = d.department_id
INNER JOIN locations l
ON d.location_id = l.location_id
WHERE e.salary > 70000
ORDER BY e.salary DESC;
```

```sql
-- Complex multi-join with aggregation
SELECT 
  d.department_name,
  j.job_title,
  COUNT(e.emp_id) AS emp_count,
  ROUND(AVG(e.salary), 2) AS avg_salary,
  MAX(e.salary) AS max_salary
FROM employees e
INNER JOIN departments d
ON e.department_id = d.department_id
INNER JOIN jobs j
ON e.job_id = j.job_id
GROUP BY d.department_name, j.job_title
HAVING COUNT(e.emp_id) > 2
ORDER BY d.department_name, avg_salary DESC;
```

```sql
-- Multi-table with LEFT JOINs for optional data
SELECT 
  e.emp_id,
  e.first_name,
  e.last_name,
  d.department_name,
  j.job_title,
  jh.start_date,
  jh.end_date
FROM employees e
INNER JOIN departments d
ON e.department_id = d.department_id
INNER JOIN jobs j
ON e.job_id = j.job_id
LEFT JOIN job_history jh
ON e.emp_id = jh.emp_id
ORDER BY e.last_name, jh.start_date DESC;
```

---

### Join Performance Considerations

#### 1. Index Usage
```sql
-- Ensure foreign key columns are indexed
-- Oracle: Check execution plan
EXPLAIN PLAN FOR
SELECT e.first_name, d.department_name
FROM employees e
INNER JOIN departments d
ON e.department_id = d.department_id;

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);

-- MySQL: Use EXPLAIN
EXPLAIN
SELECT e.first_name, d.department_name
FROM employees e
INNER JOIN departments d
ON e.department_id = d.department_id;
```

#### 2. Join Order
```sql
-- Start with most restrictive table first
-- ✅ GOOD: Filters first, then joins
SELECT e.first_name, d.department_name
FROM employees e
INNER JOIN departments d
ON e.department_id = d.department_id
WHERE e.salary > 100000;

-- Less optimal: May process more rows
SELECT e.first_name, d.department_name
FROM departments d
INNER JOIN employees e
ON d.department_id = e.department_id
WHERE e.salary > 100000;
```

#### 3. Avoid Functions on Join Columns
```sql
-- ❌ POOR: Function on indexed column disables index
SELECT e.first_name, d.department_name
FROM employees e
INNER JOIN departments d
ON UPPER(e.department_code) = UPPER(d.code);

-- ✅ GOOD: Direct comparison uses index
SELECT e.first_name, d.department_name
FROM employees e
INNER JOIN departments d
ON e.department_id = d.department_id;
```

---

### Best Practices

1. **Use meaningful table aliases**
   ```sql
   -- ❌ Unclear
   SELECT a.b, c.d FROM employees a, departments c;
   
   -- ✅ Clear
   SELECT e.first_name, d.department_name FROM employees e, departments d;
   ```

2. **Specify all columns explicitly**
   ```sql
   -- ❌ Avoid SELECT *
   SELECT * FROM employees e INNER JOIN departments d ON e.department_id = d.department_id;
   
   -- ✅ Select needed columns
   SELECT e.emp_id, e.first_name, d.department_name FROM employees e
   INNER JOIN departments d ON e.department_id = d.department_id;
   ```

3. **Put join conditions in ON clause, filters in WHERE**
   ```sql
   -- ✅ CORRECT
   SELECT e.first_name, d.department_name
   FROM employees e
   INNER JOIN departments d ON e.department_id = d.department_id
   WHERE e.salary > 50000;
   ```

4. **Use INNER JOIN unless OUTER is needed**
   ```sql
   -- INNER JOIN is more efficient
   SELECT e.first_name, d.department_name
   FROM employees e
   INNER JOIN departments d ON e.department_id = d.department_id;
   ```

5. **Validate join results with row counts**
   ```sql
   -- Check for unexpected duplicates
   SELECT COUNT(*) FROM employees;  -- 100 rows
   SELECT COUNT(*) FROM employees e
   INNER JOIN departments d ON e.department_id = d.department_id;  -- Should still be ~100
   ```

6. **Use COALESCE for OUTER JOIN NULL handling**
   ```sql
   -- Handle NULLs from outer joins
   SELECT 
     COALESCE(e.emp_id, 'No Employee') AS emp_id,
     COALESCE(d.department_name, 'No Department') AS dept_name
   FROM employees e
   FULL OUTER JOIN departments d ON e.department_id = d.department_id;
   ```

7. **Document complex joins with comments**
   ```sql
   -- Get employees with manager information (3-level hierarchy)
   -- e = employee, m1 = direct manager, m2 = manager's manager
   SELECT 
     e.first_name AS employee,
     m1.first_name AS manager,
     m2.first_name AS director
   FROM employees e
   LEFT JOIN employees m1 ON e.manager_id = m1.emp_id
   LEFT JOIN employees m2 ON m1.manager_id = m2.emp_id;
   ```

---

**Course**: Oracle 19c SQL Workshop
**Last Updated**: November 18, 2025