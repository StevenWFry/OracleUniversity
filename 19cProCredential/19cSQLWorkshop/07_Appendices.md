# Appendices

## Table of Contents
1. [Appendix A: Common SQL Errors and Troubleshooting](#appendix-a-common-sql-errors-and-troubleshooting)
2. [Appendix B: Useful SQL Scripts and Queries](#appendix-b-useful-sql-scripts-and-queries)
3. [Appendix C: SQL Resources and Further Reading](#appendix-c-sql-resources-and-further-reading)
4. [Appendix D: Quick Reference Guides](#appendix-d-quick-reference-guides)
5. [Appendix E: Best Practices](#appendix-e-best-practices)

---

## Appendix A: Common SQL Errors and Troubleshooting

### Syntax Errors

#### Missing Commas
- **Problem**: Forgetting commas between column names or expressions
- **Error**: ORA-00936 (Oracle) or Syntax error (MySQL)

```sql
-- ❌ WRONG: Missing comma between columns
SELECT first_name last_name FROM employees;

-- ✅ CORRECT: Comma added
SELECT first_name, last_name FROM employees;
```

#### Unmatched Parentheses
- **Problem**: Mismatched opening and closing parentheses
- **Error**: ORA-00907 (Oracle) or Syntax error (MySQL)

```sql
-- ❌ WRONG: Missing closing parenthesis
SELECT * FROM employees WHERE (department = 'IT' AND salary > 50000;

-- ✅ CORRECT: Parenthesis closed
SELECT * FROM employees WHERE (department = 'IT' AND salary > 50000);
```

#### Incorrectly Spelled Keywords
- **Problem**: SQL keywords misspelled or case sensitivity issues
- **Error**: ORA-00900 (Oracle) or Syntax error (MySQL)

```sql
-- ❌ WRONG: SELECT misspelled
SELCT first_name FROM employees;

-- ✅ CORRECT: Proper spelling
SELECT first_name FROM employees;

-- ❌ WRONG: WHERE misspelled
SELECT * FROM employees WERE salary > 50000;

-- ✅ CORRECT: Proper spelling
SELECT * FROM employees WHERE salary > 50000;
```

#### Missing Quotes
- **Problem**: String literals not enclosed in single quotes
- **Error**: ORA-00904 (Oracle) or Syntax error (MySQL)

```sql
-- ❌ WRONG: Unquoted string
SELECT * FROM employees WHERE department = IT;

-- ✅ CORRECT: String in quotes
SELECT * FROM employees WHERE department = 'IT';
```

### Logical Errors

#### Incorrect Operator Usage
- **Problem**: Wrong operator (= vs ==, > vs <, AND vs OR)
- **Result**: Unexpected query results

```sql
-- ❌ WRONG: Double equals (not valid in SQL)
SELECT * FROM employees WHERE salary == 50000;

-- ✅ CORRECT: Single equals
SELECT * FROM employees WHERE salary = 50000;

-- ❌ WRONG: AND when OR intended
SELECT * FROM employees WHERE first_name = 'John' AND last_name = 'Smith';
-- Returns only employees named John Smith

-- ✅ CORRECT: OR to find either condition
SELECT * FROM employees WHERE first_name = 'John' OR last_name = 'Smith';
-- Returns employees with first name John OR last name Smith
```

#### Column Name Ambiguity
- **Problem**: Column exists in multiple tables joined together
- **Error**: ORA-00918 (Oracle) or ambiguous column name (MySQL)

```sql
-- ❌ WRONG: Ambiguous 'id' column
SELECT id, name FROM employees e JOIN departments d ON e.dept_id = d.id;

-- ✅ CORRECT: Specify table alias
SELECT e.id AS emp_id, e.name, d.id AS dept_id FROM employees e JOIN departments d ON e.dept_id = d.id;
```

#### NULL Comparisons
- **Problem**: Using = or != with NULL values
- **Result**: No rows returned (NULL comparisons are always UNKNOWN)

```sql
-- ❌ WRONG: NULL comparison with =
SELECT * FROM employees WHERE commission_pct = NULL;
-- Returns no rows

-- ✅ CORRECT: Use IS NULL
SELECT * FROM employees WHERE commission_pct IS NULL;

-- ✅ CORRECT: Use COALESCE in comparison
SELECT * FROM employees WHERE COALESCE(commission_pct, 0) = 0;
```

#### Division by Zero
- **Problem**: Dividing by zero or NULL denominator
- **Error**: ORA-01476 (Oracle) or Division by zero (MySQL)

```sql
-- ❌ WRONG: May cause division by zero
SELECT revenue / costs AS profit_margin FROM quarters;

-- ✅ CORRECT: Use NULLIF to protect
SELECT revenue / NULLIF(costs, 0) AS profit_margin FROM quarters;

-- ✅ CORRECT: Check for zero first
SELECT revenue / CASE WHEN costs > 0 THEN costs ELSE 1 END AS profit_margin FROM quarters;
```

### Troubleshooting Tips

1. **Read error messages carefully**
   - Error number (ORA-XXXXX) points to specific issue
   - Line number indicates where problem occurs

2. **Break down complex queries**
   - Test SELECT with WHERE separately
   - Add one JOIN at a time
   - Verify intermediate results

3. **Use comments to isolate issues**
   ```sql
   -- Comment out sections to find problem
   SELECT first_name, last_name
   FROM employees
   -- WHERE department = 'IT'
   -- AND salary > 50000
   ORDER BY last_name;
   ```

4. **Check data types**
   ```sql
   -- View table structure
   -- Oracle:
   DESC employees;
   
   -- MySQL:
   DESCRIBE employees;
   SHOW COLUMNS FROM employees;
   ```

5. **Verify NULL handling**
   ```sql
   -- Check for NULL values
   SELECT * FROM employees WHERE commission_pct IS NULL;
   SELECT COUNT(*) FROM employees WHERE commission_pct IS NULL;
   ```

6. **Use EXPLAIN PLAN (Oracle) or EXPLAIN (MySQL)**
   ```sql
   -- Oracle: See execution plan
   EXPLAIN PLAN FOR SELECT * FROM employees WHERE salary > 50000;
   SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
   
   -- MySQL: See execution plan
   EXPLAIN SELECT * FROM employees WHERE salary > 50000;
   ```

---

## Appendix B: Useful SQL Scripts and Queries

### 1. Find Duplicate Rows
```sql
-- Find rows that appear more than once
SELECT column1, column2, COUNT(*) AS occurrences
FROM table_name
GROUP BY column1, column2
HAVING COUNT(*) > 1
ORDER BY occurrences DESC;

-- Example: Find duplicate employees
SELECT first_name, last_name, email, COUNT(*) AS count
FROM employees
GROUP BY first_name, last_name, email
HAVING COUNT(*) > 1;
```

### 2. Select Distinct Values
```sql
-- Get unique values
SELECT DISTINCT column1, column2 FROM table_name;

-- Example: Get unique departments
SELECT DISTINCT department FROM employees ORDER BY department;

-- Count distinct values
SELECT COUNT(DISTINCT department) AS unique_departments FROM employees;
```

### 3. Aggregate Functions: COUNT, MAX, MIN, SUM, AVG
```sql
-- Basic aggregates
SELECT 
  COUNT(*) AS total_records,
  COUNT(DISTINCT department) AS unique_departments,
  MAX(salary) AS highest_salary,
  MIN(salary) AS lowest_salary,
  SUM(salary) AS total_salary,
  ROUND(AVG(salary), 2) AS average_salary
FROM employees;

-- With WHERE clause
SELECT AVG(salary) AS avg_it_salary
FROM employees
WHERE department = 'IT';

-- With NULL handling
SELECT 
  COUNT(commission_pct) AS employees_with_commission,
  SUM(COALESCE(commission_pct, 0)) AS total_commission
FROM employees;
```

### 4. GROUP BY with HAVING
```sql
-- Group and filter
SELECT 
  department,
  COUNT(*) AS emp_count,
  ROUND(AVG(salary), 2) AS avg_salary
FROM employees
GROUP BY department
HAVING COUNT(*) > 5 AND AVG(salary) > 50000
ORDER BY avg_salary DESC;

-- Example: Find departments with salary variance
SELECT 
  department,
  COUNT(*) AS emp_count,
  MAX(salary) - MIN(salary) AS salary_range
FROM employees
GROUP BY department
HAVING MAX(salary) - MIN(salary) > 50000;
```

### 5. Basic INNER JOIN
```sql
-- Join two tables on common key
SELECT 
  e.emp_id,
  e.first_name,
  e.last_name,
  d.department_name,
  e.salary
FROM employees e
INNER JOIN departments d ON e.dept_id = d.dept_id
WHERE e.salary > 50000
ORDER BY e.last_name;
```

### 6. LEFT JOIN (Include unmatched rows from left table)
```sql
SELECT 
  e.emp_id,
  e.first_name,
  e.last_name,
  d.department_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.dept_id
ORDER BY e.last_name;
```

### 7. Self-Join (Table joins itself)
```sql
-- Find employees and their managers
SELECT 
  e.first_name AS employee_name,
  m.first_name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.emp_id
ORDER BY e.first_name;
```

### 8. Find Top N Records
```sql
-- Oracle: Using FETCH
SELECT first_name, salary FROM employees
ORDER BY salary DESC
FETCH FIRST 10 ROWS ONLY;

-- MySQL: Using LIMIT
SELECT first_name, salary FROM employees
ORDER BY salary DESC
LIMIT 10;
```

### 9. Find Rows Not in Another Table
```sql
-- Using NOT IN
SELECT emp_id, first_name FROM employees
WHERE emp_id NOT IN (SELECT emp_id FROM department_heads);

-- Using LEFT JOIN
SELECT e.emp_id, e.first_name
FROM employees e
LEFT JOIN department_heads d ON e.emp_id = d.emp_id
WHERE d.emp_id IS NULL;
```

### 10. Date Range Queries
```sql
-- Employees hired in last 6 months
SELECT first_name, last_name, hire_date
FROM employees
WHERE hire_date >= ADD_MONTHS(TRUNC(SYSDATE), -6)  -- Oracle
ORDER BY hire_date DESC;

-- MySQL version
SELECT first_name, last_name, hire_date
FROM employees
WHERE hire_date >= DATE_SUB(CURDATE(), INTERVAL 6 MONTH)
ORDER BY hire_date DESC;

-- Employees hired between two dates
SELECT first_name, last_name, hire_date
FROM employees
WHERE hire_date BETWEEN TO_DATE('2020-01-01', 'YYYY-MM-DD') 
                    AND TO_DATE('2023-12-31', 'YYYY-MM-DD')
ORDER BY hire_date;
```

### 11. String Pattern Matching
```sql
-- Names starting with 'J'
SELECT first_name, last_name FROM employees
WHERE first_name LIKE 'J%'
ORDER BY last_name;

-- Names containing 'man'
SELECT first_name, last_name FROM employees
WHERE last_name LIKE '%man%';

-- Exactly 5 character names
SELECT first_name FROM employees
WHERE first_name LIKE '_____';
```

### 12. UNION: Combine Results from Multiple Queries
```sql
-- Get employees and departments in one result set
SELECT emp_id, first_name AS name, 'Employee' AS type FROM employees
UNION
SELECT dept_id, dept_name, 'Department' FROM departments
ORDER BY type, name;

-- UNION ALL (includes duplicates)
SELECT salary FROM employees WHERE department = 'IT'
UNION ALL
SELECT salary FROM employees WHERE department = 'HR'
ORDER BY salary DESC;
```

### 13. Subqueries
```sql
-- Subquery in WHERE
SELECT first_name, last_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees)
ORDER BY salary DESC;

-- Subquery in FROM (derived table)
SELECT dept, avg_salary FROM (
  SELECT department AS dept, AVG(salary) AS avg_salary
  FROM employees
  GROUP BY department
) sub
WHERE avg_salary > 60000;

-- Subquery with IN
SELECT first_name, last_name
FROM employees
WHERE dept_id IN (SELECT dept_id FROM departments WHERE location = 'New York');
```

### 14. Data Insertion from SELECT
```sql
-- Copy rows from one table to another
INSERT INTO employees_backup (emp_id, first_name, last_name, salary)
SELECT emp_id, first_name, last_name, salary FROM employees
WHERE hire_date < TO_DATE('2020-01-01', 'YYYY-MM-DD');

-- Check affected rows
SELECT * FROM employees_backup;
```

### 15. Update with JOIN
```sql
-- Oracle: Update with JOIN
UPDATE employees e
SET e.salary = e.salary * 1.1
WHERE EXISTS (
  SELECT 1 FROM departments d
  WHERE d.dept_id = e.dept_id AND d.department_name = 'IT'
);

-- MySQL: Update with JOIN
UPDATE employees e
JOIN departments d ON e.dept_id = d.dept_id
SET e.salary = e.salary * 1.1
WHERE d.department_name = 'IT';
```

---

## Appendix C: SQL Resources and Further Reading

### Books
- **"SQL in 10 Minutes, Sams Teach Yourself"** by Ben Forta
  - Quick reference for common SQL tasks
  - Great for beginners
  
- **"Learning SQL"** by Alan Beaulieu
  - Comprehensive SQL fundamentals
  - Good for understanding database concepts

- **"SQL Cookbook"** by Anthony Molinaro
  - Real-world SQL recipes
  - Solutions to common problems

- **"Oracle PL/SQL Programming"** by Steven Feuerstein
  - Advanced Oracle SQL and PL/SQL
  - Professional development guide

### Online Tutorials
- **[W3Schools SQL Tutorial](https://www.w3schools.com/sql/)**
  - Interactive SQL lessons
  - Try-it-yourself editor

- **[SQLZoo](https://sqlzoo.net/)**
  - Interactive SQL practice
  - Progressive difficulty levels

- **[Mode Analytics SQL Tutorial](https://mode.com/sql-tutorial/)**
  - Practical SQL with real datasets
  - Advanced query patterns

- **[Codecademy SQL Course](https://www.codecademy.com/learn/learn-sql)**
  - Hands-on SQL lessons
  - Interactive exercises

### Official Documentation
- **[Oracle SQL Language Reference](https://docs.oracle.com/en/database/oracle/oracle-database/)**
  - Official Oracle SQL documentation
  - Complete function and syntax reference

- **[MySQL Documentation](https://dev.mysql.com/doc/)**
  - Official MySQL reference
  - Version-specific guides

- **[PostgreSQL Documentation](https://www.postgresql.org/docs/)**
  - Comprehensive PostgreSQL guide
  - Advanced features and tuning

### Community Forums
- **[Stack Overflow - SQL Tag](https://stackoverflow.com/questions/tagged/sql)**
  - Active Q&A community
  - Millions of answered questions

- **[Oracle Community](https://community.oracle.com/)**
  - Official Oracle discussion forums
  - Expert advice

- **[MySQL Forums](https://forums.mysql.com/)**
  - MySQL-specific community support
  - Official MySQL team participation

### Development Tools
- **SQL*Plus** (Oracle command-line utility)
- **SQL Developer** (Oracle GUI tool)
- **Toad for Oracle** (Professional Oracle IDE)
- **MySQL Workbench** (Official MySQL GUI tool)
- **DBeaver** (Universal database tool, free and open-source)
- **DataGrip** (JetBrains IDE for databases)

---

## Appendix D: Quick Reference Guides

### SQL Clause Order
```
1. SELECT      - Columns to retrieve
2. FROM        - Source table(s)
3. WHERE       - Row filtering
4. GROUP BY    - Grouping rows
5. HAVING      - Filtering groups
6. ORDER BY    - Sorting results
7. LIMIT/FETCH - Limiting rows
```

### Common Data Types

| Type | Oracle | MySQL | Purpose |
|------|--------|-------|---------|
| Text | VARCHAR2(n) | VARCHAR(n) | Variable-length strings |
| Fixed Text | CHAR(n) | CHAR(n) | Fixed-length strings |
| Numbers | NUMBER(p,s) | DECIMAL(p,s) | Decimal numbers |
| Integers | INTEGER | INT | Whole numbers |
| Date/Time | DATE | DATE | Dates |
| Timestamp | TIMESTAMP | TIMESTAMP | Date with time |
| Large Text | CLOB | LONGTEXT | Large text data |
| Binary Data | BLOB | LONGBLOB | Binary data |
| Boolean | CHAR(1) | BOOLEAN | True/False |

### Operator Precedence (Highest to Lowest)
1. Parentheses `()`
2. Multiplication `*`, Division `/`, Modulus `%`
3. Addition `+`, Subtraction `-`
4. Comparison `=`, `<>`, `<`, `>`, `<=`, `>=`
5. NOT `NOT`
6. AND `AND`
7. OR `OR`

---

## Appendix E: Best Practices

### Performance Best Practices
1. **Use indexes on frequently searched columns**
   ```sql
   -- Create index on employee last_name
   CREATE INDEX idx_emp_lastname ON employees(last_name);
   ```

2. **Avoid SELECT *** when possible**
   ```sql
   -- ❌ Less efficient
   SELECT * FROM employees WHERE salary > 50000;
   
   -- ✅ More efficient
   SELECT emp_id, first_name, last_name, salary FROM employees WHERE salary > 50000;
   ```

3. **Use EXPLAIN to analyze queries**
   ```sql
   -- Oracle
   EXPLAIN PLAN FOR SELECT * FROM employees WHERE emp_id = 101;
   
   -- MySQL
   EXPLAIN SELECT * FROM employees WHERE emp_id = 101;
   ```

4. **Avoid functions on indexed columns in WHERE**
   ```sql
   -- ❌ Disables index on hire_date
   SELECT * FROM employees WHERE YEAR(hire_date) = 2020;
   
   -- ✅ Uses index on hire_date
   SELECT * FROM employees 
   WHERE hire_date >= '2020-01-01' AND hire_date < '2021-01-01';
   ```

### Code Quality Best Practices
1. **Use meaningful aliases**
   ```sql
   -- ❌ Unclear
   SELECT e.a, d.b FROM employees e, departments d;
   
   -- ✅ Clear
   SELECT e.emp_id, d.dept_name FROM employees e, departments d;
   ```

2. **Format queries for readability**
   ```sql
   -- ✅ Well-formatted
   SELECT first_name, last_name, salary
   FROM employees
   WHERE salary > 50000
   ORDER BY last_name;
   ```

3. **Use comments to explain complex logic**
   ```sql
   -- Find employees eligible for bonus (hired > 1 year ago, no salary > 100k)
   SELECT first_name, last_name
   FROM employees
   WHERE MONTHS_BETWEEN(SYSDATE, hire_date) > 12
    AND salary <= 100000
   ORDER BY hire_date;
   ```

4. **Validate NULL handling**
   ```sql
   -- Explicitly handle NULLs
   SELECT first_name, 
          COALESCE(commission_pct, 0) AS commission
   FROM employees;
   ```

5. **Use parameterized queries in applications**
   ```sql
   -- Better: Use bind variables/parameters
   SELECT * FROM employees WHERE emp_id = :emp_id;
   
   -- Avoid: String concatenation (SQL injection risk)
   SELECT * FROM employees WHERE emp_id = ' + empId + ';
   ```

### Data Integrity Best Practices
1. **Always validate input data**
   ```sql
   -- Check for invalid data before inserting
   SELECT COUNT(*) FROM temp_employees
   WHERE salary IS NULL OR hire_date > SYSDATE;
   ```

2. **Use transactions for related changes**
   ```sql
   -- Oracle
   BEGIN
     INSERT INTO employees VALUES (...);
     INSERT INTO emp_history VALUES (...);
     COMMIT;
   EXCEPTION
     WHEN OTHERS THEN ROLLBACK;
   END;
   /
   ```

3. **Document business rules in comments**
   ```sql
   -- Employees must have hire_date <= SYSDATE
   -- Commission_pct range: 0.05 to 0.40
   SELECT * FROM employees
   WHERE hire_date <= SYSDATE
    AND commission_pct BETWEEN 0.05 AND 0.40;
   ```

---

**Last Updated**: November 17, 2025
**Course**: Oracle 19c SQL Workshop