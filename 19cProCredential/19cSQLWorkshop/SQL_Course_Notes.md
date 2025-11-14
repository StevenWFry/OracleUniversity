# SQL Course Material - Study Notes

## Table of Contents
1. [SQL Fundamentals](#sql-fundamentals)
   - [Comparison Operators](#comparison-operators)
   - [WHERE Clause](#where-clause)
   - [Logical Operators: AND, OR, NOT](#logical-operators-and-or-not)
   - [Rules of Precedence](#rules-of-precedence)
   - [BETWEEN Operator](#between-operator)
   - [IN Operator](#in-operator)
   - [LIKE Operator](#like-operator)
   - [Escape Character](#escape-character)
   - [Q Operator](#q-operator-oracle-string-literals)
   - [Substitution Variables](#substitution-variables-sqlplus--sqlcl)

## SQL Fundamentals
- **Definition**: Structured Query Language for managing relational databases
- **Key Concepts**:
  - Databases and Tables
  - Rows and Columns
  - Primary Keys and Foreign Keys
  - Data Types

### Comparison Operators
- **=** : Equal to
- **!=** or **<>** : Not equal to
- **>** : Greater than
- **<** : Less than
- **>=** : Greater than or equal to
- **<=** : Less than or equal to
- **BETWEEN** : Range comparison (inclusive)
- **IN** : Match any value in a list
- **LIKE** : Pattern matching with wildcards (% and _)
- **IS NULL** : Test for NULL values
- **IS NOT NULL** : Test for non-NULL values

**Examples**:
```sql
SELECT * FROM employees WHERE salary > 50000;
SELECT * FROM employees WHERE department IN ('HR', 'IT', 'Finance');
SELECT * FROM employees WHERE name LIKE 'J%';
SELECT * FROM employees WHERE hire_date BETWEEN '2020-01-01' AND '2023-12-31';
```

### WHERE Clause
- **Purpose**: Filter rows based on specified conditions
- **Syntax**: `SELECT column(s) FROM table WHERE condition;`
- **Logical Operators**:
  - **AND** : All conditions must be true
  - **OR** : At least one condition must be true
  - **NOT** : Negates a condition

**Examples**:
```sql
-- AND operator
SELECT * FROM employees WHERE department = 'IT' AND salary > 60000;

-- OR operator
SELECT * FROM employees WHERE department = 'HR' OR department = 'Finance';

-- NOT operator
SELECT * FROM employees WHERE NOT department = 'IT';

-- Complex conditions
SELECT * FROM employees WHERE (department = 'IT' OR department = 'HR') AND salary > 50000;
```

### Logical Operators: AND, OR, NOT
- **AND**: Returns TRUE only if ALL conditions are TRUE
- **OR**: Returns TRUE if AT LEAST ONE condition is TRUE
- **NOT**: Reverses the result of a condition (returns TRUE if condition is FALSE)
- **Operator Precedence**: NOT > AND > OR (use parentheses to control order)

**AND Examples**:
```sql
-- All conditions must be true
SELECT * FROM employees WHERE department = 'IT' AND salary > 60000;

-- Multiple AND conditions
SELECT * FROM employees WHERE department = 'IT' AND salary > 60000 AND years_employed > 2;

-- AND with different operators
SELECT * FROM employees WHERE hire_date > '2020-01-01' AND status = 'Active' AND department IN ('IT', 'HR');

-- AND with BETWEEN
SELECT * FROM employees WHERE salary BETWEEN 50000 AND 100000 AND department = 'IT' AND status = 'Active';
```

**OR Examples**:
```sql
-- At least one condition must be true
SELECT * FROM employees WHERE department = 'HR' OR department = 'Finance';

-- Multiple OR conditions
SELECT * FROM employees WHERE department = 'IT' OR department = 'HR' OR department = 'Finance';

-- OR with different data types
SELECT * FROM employees WHERE status = 'Inactive' OR hire_date < '2018-01-01' OR salary < 40000;

-- OR matching multiple job titles
SELECT * FROM employees WHERE job_title = 'Manager' OR job_title = 'Senior Manager' OR job_title = 'Director';
```

**NOT Examples**:
```sql
-- Simple NOT condition
SELECT * FROM employees WHERE NOT department = 'IT';
-- Equivalent to: SELECT * FROM employees WHERE department != 'IT';

-- NOT with IN operator
SELECT * FROM employees WHERE NOT status IN ('Inactive', 'On Leave');
-- Equivalent to: SELECT * FROM employees WHERE status NOT IN ('Inactive', 'On Leave');

-- NOT with LIKE
SELECT * FROM employees WHERE NOT email LIKE '%@company.com';
-- Equivalent to: SELECT * FROM employees WHERE email NOT LIKE '%@company.com';

-- NOT with BETWEEN
SELECT * FROM employees WHERE NOT salary BETWEEN 50000 AND 100000;
-- Equivalent to: SELECT * FROM employees WHERE salary NOT BETWEEN 50000 AND 100000;

-- NOT with comparison
SELECT * FROM employees WHERE NOT (department = 'IT' AND salary > 80000);
```

**Combined AND, OR, NOT Examples**:
```sql
-- AND with OR (parentheses control precedence)
SELECT * FROM employees WHERE (department = 'IT' OR department = 'HR') AND salary > 60000;

-- AND with OR and NOT
SELECT * FROM employees WHERE (department = 'IT' OR department = 'HR') AND NOT status = 'Inactive';

-- Complex nested conditions
SELECT * FROM employees WHERE (department = 'IT' AND salary > 70000) OR (department = 'Finance' AND salary > 80000);

-- Multiple operators with NOT
SELECT * FROM employees WHERE NOT (department = 'HR' AND salary < 50000) AND years_employed > 3;

-- Real-world example: Active employees in IT or Finance with salary between ranges
SELECT * FROM employees 
WHERE status = 'Active' 
  AND (
    (department = 'IT' AND salary BETWEEN 60000 AND 100000)
    OR (department = 'Finance' AND salary BETWEEN 70000 AND 120000)
  )
  AND NOT job_title = 'Intern';
```

**Truth Table**:
| Condition1 | AND | Condition2 | Result |
|------------|-----|------------|--------|
| TRUE       | AND | TRUE       | TRUE   |
| TRUE       | AND | FALSE      | FALSE  |
| FALSE      | AND | TRUE       | FALSE  |
| FALSE      | AND | FALSE      | FALSE  |

| Condition1 | OR  | Condition2 | Result |
|------------|-----|------------|--------|
| TRUE       | OR  | TRUE       | TRUE   |
| TRUE       | OR  | FALSE      | TRUE   |
| FALSE      | OR  | TRUE       | TRUE   |
| FALSE      | OR  | FALSE      | FALSE  |

| Condition  | NOT | Result |
|------------|-----|--------|
| TRUE       | NOT | FALSE  |
| FALSE      | NOT | TRUE   |

### Rules of Precedence
- **Definition**: Order in which operators are evaluated in SQL queries
- **Purpose**: Determines which conditions are evaluated first when multiple operators are used
- **Default Order**: NOT > AND > OR (highest to lowest)
- **Override**: Use parentheses () to change evaluation order and improve clarity

**Operator Precedence (Highest to Lowest)**:
1. **Parentheses ( )** - Force evaluation of expressions inside first
2. **NOT** - Logical negation
3. **AND** - Logical conjunction (all conditions must be true)
4. **OR** - Logical disjunction (at least one condition must be true)
5. **Comparison Operators** (=, <>, <, >, <=, >=, IN, BETWEEN, LIKE, IS NULL)
6. **Arithmetic Operators** (+, -, *, /) - evaluated before comparisons in expressions

**Examples Without Parentheses**:
```sql
-- Default: NOT evaluated first, then AND, then OR
-- This reads as: (NOT a) AND b OR c
SELECT * FROM employees 
WHERE NOT department = 'HR' AND salary > 50000 OR status = 'Active';
-- Result: (employees NOT in HR AND salary > 50000) OR (anyone who is Active)

-- Default precedence applied
-- Reads as: a AND (NOT b) OR c
SELECT * FROM employees 
WHERE department = 'IT' AND NOT status = 'Inactive' OR years_employed > 10;
-- Result: (IT department AND NOT inactive) OR (anyone with > 10 years)
```

**Examples With Parentheses (Changing Precedence)**:
```sql
-- Using parentheses to change evaluation order
-- Without parentheses: (a AND b) OR c
-- With parentheses: a AND (b OR c)
SELECT * FROM employees 
WHERE department = 'IT' AND (salary > 60000 OR status = 'Manager');
-- Result: IT department AND (salary > 60000 OR Manager status)

-- Clarifying complex conditions
SELECT * FROM employees 
WHERE (department = 'IT' OR department = 'HR') 
  AND (salary BETWEEN 50000 AND 100000) 
  AND NOT status = 'Inactive';
-- Result: (IT or HR) AND (salary in range) AND (active employees)

-- Multiple levels of nesting
SELECT * FROM employees 
WHERE ((department = 'IT' AND salary > 70000) 
       OR (department = 'Finance' AND salary > 80000))
  AND status = 'Active';
-- Result: (high-paid IT or high-paid Finance) AND (must be active)
```

**Comparison: Without vs With Parentheses**:
```sql
-- Example 1: Without parentheses
-- Evaluated as: (a AND b) OR c
SELECT * FROM employees WHERE department = 'IT' AND salary > 50000 OR years_employed > 15;

-- Same query with explicit parentheses for clarity
SELECT * FROM employees WHERE (department = 'IT' AND salary > 50000) OR years_employed > 15;

-- Different result: forces different grouping
SELECT * FROM employees WHERE department = 'IT' AND (salary > 50000 OR years_employed > 15);
-- This requires IT department AND (salary > 50000 OR years_employed > 15)
```

**Real-World Examples with Precedence**:
```sql
-- Example 1: Employee selection for promotion
-- Precedence: NOT first, then AND, then OR
-- Without clear parentheses, can be ambiguous
SELECT * FROM employees 
WHERE NOT probation_status = 'Active' 
  AND rating >= 4 
  OR tenure > 10;
-- Reads as: (NOT on probation AND high rating) OR (long tenure)

-- Better with parentheses for clarity
SELECT * FROM employees 
WHERE (NOT probation_status = 'Active' AND rating >= 4) 
  OR tenure > 10;

-- Example 2: Complex filtering
SELECT * FROM employees 
WHERE status = 'Active' 
  AND (department IN ('IT', 'HR') AND salary > 60000 
       OR department = 'Executive');
-- Reads as: Active AND ((IT or HR with high salary) OR Executive)

-- Example 3: Avoid confusion with NOT
-- Without parentheses
SELECT * FROM employees WHERE NOT department = 'IT' AND salary > 50000;
-- Reads as: (NOT IT) AND (salary > 50000) - both conditions required

-- To get OR behavior, must use parentheses
SELECT * FROM employees WHERE NOT (department = 'IT' OR salary < 50000);
-- This gets NOT in IT OR NOT low salary = (not IT) AND (high salary)
```

**Precedence Evaluation Step-by-Step**:
```sql
-- Query: WHERE a = 1 AND b = 2 OR NOT c = 3 AND d = 4

-- Step 1: Evaluate NOT
-- WHERE a = 1 AND b = 2 OR (NOT c = 3) AND d = 4

-- Step 2: Evaluate AND (left to right)
-- WHERE (a = 1 AND b = 2) OR ((NOT c = 3) AND d = 4)

-- Step 3: Evaluate OR (left to right)
-- WHERE ((a = 1 AND b = 2) OR ((NOT c = 3) AND d = 4))

-- With explicit parentheses for absolute clarity
SELECT * FROM employees 
WHERE ((a = 1 AND b = 2) OR ((NOT c = 3) AND d = 4));
```

**Best Practices**:
- Always use parentheses in complex queries for clarity
- Group related conditions together
- Avoid relying on default precedence
- Use parentheses even when not strictly necessary for documentation
- Test queries with sample data to verify expected results

### BETWEEN Operator
- **Purpose**: Select values within a specified range (inclusive)
- **Syntax**: `WHERE column BETWEEN value1 AND value2;`
- **Works with**: Numbers, Dates, Text
- **Equivalent to**: `WHERE column >= value1 AND column <= value2`

**Examples**:
```sql
-- Numeric range
SELECT * FROM employees WHERE salary BETWEEN 50000 AND 100000;

-- Date range
SELECT * FROM employees WHERE hire_date BETWEEN '2020-01-01' AND '2023-12-31';

-- Text range (alphabetical)
SELECT * FROM employees WHERE last_name BETWEEN 'A' AND 'M';

-- NOT BETWEEN
SELECT * FROM employees WHERE salary NOT BETWEEN 50000 AND 100000;

-- BETWEEN with multiple conditions
SELECT * FROM employees WHERE salary BETWEEN 50000 AND 100000 AND department = 'IT';
```

### IN Operator
- **Purpose**: Match any value in a specified list
- **Syntax**: `WHERE column IN (value1, value2, value3, ...);`
- **Works with**: Numbers, Text, Dates
- **Equivalent to**: Multiple OR conditions joined together
- **Opposite**: NOT IN (excludes specified values)

**Examples**:
```sql
-- Match multiple departments
SELECT * FROM employees WHERE department IN ('HR', 'IT', 'Finance');

-- Match multiple employee IDs
SELECT * FROM employees WHERE emp_id IN (101, 105, 110, 115);

-- Match multiple job titles
SELECT * FROM employees WHERE job_title IN ('Manager', 'Developer', 'Analyst');

-- NOT IN operator (exclude values)
SELECT * FROM employees WHERE department NOT IN ('HR', 'Finance');

-- IN with subquery
SELECT * FROM employees WHERE emp_id IN (SELECT manager_id FROM employees);

-- IN with multiple conditions
SELECT * FROM employees WHERE department IN ('IT', 'HR') AND salary > 60000;
```

### LIKE Operator
- **Purpose**: Pattern matching using wildcards
- **Syntax**: `WHERE column LIKE pattern;`
- **Wildcards**:
  - **%** : Matches zero or more characters
  - **_** : Matches exactly one character
  - **[char_list]** : Matches any single character in the list (Oracle uses REGEXP_LIKE)
- **Case Sensitivity**: Depends on database collation
- **Opposite**: NOT LIKE (excludes matching patterns)

**Examples**:
```sql
-- Starts with 'J'
SELECT * FROM employees WHERE first_name LIKE 'J%';

-- Ends with 'son'
SELECT * FROM employees WHERE last_name LIKE '%son';

-- Contains 'man' anywhere
SELECT * FROM employees WHERE job_title LIKE '%man%';

-- Exactly 5 characters
SELECT * FROM employees WHERE department LIKE '_____';

-- Second character is 'a'
SELECT * FROM employees WHERE first_name LIKE '_a%';

-- NOT LIKE (exclude patterns)
SELECT * FROM employees WHERE email NOT LIKE '%@company.com';

-- Case-insensitive matching (Oracle)
SELECT * FROM employees WHERE UPPER(last_name) LIKE 'SMITH%';
```

### Escape Character
- **Purpose**: Treat special characters (%, _) as literal characters instead of wildcards
- **Syntax**: `WHERE column LIKE pattern ESCAPE 'escape_char';`
- **Common Escape Characters**: `\`, `|`, `!`, `^`
- **Use Case**: When searching for strings containing % or _ literals

**Examples**:
```sql
-- Search for filenames with % in them
SELECT * FROM files WHERE filename LIKE '%\%%' ESCAPE '\';

-- Search for discount codes containing underscore
SELECT * FROM products WHERE discount_code LIKE '%\_code%' ESCAPE '\';

-- Search for percentage values (e.g., "50%")
SELECT * FROM reports WHERE data LIKE '%50\%%' ESCAPE '\';

-- Using different escape character
SELECT * FROM items WHERE description LIKE '%|_%' ESCAPE '|';

-- Search for literal asterisk or question mark
SELECT * FROM logs WHERE message LIKE '%\*%' ESCAPE '\';
```

### Q Operator (Oracle String Literals)
- **Purpose**: Simplify string literals containing quotes and special characters
- **Syntax**: `q'[string]'` or `q'{string}'` or `q'(string)'` or `q'<string>'`
- **Delimiter**: Can use any non-alphanumeric character as delimiter
- **Advantage**: Avoids escaping single quotes by doubling them
- **Oracle Only**: Specific to Oracle databases

**Examples**:
```sql
-- Using brackets as delimiter
SELECT * FROM employees WHERE job_title = q'[Don't Stop]';

-- Using braces as delimiter
SELECT * FROM employees WHERE department = q'{Human's Resource}';

-- Using parentheses as delimiter
SELECT * FROM employees WHERE email LIKE q'(user@company.com)';

-- Using angle brackets as delimiter
SELECT * FROM employees WHERE notes = q'<It's a test>';

-- Inserting data with apostrophes
INSERT INTO employees (name, comments) VALUES ('John O''Brien', q'[He said, "It's fine"]');

-- Instead of escaping: 'It''s'
SELECT * FROM employees WHERE content = q'[It's working now]';

-- Nesting with different delimiters
SELECT * FROM logs WHERE message = q'<Can't stop won't stop>';
```

### Substitution Variables (SQL*Plus / SQLcl)

- Purpose: Provide run-time values to scripts or prompt users; two flavors: substitution variables (&, &&) and bind variables (:).
- Context: Used in SQL*Plus, SQLcl, SQL Developer script execution.

Basic substitution
```sql
-- Prompts for value each time & is used
SELECT * FROM employees WHERE department_id = &dept_id;

-- Persistent after first prompt (&&)
SELECT * FROM employees WHERE department_id = &&dept_id;
SELECT * FROM departments WHERE department_id = &&dept_id;
```

Positional parameters (script)
```sql
-- script: find_by_dept.sql
SELECT * FROM employees WHERE department_id = &1;

-- Run from SQL*Plus / SQLcl:
@find_by_dept.sql 30
```

ACCEPT / DEFINE
```sql
-- Prompt with ACCEPT and reuse variable
ACCEPT dept PROMPT 'Enter department id: '
SELECT * FROM employees WHERE department_id = &dept;

-- Define without prompting
DEFINE dept = 50
SELECT * FROM employees WHERE department_id = &dept;
```

Control substitution behavior
```sql
SET VERIFY OFF;    -- don't show before/after substitution
SET VERIFY ON;     -- show substitution
SET DEFINE OFF;    -- disable & substitution (useful for scripts with & literals)
```

### VERIFY and ECHO (SQL*Plus / SQLcl)
- SET VERIFY: shows the SQL before and after substitution when ON.
- SET ECHO: when ON, echoes commands in a script as they are executed.
- PROMPT / ECHO: print messages (PROMPT works in SQL*Plus; SQLcl supports ECHO).

**Examples**
```sql
-- VERIFY ON shows substitution details
SET VERIFY ON
DEFINE dept = 10
SELECT * FROM employees WHERE department_id = &dept;
-- Output will show:
-- old  1: SELECT * FROM employees WHERE department_id = &dept
-- new  1: SELECT * FROM employees WHERE department_id = 10

-- Turn VERIFY off to suppress the before/after lines
SET VERIFY OFF

-- Use SET ECHO in scripts to display commands as they run
SET ECHO ON
-- script: show_dept.sql
PROMPT Running department query...
ACCEPT dept PROMPT 'Enter department id: '
SELECT * FROM employees WHERE department_id = &dept;
SET ECHO OFF

-- SQLcl's ECHO prints text (SQL*Plus uses PROMPT)
ECHO 'Script completed successfully.'
```

## Data Retrieval
- SELECT statements
- WHERE clauses
- ORDER BY
- DISTINCT keyword
- Filtering and Sorting

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

**MySQL equivalents**:
```sql
-- MySQL: LIMIT count OFFSET offset
SELECT * FROM employees ORDER BY hire_date DESC LIMIT 10 OFFSET 20;
-- or
SELECT * FROM employees ORDER BY hire_date DESC LIMIT 20, 10;
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
ORDER BY hire_date DESC LIMIT 20, 10;
```

-- Mapping to OFFSET/FETCH:
- LIMIT 10 OFFSET 20  <=>  OFFSET 20 ROWS FETCH NEXT 10 ROWS ONLY

## Data Manipulation
- INSERT statements
- UPDATE statements
- DELETE statements
- COMMIT and ROLLBACK
- Transaction Control

## Database Objects
- Tables
- Views
- Indexes
- Sequences
- Synonyms

## Advanced Topics
- Joins (INNER, LEFT, RIGHT, FULL)
- Subqueries
- Aggregate Functions
- GROUP BY and HAVING
- Set Operations (UNION, INTERSECT, MINUS)

### Date Differences: Oracle vs MySQL

| Feature | Oracle | MySQL |
|---------|--------|-------|
| **Current Date/Time** | `SYSDATE` | `NOW()` or `CURDATE()` |
| **Date Format** | `DD-MON-YY` (default) | `YYYY-MM-DD` (default) |
| **Add Days** | `date_column + 1` | `DATE_ADD(date_column, INTERVAL 1 DAY)` |
| **Subtract Days** | `date_column - 1` | `DATE_SUB(date_column, INTERVAL 1 DAY)` |
| **Date Difference** | `TRUNC(date1 - date2)` | `DATEDIFF(date1, date2)` |
| **Extract Year** | `EXTRACT(YEAR FROM date_col)` | `YEAR(date_col)` or `EXTRACT(YEAR FROM date_col)` |
| **Extract Month** | `EXTRACT(MONTH FROM date_col)` | `MONTH(date_col)` or `EXTRACT(MONTH FROM date_col)` |
| **Extract Day** | `EXTRACT(DAY FROM date_col)` | `DAY(date_col)` or `EXTRACT(DAY FROM date_col)` |
| **Date Arithmetic** | `date + (hours/24)` for time | `DATE_ADD()` with TIME intervals |
| **Format Conversion** | `TO_DATE()`, `TO_CHAR()` | `DATE_FORMAT()`, `STR_TO_DATE()` |

**Examples**:

*Oracle*:
```sql
-- Get current date
SELECT SYSDATE FROM dual;

-- Add 30 days
SELECT hire_date + 30 FROM employees;

-- Calculate days between dates
SELECT TRUNC(SYSDATE - hire_date) AS days_employed FROM employees;

-- Format date
SELECT TO_CHAR(SYSDATE, 'DD-MON-YYYY HH:MI:SS') FROM dual;
```

*MySQL*:
```sql
-- Get current date
SELECT NOW() FROM dual;

-- Add 30 days
SELECT DATE_ADD(hire_date, INTERVAL 30 DAY) FROM employees;

-- Calculate days between dates
SELECT DATEDIFF(NOW(), hire_date) AS days_employed FROM employees;

-- Format date
SELECT DATE_FORMAT(NOW(), '%d-%b-%Y %H:%i:%S') FROM dual; -- MySQL
SELECT DATE_FORMAT(hire_date, '%Y/%m/%d') FROM employees; -- MySQL
SELECT DATE_FORMAT(hire_date, '%W, %M %d, %Y') FROM employees; -- MySQL
```

---

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

### REPLACE()
- **Purpose**: Replace occurrences of a substring with another substring.
- **Syntax (Oracle/MySQL)**: `REPLACE(string, search_string, replace_string)`
- **Notes**: Oracle also provides REGEXP_REPLACE for regex-based and case-insensitive replacements.

```sql
-- Simple replacement (Oracle/MySQL)
SELECT REPLACE(email, '@oldcompany.com', '@newcompany.com') FROM employees;

-- Remove characters (e.g., dashes from phone numbers)
SELECT emp_id, REPLACE(phone, '-', '') AS phone_digits FROM employees;

-- Nested REPLACE for multiple simple substitutions
SELECT REPLACE(REPLACE(address, ',', ''), '.', '') AS address_clean FROM locations;

-- Oracle: regex replace (case-insensitive) to replace 'foo' with 'bar'
SELECT REGEXP_REPLACE(notes, 'foo', 'bar', 1, 0, 'i') AS notes_fixed FROM logs;

-- Oracle: remove all non-alphanumeric characters using regex
SELECT REGEXP_REPLACE(product_code, '[^A-Za-z0-9]', '') AS product_clean FROM products;

-- MySQL: REPLACE usage (no regex)
SELECT REPLACE(description, 'old', 'new') AS description_new FROM products;
```

---

## Single-Row Functions

Single-row functions operate on individual rows and return one result per row. They can be used in SELECT, WHERE, and ORDER BY clauses. Consolidated here are all character, numeric, date, conversion, and conditional single-row functions with examples.

### Character/String Functions

#### INITCAP()
- Purpose: Convert first letter of each word to uppercase, rest to lowercase
- Syntax: `INITCAP(column)`

```sql
SELECT INITCAP(first_name), INITCAP(last_name) FROM employees;
SELECT INITCAP('john smith') FROM dual;
-- Handle hyphenated names
SELECT REPLACE(INITCAP(REPLACE(last_name, '-', ' ')), ' ', '-') AS formatted_last_name
FROM employees;
```

#### UPPER() / LOWER()
- Purpose: Convert strings to uppercase or lowercase
- Syntax: `UPPER(column)` / `LOWER(column)`

```sql
SELECT UPPER(first_name), LOWER(email) FROM employees;
SELECT * FROM employees WHERE UPPER(last_name) = 'SMITH';
```

#### LENGTH()
- Purpose: Return number of characters
- Syntax: `LENGTH(column)` (Oracle) / `LEN(column)` (SQL Server)

```sql
SELECT first_name, LENGTH(first_name) AS name_length FROM employees;
SELECT * FROM employees WHERE LENGTH(last_name) > 5;
```

#### SUBSTR() / SUBSTRING()
- Purpose: Extract substring
- Syntax: `SUBSTR(string, start [, length])` (Oracle) / `SUBSTRING(string, start, length)`

```sql
SELECT SUBSTR(first_name,1,3) AS first3 FROM employees;
SELECT SUBSTR(first_name, -3) AS last3 FROM employees;
-- username before '@'
SELECT SUBSTR(email, 1, INSTR(email, '@') - 1) AS username FROM employees;
```

#### CONCAT() / ||
- Purpose: Concatenate strings
- Syntax: `CONCAT(arg1, arg2, ...)` or `str1 || str2` (Oracle)

```sql
SELECT first_name || ' ' || last_name AS full_name FROM employees;
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM employees; -- MySQL/ANSI
SELECT CONCAT_WS(' ', first_name, middle_name, last_name) AS full_name FROM employees; -- MySQL
```

#### TRIM() / LTRIM() / RTRIM()
- Purpose: Remove leading/trailing characters (default whitespace)

```sql
SELECT TRIM('  hello  ') FROM dual;
SELECT TRIM(BOTH '#' FROM '##value##') FROM dual;
SELECT LTRIM('---note','-') FROM dual;
SELECT RTRIM('note***','*') FROM dual;
SELECT TRIM(first_name) || ' ' || TRIM(last_name) AS full_name FROM employees;
```

#### LPAD() / RPAD()
- Purpose: Pad strings left or right to fixed length

```sql
SELECT LPAD(emp_id,5,'0') AS emp_id_padded FROM employees; -- 123 -> 00123
SELECT RPAD(first_name || ' ' || last_name, 30, ' ') AS name_col FROM employees;
SELECT RPAD(first_name, 20, '.') FROM employees;
```

#### REPLACE() / REGEXP_REPLACE()
- Purpose: Replace substring (REGEXP_REPLACE for regex in Oracle)

```sql
SELECT REPLACE(email, '@oldcompany.com', '@newcompany.com') FROM employees;
SELECT REPLACE(phone, '-', '') AS phone_digits FROM employees;
SELECT REGEXP_REPLACE(notes, 'foo', 'bar', 1, 0, 'i') AS notes_fixed FROM logs; -- Oracle
SELECT REGEXP_REPLACE(product_code, '[^A-Za-z0-9]', '') AS product_clean FROM products;
```

#### INSTR() / LOCATE()
- Purpose: Find position of substring

```sql
SELECT email, INSTR(email, '@') AS at_pos FROM employees; -- Oracle
SELECT INSTR('abracadabra','a',1,3) FROM dual; -- 3rd occurrence
SELECT LOCATE('@', email) AS at_pos_locate FROM employees; -- MySQL
```

### Numeric Functions

#### ROUND()
- Purpose: Round number to specified decimals

```sql
SELECT salary, ROUND(salary, 2) FROM employees;
SELECT ROUND(45.7) FROM dual;
SELECT ROUND(45678, -2) FROM dual; -- 45700
```

#### TRUNC()
- Purpose: Truncate number (no rounding)

```sql
SELECT TRUNC(50000.567, 2) FROM dual;
SELECT TRUNC(45.9) FROM dual;
```

#### ABS(), CEIL(), FLOOR(), MOD(), POWER(), SQRT()
- Purpose: Absolute, ceiling, floor, modulus, power, square root

```sql
SELECT ABS(-100) FROM dual;
SELECT CEIL(45.2) FROM dual;
SELECT FLOOR(45.9) FROM dual;
SELECT MOD(emp_id,2) AS rem FROM employees;
SELECT POWER(1.05, years) AS multiplier FROM investments;
SELECT SQRT(16) FROM dual;
```

### Date Functions

#### SYSDATE / NOW()
- **Purpose**: Current date/time

```sql
SELECT SYSDATE FROM dual; -- Oracle
SELECT NOW() FROM dual;   -- MySQL
```

#### ADD_MONTHS(), MONTHS_BETWEEN(), NEXT_DAY(), LAST_DAY() — additional examples

```sql
-- ADD_MONTHS: probation end date (3 months after hire)
SELECT emp_id, hire_date,
       ADD_MONTHS(hire_date, 3) AS probation_end
FROM employees;

-- ADD_MONTHS: calculate 5-year maturity date
SELECT loan_id, start_date, ADD_MONTHS(start_date, 12 * 5) AS maturity_date
FROM loans;

-- MONTHS_BETWEEN: fractional months employed and whole months
SELECT emp_id,
       MONTHS_BETWEEN(SYSDATE, hire_date)                    AS months_employed_exact,
       TRUNC(MONTHS_BETWEEN(SYSDATE, hire_date))             AS months_employed_whole,
       ROUND(MONTHS_BETWEEN(SYSDATE, hire_date) / 12, 2)     AS years_employed_rounded
FROM employees;

-- MONTHS_BETWEEN: compare two dates (order matters: date1 - date2)
SELECT TO_CHAR(hire_date,'DD-MON-YYYY') d,
       TO_CHAR(SYSDATE,'DD-MON-YYYY') today,
       MONTHS_BETWEEN(SYSDATE, hire_date) AS months_diff
FROM employees
WHERE emp_id = 101;

-- NEXT_DAY: schedule next occurrence of a weekday (e.g., next Friday after SYSDATE)
SELECT SYSDATE AS today,
       NEXT_DAY(SYSDATE, 'FRIDAY') AS next_friday
FROM dual;

-- NEXT_DAY: find next Monday after hire_date (useful for onboarding)
SELECT emp_id, hire_date, NEXT_DAY(hire_date, 'MONDAY') AS onboarding_monday
FROM employees;

-- LAST_DAY: last day of hire month and days remaining in hire month
SELECT emp_id, hire_date,
       LAST_DAY(hire_date) AS hire_month_end,
       TRUNC(LAST_DAY(hire_date) - hire_date) AS days_until_month_end
FROM employees;

-- LAST_DAY: last day of next month
SELECT SYSDATE,
       LAST_DAY(ADD_MONTHS(SYSDATE, 1)) AS last_day_next_month
FROM dual;

-- Combined example: give bonus to employees employed at least 12 months
SELECT emp_id, first_name, hire_date,
       FLOOR(MONTHS_BETWEEN(SYSDATE, hire_date) / 12) AS years_employed
FROM employees
WHERE FLOOR(MONTHS_BETWEEN(SYSDATE, hire_date) / 12) >= 1;

-- MySQL equivalents:
-- ADD_MONTHS -> DATE_ADD(..., INTERVAL n MONTH)
SELECT hire_date, DATE_ADD(hire_date, INTERVAL 3 MONTH) AS probation_end FROM employees;

-- MONTHS_BETWEEN -> TIMESTAMPDIFF(MONTH, start_date, end_date)
SELECT TIMESTAMPDIFF(MONTH, hire_date, NOW()) AS months_employed FROM employees;

-- LAST_DAY exists in MySQL too
SELECT hire_date, LAST_DAY(hire_date) AS hire_month_end FROM employees;

-- NEXT_DAY not built-in in MySQL; compute next weekday (example: next Monday)
-- (DAYOFWEEK: Sunday=1,... Saturday=7)
SELECT hire_date,
       DATE_ADD(hire_date,
                INTERVAL ((9 - DAYOFWEEK(hire_date)) % 7) DAY) AS next_monday
FROM employees;
```

---

## Using Conversion Functions and Conditional Expressions

This section groups conversion functions and conditional expressions used to transform and control output.

### Conversion Functions

- TO_CHAR (Oracle) / DATE_FORMAT (MySQL) — format dates/numbers as strings
- TO_DATE (Oracle) / STR_TO_DATE (MySQL) — parse strings to dates
- TO_NUMBER (Oracle) — parse strings to numbers
- CAST (ANSI) — convert between types

Examples:
```sql
-- Oracle: format date and number
SELECT TO_CHAR(hire_date, 'DD-MON-YYYY') AS hire_fmt,
       TO_CHAR(salary, 'FM999,999.99')       AS salary_fmt
FROM employees;

-- Oracle: parse string to date (with RR two-digit year handling)
SELECT TO_DATE('01-JAN-30', 'DD-MON-RR') FROM dual;

-- Oracle: parse number and use in calculation
SELECT TO_NUMBER('12345') + 100 AS total FROM dual;

-- ANSI CAST examples
SELECT CAST(emp_id AS VARCHAR2(10)) AS emp_id_str FROM employees;
SELECT CAST('2020-01-15' AS DATE) FROM dual; -- Oracle may need TO_DATE in practice

-- MySQL: format & parse dates
SELECT DATE_FORMAT(hire_date, '%d-%b-%Y') AS hire_fmt FROM employees;
SELECT STR_TO_DATE('15-Jan-2020', '%d-%b-%Y') AS hire_date FROM dual;
```

Notes:
- Use TO_CHAR to control locale/formatting (currency, leading zeros).
- Prefer CAST for portability when supported; use Oracle-specific TO_*/DATE_FORMAT when exact behavior required.

### Conditional Expressions

- CASE (ANSI) — flexible searched or simple form
- DECODE (Oracle) — Oracle shorthand for equality comparisons
- COALESCE / NVL / NVL2 / NULLIF — NULL handling
- IFNULL / IF (MySQL) — MySQL equivalents

Examples:
```sql
-- CASE (searched)
SELECT first_name,
       CASE
         WHEN salary >= 100000 THEN 'Executive'
         WHEN salary >= 75000  THEN 'Senior'
         WHEN salary >= 50000  THEN 'Mid-level'
         ELSE 'Junior'
       END AS grade
FROM employees;

-- CASE (simple)
SELECT first_name,
       CASE department
         WHEN 'IT'     THEN 'Information Technology'
         WHEN 'HR'     THEN 'Human Resources'
         ELSE 'Other'
       END AS dept_name
FROM employees;

-- DECODE (Oracle)
SELECT emp_id, DECODE(status, 'A','Active','I','Inactive','Unknown') AS status_text FROM employees;

-- COALESCE / NVL: first non-NULL
SELECT first_name, COALESCE(middle_name, last_name, 'Unknown') AS name_fallback FROM employees;
SELECT first_name, NVL(commission_pct, 0) AS commission FROM employees; -- Oracle

-- NVL2: different results for NULL vs NOT NULL (Oracle)
SELECT emp_id, NVL2(commission_pct, 'Has Commission', 'No Commission') AS comm_status FROM employees;

-- NULLIF: return NULL when equal (avoid divide-by-zero)
SELECT revenue, costs, revenue / NULLIF(costs, 0) AS profit_margin FROM quarters;

-- MySQL IFNULL / IF
SELECT first_name, IFNULL(commission_pct, 0) AS commission FROM employees;
SELECT emp_id, IF(salary > 100000, 'High', 'Normal') AS tier FROM employees;
```

Best practices:
- Use COALESCE for portable multi-argument NULL fallback.
- Use CASE for complex conditional logic; prefer over nested IF/DECODE for readability.
- Use NULLIF to protect calculations (e.g., divide-by-zero).
- Format final output with TO_CHAR / DATE_FORMAT as the last step in SELECT.

---

## Appendix A: Common SQL Errors and Troubleshooting

### Syntax Errors
- **Missing commas** between column names or values.
- **Unmatched parentheses** in expressions or function calls.
- **Incorrectly spelled SQL keywords** (e.g., SELEC instead of SELECT).

**Examples**:
```sql
-- Missing comma
SELECT first_name last_name FROM employees;

-- Unmatched parentheses
SELECT * FROM employees WHERE (department = 'IT' AND salary > 50000;

-- Incorrect keyword
SELEC * FROM employees;
```

### Logical Errors
- **Wrong use of operators** (e.g., using = instead of IN for a list).
- **Misplaced or missing parentheses** affecting order of evaluation.
- **Using AND where OR is needed**, or vice versa.

**Examples**:
```sql
-- Using = instead of IN
SELECT * FROM employees WHERE department_id = (10, 20, 30);

-- Misplaced parentheses
SELECT * FROM employees WHERE department = 'IT' AND (salary > 50000 OR bonus > 1000;

-- AND instead of OR
SELECT * FROM employees WHERE department = 'IT' AND salary > 50000 AND department = 'HR';
```

### Common Warnings
- **Deprecated features**: Using features or syntax that are outdated and may be removed in future versions.
- **Performance warnings**: Such as those for Cartesian products (missing JOIN conditions).

**Examples**:
```sql
-- Deprecated: old-style outer join
SELECT * FROM employees, departments WHERE employees.dept_id = departments.dept_id(+);

-- Performance warning: Cartesian product
SELECT * FROM employees, departments;
```

### Troubleshooting Tips
- **Check SQL syntax**: Use a SQL reference or documentation for the correct syntax.
- **Break down complex queries**: Simplify queries to isolate the part causing the error.
- **Use comments**: Temporarily comment out parts of the query to identify the issue.
- **Consult the database documentation**: For specific functions, operators, or error codes.
- **Ask for help**: When stuck, seek assistance from forums, colleagues, or instructors.

---

## Appendix B: Useful SQL Scripts and Queries

### 1. Find Duplicate Rows
```sql
SELECT column1, column2, COUNT(*)
FROM table_name
GROUP BY column1, column2
HAVING COUNT(*) > 1;
```

### 2. Delete Duplicate Rows (Keep One)
```sql
DELETE FROM table_name
WHERE rowid NOT IN (
  SELECT MIN(rowid)
  FROM table_name
  GROUP BY column1, column2
);
```

### 3. Select Distinct Values
```sql
SELECT DISTINCT column1, column2
FROM table_name;
```

### 4. Count Rows in a Table
```sql
SELECT COUNT(*) FROM table_name;
```

### 5. Find Maximum/Minimum Value
```sql
SELECT MAX(column_name), MIN(column_name)
FROM table_name;
```

### 6. Calculate Average Value
```sql
SELECT AVG(column_name) FROM table_name;
```

### 7. Sum Values
```sql
SELECT SUM(column_name) FROM table_name;
```

### 8. Group By Example
```sql
SELECT column1, COUNT(*)
FROM table_name
GROUP BY column1
ORDER BY COUNT(*) DESC;
```

### 9. Having Clause Example
```sql
SELECT column1, SUM(column2)
FROM table_name
GROUP BY column1
HAVING SUM(column2) > 1000;
```

### 10. Join Example
```sql
SELECT a.column1, b.column2
FROM table1 a
JOIN table2 b ON a.common_field = b.common_field
WHERE a.condition_field = 'some_value';
```

---

## Appendix C: SQL Resources and Further Reading

- **Books**:
  - "SQL in 10 Minutes, Sams Teach Yourself" by Ben Forta
  - "Learning SQL" by Alan Beaulieu
  - "SQL Cookbook" by Anthony Molinaro

- **Online Tutorials**:
  - W3Schools SQL Tutorial
  - SQLZoo
  - Mode Analytics SQL Tutorial

- **Documentation**:
  - Oracle SQL Language Reference
  - MySQL Documentation
  - PostgreSQL Documentation

- **Forums and Q&A Sites**:
  - Stack Overflow
  - Oracle Community
  - MySQL Forums

---

**Last Updated**: November 14, 2025
**Course**: Oracle 19c SQL Workshop