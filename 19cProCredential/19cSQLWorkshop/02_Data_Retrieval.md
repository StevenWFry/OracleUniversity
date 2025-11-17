# Data Retrieval

## Table of Contents
1. [SELECT Statements](#select-statements)
2. [WHERE Clauses](#where-clauses)
3. [ORDER BY (Sorting)](#order-by-sorting)
4. [FETCH and OFFSET (ANSI Pagination)](#fetch-and-offset-ansi-pagination)
5. [LIMIT (MySQL/PostgreSQL)](#limit-mysqlpostgresql)
6. [Subqueries (Subselects)](#subqueries-subselects)
7. [ROWNUM (Oracle)](#rownum-oracle--limiting-and-pagination-patterns)

### SELECT Statements
- **Purpose**: Retrieve data from database tables
- **Syntax**: `SELECT [DISTINCT] column1, column2, ... FROM table_name [WHERE condition] [ORDER BY ...];`
- **Columns**: Can specify specific columns or use * for all columns
- **DISTINCT**: Removes duplicate rows from results

**Examples**:
```sql
-- Select all columns
SELECT * FROM employees;

-- Select specific columns
SELECT emp_id, first_name, last_name, salary FROM employees;

-- Select with arithmetic expression
SELECT emp_id, first_name, salary * 12 AS annual_salary FROM employees;

-- Select with column alias
SELECT first_name AS fname, last_name AS lname FROM employees;

-- DISTINCT values
SELECT DISTINCT department FROM employees;

-- Concatenate columns
SELECT first_name || ' ' || last_name AS full_name FROM employees;
```

### WHERE Clauses
- **Purpose**: Filter rows based on conditions
- **Syntax**: `WHERE condition1 [AND/OR condition2 ...];`

**Examples**:
```sql
-- Single condition
SELECT * FROM employees WHERE department = 'IT';

-- Multiple conditions with AND
SELECT * FROM employees WHERE department = 'IT' AND salary > 60000;

-- Multiple conditions with OR
SELECT * FROM employees WHERE department = 'HR' OR department = 'Finance';

-- Using IN
SELECT * FROM employees WHERE department IN ('HR', 'IT', 'Finance');

-- Using BETWEEN
SELECT * FROM employees WHERE hire_date BETWEEN '2020-01-01' AND '2023-12-31';

-- Using LIKE
SELECT * FROM employees WHERE last_name LIKE 'S%';

-- Using IS NULL
SELECT * FROM employees WHERE commission_pct IS NULL;
```

### ORDER BY (Sorting)
- **Purpose**: Sort result sets by one or more columns or expressions.
- **Syntax**: `SELECT columns FROM table [WHERE ...] ORDER BY expression [ASC|DESC] [, expression2 [ASC|DESC], ...];`
- **Default**: ASC (ascending)
- **Multiple columns**: Primary sort by the first expression, then tie-breakers by subsequent expressions.
- **NULL ordering (Oracle)**: `NULLS FIRST` or `NULLS LAST` (use to control where NULLs appear).
- **Other notes**:
  - You can ORDER BY column position (e.g., `ORDER BY 2`) or column alias.
  - Expressions and functions are allowed in ORDER BY.
  - ORDER BY is applied after SELECT projection and WHERE filtering.

**Examples**:
```sql
-- Simple ascending
SELECT first_name, last_name, salary
FROM employees
ORDER BY last_name;

-- Descending
SELECT first_name, last_name, salary
FROM employees
ORDER BY salary DESC;

-- Multiple columns (department, then salary desc)
SELECT first_name, last_name, department, salary
FROM employees
ORDER BY department, salary DESC;

-- Order by expression / alias (ascending/descending using alias)
SELECT first_name, last_name, salary, salary * 1.1 AS adj_salary
FROM employees
ORDER BY adj_salary DESC, last_name ASC;

-- NULL handling (Oracle)
SELECT first_name, commission_pct
FROM employees
ORDER BY commission_pct DESC NULLS LAST;

-- Order by column position (3 = salary)
SELECT first_name, last_name, salary
FROM employees
ORDER BY 3 DESC;
```

### FETCH and OFFSET (ANSI Pagination)

- **Purpose**: Standard pagination syntax supported by Oracle 12c+ and many other RDBMS.
- **Notes**: ORDER BY should be used for deterministic paging. FETCH/OFFSET are applied after ORDER BY.

**Syntax (Oracle 12c+ / ANSI)**:
```sql
SELECT columns
FROM table
ORDER BY expression
OFFSET <n> ROWS
FETCH NEXT <m> ROWS ONLY;
```

**Examples**:
```sql
-- Top 5 rows by salary (equivalent to FETCH FIRST 5 ROWS ONLY)
SELECT * FROM employees
ORDER BY salary DESC
FETCH FIRST 5 ROWS ONLY;

-- Page 1 (rows 1-10)
SELECT * FROM employees
ORDER BY hire_date DESC
OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY;

-- Page 3 (rows 21-30)
SELECT * FROM employees
ORDER BY hire_date DESC
OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY;

-- Using FETCH with WITH TIES (returns additional rows that tie on ORDER BY expression)
SELECT * FROM employees
ORDER BY salary DESC
FETCH FIRST 5 ROWS WITH TIES;

-- Combined with projection/alias
SELECT emp_id, first_name, last_name, salary,
       ROW_NUMBER() OVER (ORDER BY salary DESC) AS rn
FROM employees
ORDER BY salary DESC
OFFSET 10 ROWS FETCH NEXT 10 ROWS ONLY;
```

### LIMIT (MySQL / PostgreSQL)
- **Purpose**: Simplified pagination/row limiting used by MySQL and PostgreSQL.
- **Notes**: ORDER BY is recommended for deterministic results. LIMIT can be used with or without OFFSET.
- **Syntax**:
  - LIMIT count
  - LIMIT count OFFSET offset
  - MySQL alternate: LIMIT offset, count

**Examples**:
```sql
-- Top 5 rows
SELECT * FROM employees
ORDER BY salary DESC
LIMIT 5;

-- Page 1 (rows 1-10)
SELECT * FROM employees
ORDER BY hire_date DESC
LIMIT 10 OFFSET 0;

-- Page 3 (rows 21-30)
SELECT * FROM employees
ORDER BY hire_date DESC
LIMIT 10 OFFSET 20;

-- MySQL alternate syntax (offset, count)
SELECT * FROM employees
ORDER BY hire_date DESC
LIMIT 20, 10;
```

**Mapping to OFFSET/FETCH**:
- LIMIT 10 OFFSET 20  <=>  OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY

### Subqueries (Subselects)
- **Purpose**: Use a SELECT inside another statement to provide values or filter rows.
- **Types**: Scalar (single value), Single-row, Multi-row, Correlated, Inline view (FROM subquery).

**Examples**:
```sql
-- Simple IN subquery (multi-row)
SELECT * FROM employees
WHERE department_id IN (SELECT department_id FROM departments WHERE location_id = 1700);

-- Correlated subquery (references outer query)
SELECT e.first_name, e.last_name, e.salary, e.department_id
FROM employees e
WHERE e.salary > (
  SELECT AVG(salary)
  FROM employees
  WHERE department_id = e.department_id
);

-- EXISTS (efficient for existence checks)
SELECT e.*
FROM employees e
WHERE EXISTS (
  SELECT 1 FROM bonuses b WHERE b.emp_id = e.emp_id AND b.amount > 1000
);

-- Scalar subquery in SELECT (returns single value per row)
SELECT e.emp_id, e.first_name,
       (SELECT COUNT(*) FROM orders o WHERE o.emp_id = e.emp_id) AS order_count
FROM employees e;

-- Inline view (FROM subquery) with aggregation filtering
SELECT dept, avg_sal
FROM (
  SELECT department_id AS dept, AVG(salary) AS avg_sal
  FROM employees
  GROUP BY department_id
) d
WHERE avg_sal > 70000;
```

### ROWNUM (Oracle) — limiting and pagination patterns
- **Purpose**: Pseudo-column that numbers rows returned by a query (assigned before ORDER BY in a single SELECT).
- **Common use**: Limit top N rows in Oracle 11g and earlier. For ordered top-N, apply ORDER BY inside a subquery.

**Examples**:
```sql
-- Top 5 arbitrary rows (order not guaranteed)
SELECT * FROM employees WHERE ROWNUM <= 5;

-- Correct way to get top 5 by salary (ORDER BY must happen before ROWNUM filter)
SELECT * FROM (
  SELECT * FROM employees ORDER BY salary DESC
) WHERE ROWNUM <= 5;

-- Pagination pattern: rows 6-10 using nested ROWNUM
SELECT * FROM (
  SELECT e.*, ROWNUM rnum
  FROM (
    SELECT * FROM employees ORDER BY salary DESC
  ) e
  WHERE ROWNUM <= 10
)
WHERE rnum > 5;

-- Preferred modern approach: ROW_NUMBER() analytic function (more flexible)
SELECT * FROM (
  SELECT e.*, ROW_NUMBER() OVER (ORDER BY salary DESC) rn
  FROM employees e
) WHERE rn BETWEEN 1 AND 5;
```

Notes:
- ROWNUM is assigned as rows are returned; it cannot be reliably used with ORDER BY in the same SELECT without nesting.
- For advanced pagination and ANSI-compliant numbering, prefer ROW_NUMBER() OVER (...) when available.

---

**Last Updated**: November 17, 2025
**Course**: Oracle 19c SQL Workshop