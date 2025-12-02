# Subqueries

## Table of Contents
1. [Introduction to Subqueries](#introduction-to-subqueries)
2. [Types of Subqueries](#types-of-subqueries)
3. [Single-Row Subqueries](#single-row-subqueries)
4. [Multiple-Row Subqueries](#multiple-row-subqueries)
5. [Multiple-Column Subqueries](#multiple-column-subqueries)
6. [Correlated Subqueries](#correlated-subqueries)
7. [Subqueries in Different Clauses](#subqueries-in-different-clauses)
8. [Best Practices](#best-practices)

---

### Introduction to Subqueries

**Definition**: A subquery (also called an inner query or nested query) is a query within another SQL query. The subquery provides data to the main query.

**Purpose**:
- Filter data based on aggregated values
- Compare values against result sets
- Perform multi-step filtering logic
- Reuse query results without creating temporary tables

**Key Characteristics**:
- Enclosed in parentheses
- Executed before (or with) the outer query
- Can return 0, 1, or multiple rows
- Can return 1 or multiple columns
- Inner query result is used by outer query

**Basic Syntax**:
```sql
SELECT column1, column2
FROM table1
WHERE column1 IN (
  SELECT column_from_table2
  FROM table2
  WHERE condition
);
```

---

### Types of Subqueries

**Classification by Return Value**:

| Type | Rows | Columns | Description |
|------|------|---------|-------------|
| **Scalar** | 1 | 1 | Single value; safe in SELECT |
| **Single-Row** | 1 | Multiple | One row; use `=`, `!=`, `<`, `>`, etc. |
| **Multiple-Row** | Many | 1 | Many rows; use `IN`, `ANY`, `ALL` |
| **Multiple-Column** | Many | Many | Multiple columns; use tuple comparison |

**Classification by Dependency**:

| Type | Definition |
|------|-----------|
| **Non-Correlated** | Inner query independent of outer query; executes once |
| **Correlated** | Inner query references columns from outer query; executes for each row |

---

### Single-Row Subqueries

**Purpose**: Return exactly one row and one column, used with comparison operators (=, !=, <, >, <=, >=).

**Characteristics**:
- Must return exactly 1 row
- Returns 1 column
- Uses single-row comparison operators
- Error if subquery returns 0 or multiple rows (ORA-01427, ORA-01422 in Oracle)

**Examples**:

```sql
-- Find employees earning more than the average salary
SELECT first_name, last_name, salary
FROM employees
WHERE salary > (
  SELECT AVG(salary)
  FROM employees
);
```

```sql
-- Find employees in the same department as employee 101
SELECT first_name, last_name, department_id
FROM employees
WHERE department_id = (
  SELECT department_id
  FROM employees
  WHERE emp_id = 101
);
```

```sql
-- Find the employee with the highest salary
SELECT first_name, last_name, salary
FROM employees
WHERE salary = (
  SELECT MAX(salary)
  FROM employees
);
```

```sql
-- Find employees hired on the earliest date
SELECT first_name, last_name, hire_date
FROM employees
WHERE hire_date = (
  SELECT MIN(hire_date)
  FROM employees
);
```

```sql
-- Find employees in departments with average salary > 60000
SELECT first_name, last_name, salary, department_id
FROM employees
WHERE department_id IN (
  SELECT department_id
  FROM employees
  GROUP BY department_id
  HAVING AVG(salary) > 60000
);
```

```sql
-- Use scalar subquery in SELECT clause (returns one value per row)
SELECT first_name, last_name, salary,
       (SELECT AVG(salary) FROM employees) AS company_avg_salary
FROM employees;
```

---

### Multiple-Row Subqueries

**Purpose**: Return multiple rows (0 or more), used with operators like `IN`, `ANY`, `ALL`.

**Operators**:

| Operator | Meaning |
|----------|---------|
| **IN** | Value equals any in subquery result |
| **NOT IN** | Value does not equal any in subquery result |
| **ANY / SOME** | Compares value to ANY value in subquery (=ANY, >ANY, <ANY, >=ANY, <=ANY, !=ANY) |
| **ALL** | Compares value to ALL values in subquery (=ALL, >ALL, <ALL, >=ALL, <=ALL, !=ALL) |
| **EXISTS** | Checks if subquery returns any rows |
| **NOT EXISTS** | Checks if subquery returns no rows |

**Examples with IN**:

```sql
-- Find employees in specific departments
SELECT first_name, last_name, department_id
FROM employees
WHERE department_id IN (
  SELECT department_id
  FROM departments
  WHERE location_id = 1700
);
```

```sql
-- Find employees who are managers
SELECT first_name, last_name
FROM employees
WHERE emp_id IN (
  SELECT manager_id
  FROM employees
);
```

```sql
-- Find employees NOT in specific job titles
SELECT first_name, last_name, job_title
FROM employees
WHERE job_id NOT IN (
  SELECT job_id
  FROM jobs
  WHERE job_title = 'Clerk'
);
```

**Examples with ANY**:

```sql
-- Find employees earning more than ANY employee in department 50
-- (more than the minimum in that department)
SELECT first_name, last_name, salary
FROM employees
WHERE salary > ANY (
  SELECT salary
  FROM employees
  WHERE department_id = 50
);
```

```sql
-- Find employees earning less than ANY employee in department 60
SELECT first_name, last_name, salary, department_id
FROM employees
WHERE salary < ANY (
  SELECT salary
  FROM employees
  WHERE department_id = 60
);
```

**Examples with ALL**:

```sql
-- Find employees earning more than ALL employees in department 50
-- (more than the maximum in that department)
SELECT first_name, last_name, salary
FROM employees
WHERE salary > ALL (
  SELECT salary
  FROM employees
  WHERE department_id = 50
);
```

```sql
-- Find employees earning the same or less than ALL managers' salaries
SELECT first_name, last_name, salary
FROM employees
WHERE salary <= ALL (
  SELECT salary
  FROM employees
  WHERE job_title = 'Manager'
);
```

**Examples with EXISTS**:

```sql
-- Find employees who have made sales (most efficient for existence checks)
SELECT first_name, last_name
FROM employees e
WHERE EXISTS (
  SELECT 1
  FROM orders o
  WHERE o.emp_id = e.emp_id
);
```

```sql
-- Find employees who have NOT made any sales
SELECT first_name, last_name
FROM employees e
WHERE NOT EXISTS (
  SELECT 1
  FROM orders o
  WHERE o.emp_id = e.emp_id
);
```

```sql
-- Find departments that have employees
SELECT department_name
FROM departments d
WHERE EXISTS (
  SELECT 1
  FROM employees e
  WHERE e.department_id = d.department_id
);
```

---

### Multiple-Column Subqueries

**Purpose**: Return multiple columns for tuple (row) comparison.

**Syntax**: Compares multiple column values from outer query against multiple column values from subquery.

**Examples**:

```sql
-- Find employees with the same job_id and manager_id combination as employee 101
SELECT emp_id, first_name, job_id, manager_id
FROM employees
WHERE (job_id, manager_id) IN (
  SELECT job_id, manager_id
  FROM employees
  WHERE emp_id = 101
);
```

```sql
-- Find employees in the same job and department as any employee with salary > 100000
SELECT first_name, last_name, job_id, department_id, salary
FROM employees
WHERE (job_id, department_id) IN (
  SELECT job_id, department_id
  FROM employees
  WHERE salary > 100000
);
```

```sql
-- Find duplicate employee records (same first_name and last_name)
SELECT emp_id, first_name, last_name, COUNT(*)
FROM employees
GROUP BY first_name, last_name
HAVING COUNT(*) > 1
UNION ALL
SELECT emp_id, first_name, last_name, 1
FROM employees
WHERE (first_name, last_name) IN (
  SELECT first_name, last_name
  FROM employees
  GROUP BY first_name, last_name
  HAVING COUNT(*) > 1
);
```

```sql
-- Find orders matching both order_date and customer_id from a specific transaction
SELECT order_id, order_date, customer_id, amount
FROM orders
WHERE (order_date, customer_id) IN (
  SELECT order_date, customer_id
  FROM orders
  WHERE order_id = 5001
);
```

---

### Correlated Subqueries

**Purpose**: Inner query references columns from outer query; executes once for each row in outer query.

**Characteristics**:
- Inner query depends on outer query
- Executes multiple times (once per outer row)
- Generally slower than joins for large datasets
- Useful for row-by-row conditional logic
- Commonly used with EXISTS or comparison operators

**Examples**:

```sql
-- Find employees earning more than average in their department
SELECT first_name, last_name, salary, department_id
FROM employees e
WHERE salary > (
  SELECT AVG(salary)
  FROM employees
  WHERE department_id = e.department_id
);
```

```sql
-- Find employees with the highest salary in their department
SELECT first_name, last_name, salary, department_id
FROM employees e
WHERE salary = (
  SELECT MAX(salary)
  FROM employees
  WHERE department_id = e.department_id
);
```

```sql
-- Find employees who have NOT received any orders
SELECT first_name, last_name, emp_id
FROM employees e
WHERE NOT EXISTS (
  SELECT 1
  FROM orders o
  WHERE o.emp_id = e.emp_id
);
```

```sql
-- Find employees hired after most employees in their department
SELECT first_name, last_name, hire_date, department_id
FROM employees e
WHERE hire_date > (
  SELECT AVG(TRUNC(hire_date))
  FROM employees
  WHERE department_id = e.department_id
);
```

```sql
-- Compare employee salary to department average with hire date context
SELECT e.first_name, e.last_name, e.salary, e.department_id,
       ROUND((
         SELECT AVG(salary)
         FROM employees
         WHERE department_id = e.department_id
       ), 2) AS dept_avg_salary
FROM employees e
WHERE e.salary > (
  SELECT AVG(salary) * 1.1
  FROM employees
  WHERE department_id = e.department_id
    AND hire_date < e.hire_date
);
```

---

### Subqueries in Different Clauses

#### Subqueries in SELECT Clause

**Purpose**: Return a scalar value for each row; must return exactly 1 column and 1 row per outer query row.

```sql
-- Add aggregate data to each employee row
SELECT 
  emp_id,
  first_name,
  salary,
  (SELECT AVG(salary) FROM employees) AS company_avg,
  (SELECT MAX(salary) FROM employees) AS company_max,
  (SELECT COUNT(*) FROM employees) AS total_employees
FROM employees
WHERE emp_id <= 105;
```

```sql
-- Count orders per employee
SELECT 
  first_name,
  last_name,
  (SELECT COUNT(*) FROM orders o WHERE o.emp_id = e.emp_id) AS order_count
FROM employees e
ORDER BY order_count DESC;
```

```sql
-- Show department name with employee info
SELECT 
  e.first_name,
  e.last_name,
  (SELECT d.department_name FROM departments d WHERE d.department_id = e.department_id) AS dept_name
FROM employees e;
```

#### Subqueries in FROM Clause (Derived Tables / Inline Views)

**Purpose**: Treat subquery result as a temporary table.

```sql
-- Find departments with above-average salary
SELECT dept_id, dept_name, avg_salary
FROM (
  SELECT 
    d.department_id AS dept_id,
    d.department_name AS dept_name,
    AVG(e.salary) AS avg_salary
  FROM departments d
  LEFT JOIN employees e ON d.department_id = e.department_id
  GROUP BY d.department_id, d.department_name
) dept_stats
WHERE avg_salary > (
  SELECT AVG(salary) FROM employees
);
```

```sql
-- Find employees whose salary is in the top quartile
SELECT first_name, last_name, salary
FROM employees
WHERE salary >= (
  SELECT salary FROM (
    SELECT DISTINCT salary
    FROM employees
    ORDER BY salary DESC
    FETCH FIRST 25 PERCENT ROWS ONLY  -- Oracle 12c+
  )
  WHERE ROWNUM = 1  -- Get the cutoff value
);
```

```sql
-- Rank employees within department
SELECT 
  emp_id,
  first_name,
  salary,
  dept_rank
FROM (
  SELECT 
    e.emp_id,
    e.first_name,
    e.salary,
    ROW_NUMBER() OVER (PARTITION BY e.department_id ORDER BY e.salary DESC) AS dept_rank
  FROM employees e
)
WHERE dept_rank <= 3;  -- Top 3 earners per department
```

#### Subqueries in WHERE Clause

(Already covered extensively in previous sections)

```sql
-- Standard usage
SELECT * FROM employees
WHERE department_id IN (
  SELECT department_id FROM departments WHERE location_id = 1700
);
```

#### Subqueries in HAVING Clause

**Purpose**: Filter groups based on aggregated values from subqueries.

```sql
-- Find departments with more employees than average
SELECT 
  department_id,
  COUNT(*) AS emp_count
FROM employees
GROUP BY department_id
HAVING COUNT(*) > (
  SELECT AVG(cnt)
  FROM (
    SELECT COUNT(*) AS cnt
    FROM employees
    GROUP BY department_id
  )
);
```

```sql
-- Find job titles where average salary exceeds company average
SELECT 
  job_title,
  ROUND(AVG(salary), 2) AS avg_salary
FROM employees
GROUP BY job_title
HAVING AVG(salary) > (
  SELECT AVG(salary) FROM employees
);
```

---

### Best Practices

1. **Use EXISTS instead of IN for large datasets**
   ```sql
   -- ✅ Better performance with EXISTS
   SELECT * FROM employees e
   WHERE EXISTS (
     SELECT 1 FROM orders o WHERE o.emp_id = e.emp_id
   );
   
   -- ✓ Works but slower with IN on large result sets
   SELECT * FROM employees
   WHERE emp_id IN (SELECT emp_id FROM orders);
   ```

2. **Avoid NOT IN with NULL values**
   ```sql
   -- ❌ WRONG: If subquery returns NULL, result is always NULL
   SELECT * FROM employees
   WHERE emp_id NOT IN (SELECT emp_id FROM orders WHERE cancelled = 'Y');
   -- If cancelled orders include NULL emp_id, NO rows returned
   
   -- ✅ CORRECT: Use NOT EXISTS or handle NULL explicitly
   SELECT * FROM employees e
   WHERE NOT EXISTS (
     SELECT 1 FROM orders o 
     WHERE o.emp_id = e.emp_id AND o.cancelled = 'Y'
   );
   
   -- ✅ Alternative: Use COALESCE to handle NULL
   SELECT * FROM employees
   WHERE emp_id NOT IN (
     SELECT COALESCE(emp_id, -1) FROM orders WHERE cancelled = 'Y'
   );
   ```

3. **Prefer JOINs over subqueries when possible (usually better performance)**
   ```sql
   -- Subquery approach
   SELECT first_name, last_name
   FROM employees
   WHERE department_id IN (SELECT department_id FROM departments WHERE location_id = 1700);
   
   -- JOIN approach (usually faster)
   SELECT DISTINCT e.first_name, e.last_name
   FROM employees e
   INNER JOIN departments d ON e.department_id = d.department_id
   WHERE d.location_id = 1700;
   ```

4. **Use scalar subqueries in SELECT for reporting**
   ```sql
   -- Adds computed values without changing row count
   SELECT 
     first_name,
     salary,
     (SELECT AVG(salary) FROM employees) AS company_avg
   FROM employees;
   ```

5. **Correlated subqueries: Check if you can convert to JOIN**
   ```sql
   -- Correlated subquery (slower)
   SELECT first_name, salary
   FROM employees e
   WHERE salary > (SELECT AVG(salary) FROM employees WHERE department_id = e.department_id);
   
   -- JOIN approach (faster)
   SELECT DISTINCT e.first_name, e.salary
   FROM employees e
   INNER JOIN (
     SELECT department_id, AVG(salary) AS avg_sal
     FROM employees
     GROUP BY department_id
   ) dept_avg ON e.department_id = dept_avg.department_id
   WHERE e.salary > dept_avg.avg_sal;
   ```

6. **Use EXPLAIN to verify subquery performance**
   ```sql
   -- Oracle
   EXPLAIN PLAN FOR SELECT * FROM employees WHERE dept_id IN (SELECT dept_id FROM departments);
   SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
   
   -- MySQL
   EXPLAIN SELECT * FROM employees WHERE dept_id IN (SELECT dept_id FROM departments);
   ```

7. **Document complex subqueries with comments**
   ```sql
   -- Find employees earning above their department average
   -- Inner query: compute average salary per department
   -- Outer query: filter employees exceeding their department average
   SELECT first_name, last_name, salary, department_id
   FROM employees e
   WHERE salary > (
     -- Subquery: average salary for this employee's department
     SELECT AVG(salary)
     FROM employees
     WHERE department_id = e.department_id
   );
   ```

8. **Test subqueries independently**
   ```sql
   -- Always test the inner query first
   SELECT AVG(salary) FROM employees;
   
   -- Then test with the outer query
   SELECT first_name, salary
   FROM employees
   WHERE salary > (SELECT AVG(salary) FROM employees);
   ```

---

## Summary

In this lesson, you should have learned how to:

1. **Define subqueries**
   - Queries nested within other queries
   - Used to provide values to outer query
   - Enclosed in parentheses

2. **Describe the types of problems that subqueries can solve**
   - Multi-level filtering (filter based on aggregates)
   - Comparison against result sets
   - Finding existence of data
   - Row-by-row conditional logic
   - Reusing query results

3. **Identify the types of subqueries**
   - Single-row (1 row, 1 column)
   - Multiple-row (many rows, 1 column)
   - Multiple-column (many rows, many columns)
   - Correlated (references outer query)
   - Non-correlated (independent execution)

4. **Write single-row, multiple-row, multiple-column subqueries**
   - Single-row: Use `=`, `<`, `>`, `!=` operators
   - Multiple-row: Use `IN`, `ANY`, `ALL`, `EXISTS` operators
   - Multiple-column: Compare tuples with `IN` or `EXISTS`
   - Correlated: Reference outer query columns
   - Non-correlated: Execute once independently

---

**Course**: Oracle 19c SQL Workshop
**Last Updated**: November 18, 2025