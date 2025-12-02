# Oracle vs MySQL Comparison

## Table of Contents
1. [Date Differences](#date-differences)
2. [String Functions Comparison](#string-functions-comparison)
3. [Numeric Functions Comparison](#numeric-functions-comparison)
4. [Conversion Functions Comparison](#conversion-functions-comparison)
5. [NULL Handling Comparison](#null-handling-comparison)
6. [Conditional Functions Comparison](#conditional-functions-comparison)
7. [System Tables and Queries](#system-tables-and-queries)

## Date Differences

| Feature | Oracle | MySQL |
|---------|--------|-------|
| **Current Date/Time** | `SYSDATE` | `NOW()` or `CURDATE()` |
| **Current Timestamp** | `SYSTIMESTAMP` | `NOW()` with microseconds |
| **Date Format** | `DD-MON-YY` (default) | `YYYY-MM-DD` (default) |
| **Add Days** | `date_column + 1` | `DATE_ADD(date_column, INTERVAL 1 DAY)` |
| **Subtract Days** | `date_column - 1` | `DATE_SUB(date_column, INTERVAL 1 DAY)` |
| **Add Months** | `ADD_MONTHS(date, n)` | `DATE_ADD(date, INTERVAL n MONTH)` |
| **Date Difference (days)** | `TRUNC(date1 - date2)` | `DATEDIFF(date1, date2)` |
| **Date Difference (months)** | `MONTHS_BETWEEN(date1, date2)` | `TIMESTAMPDIFF(MONTH, date1, date2)` |
| **Extract Year** | `EXTRACT(YEAR FROM date_col)` | `YEAR(date_col)` |
| **Extract Month** | `EXTRACT(MONTH FROM date_col)` | `MONTH(date_col)` |
| **Extract Day** | `EXTRACT(DAY FROM date_col)` | `DAY(date_col)` |
| **Extract Hour** | `EXTRACT(HOUR FROM date_col)` | `HOUR(date_col)` |
| **Last Day of Month** | `LAST_DAY(date)` | `LAST_DAY(date)` |
| **Next Weekday** | `NEXT_DAY(date, 'MONDAY')` | No built-in; use DATE_ADD with DAYOFWEEK |
| **Format Conversion** | `TO_DATE()`, `TO_CHAR()` | `DATE_FORMAT()`, `STR_TO_DATE()` |
| **Truncate Date** | `TRUNC(date, 'MONTH')` | `DATE_TRUNC(date)` (limited) |

**Oracle Examples**:
```sql
SELECT SYSDATE FROM dual;
SELECT SYSTIMESTAMP FROM dual;
SELECT hire_date + 30 FROM employees;
SELECT TRUNC(SYSDATE - hire_date) AS days_employed FROM employees;
SELECT ADD_MONTHS(hire_date, 6) AS review_date FROM employees;
SELECT MONTHS_BETWEEN(SYSDATE, hire_date) AS months_employed FROM employees;
SELECT EXTRACT(YEAR FROM hire_date) FROM employees;
SELECT LAST_DAY(hire_date) FROM employees;
SELECT NEXT_DAY(SYSDATE, 'FRIDAY') FROM dual;
SELECT TO_CHAR(hire_date, 'DD-MON-YYYY') FROM employees;
```

**MySQL Examples**:
```sql
SELECT NOW() FROM dual;
SELECT CURDATE() FROM dual;
SELECT DATE_ADD(hire_date, INTERVAL 30 DAY) FROM employees;
SELECT DATEDIFF(NOW(), hire_date) AS days_employed FROM employees;
SELECT DATE_ADD(hire_date, INTERVAL 6 MONTH) AS review_date FROM employees;
SELECT TIMESTAMPDIFF(MONTH, hire_date, NOW()) AS months_employed FROM employees;
SELECT YEAR(hire_date) FROM employees;
SELECT LAST_DAY(hire_date) FROM employees;
-- Next Friday (DAYOFWEEK: Sun=1, Mon=2, ..., Fri=6)
SELECT DATE_ADD(hire_date, INTERVAL (13 - DAYOFWEEK(hire_date)) % 7 DAY) AS next_friday FROM employees;
SELECT DATE_FORMAT(hire_date, '%d-%b-%Y') FROM employees;
```

---

## String Functions Comparison

| Function | Oracle | MySQL | Purpose |
|----------|--------|-------|---------|
| **Uppercase** | `UPPER(col)` | `UPPER(col)` | Convert to uppercase |
| **Lowercase** | `LOWER(col)` | `LOWER(col)` | Convert to lowercase |
| **Proper Case** | `INITCAP(col)` | `CONCAT(UPPER(LEFT(col,1)), LOWER(SUBSTRING(col,2)))` | First letter uppercase |
| **Substring** | `SUBSTR(col, start, len)` | `SUBSTRING(col, start, len)` | Extract substring |
| **Length** | `LENGTH(col)` | `LENGTH(col)` or `CHAR_LENGTH(col)` | String length |
| **Concatenate** | `col1 \|\| col2` or `CONCAT(col1, col2)` | `CONCAT(col1, col2)` or `col1 + col2` | Join strings |
| **Concat with Separator** | — | `CONCAT_WS(sep, col1, col2, ...)` | Join with separator |
| **Trim Spaces** | `TRIM(col)` | `TRIM(col)` | Remove leading/trailing spaces |
| **Left Trim** | `LTRIM(col)` | `LTRIM(col)` | Remove leading spaces |
| **Right Trim** | `RTRIM(col)` | `RTRIM(col)` | Remove trailing spaces |
| **Pad Left** | `LPAD(col, len, char)` | `LPAD(col, len, char)` | Pad left to length |
| **Pad Right** | `RPAD(col, len, char)` | `RPAD(col, len, char)` | Pad right to length |
| **Find Position** | `INSTR(col, substr)` | `LOCATE(substr, col)` or `POSITION(substr IN col)` | Find substring position |
| **Replace** | `REPLACE(col, old, new)` | `REPLACE(col, old, new)` | Replace substring |
| **Regex Replace** | `REGEXP_REPLACE(col, pattern, repl)` | `REGEXP_REPLACE(col, pattern, repl)` | Replace using regex |

**Oracle Examples**:
```sql
SELECT UPPER(first_name), LOWER(email) FROM employees;
SELECT INITCAP(name) FROM employees;
SELECT SUBSTR(email, 1, INSTR(email, '@') - 1) AS username FROM employees;
SELECT LENGTH(first_name) FROM employees;
SELECT first_name || ' ' || last_name FROM employees;
SELECT TRIM('  hello  ') FROM dual;
SELECT LPAD(emp_id, 5, '0') FROM employees;
SELECT REPLACE(phone, '-', '') FROM employees;
```

**MySQL Examples**:
```sql
SELECT UPPER(first_name), LOWER(email) FROM employees;
SELECT CONCAT(UPPER(LEFT(name,1)), LOWER(SUBSTRING(name,2))) FROM employees;
SELECT SUBSTRING(email, 1, LOCATE('@', email) - 1) AS username FROM employees;
SELECT LENGTH(first_name) FROM employees;
SELECT CONCAT(first_name, ' ', last_name) FROM employees;
SELECT CONCAT_WS(' ', first_name, middle_name, last_name) FROM employees;
SELECT TRIM('  hello  ') FROM dual;
SELECT LPAD(emp_id, 5, '0') FROM employees;
SELECT REPLACE(phone, '-', '') FROM employees;
```

---

## Numeric Functions Comparison

| Function | Oracle | MySQL | Purpose |
|----------|--------|-------|---------|
| **Round** | `ROUND(num, decimals)` | `ROUND(num, decimals)` | Round to decimals |
| **Truncate** | `TRUNC(num, decimals)` | `TRUNCATE(num, decimals)` | Truncate to decimals |
| **Absolute Value** | `ABS(num)` | `ABS(num)` | Absolute value |
| **Ceiling** | `CEIL(num)` | `CEIL(num)` or `CEILING(num)` | Round up |
| **Floor** | `FLOOR(num)` | `FLOOR(num)` | Round down |
| **Modulus** | `MOD(num, divisor)` | `MOD(num, divisor)` or `num % divisor` | Remainder |
| **Power** | `POWER(base, exp)` | `POWER(base, exp)` | Raise to power |
| **Square Root** | `SQRT(num)` | `SQRT(num)` | Square root |
| **Sign** | `SIGN(num)` | `SIGN(num)` | -1, 0, or 1 |
| **Greatest** | `GREATEST(col1, col2, ...)` | `GREATEST(col1, col2, ...)` | Maximum value |
| **Least** | `LEAST(col1, col2, ...)` | `LEAST(col1, col2, ...)` | Minimum value |

**Oracle Examples**:
```sql
SELECT ROUND(salary, 2) FROM employees;
SELECT TRUNC(45.9) FROM dual;
SELECT ABS(-100) FROM dual;
SELECT CEIL(45.2), FLOOR(45.9) FROM dual;
SELECT MOD(emp_id, 2) FROM employees;
SELECT POWER(2, 3) FROM dual;
SELECT SQRT(16) FROM dual;
SELECT SIGN(salary - 50000) FROM employees;
SELECT GREATEST(sal1, sal2, sal3) FROM salaries;
```

**MySQL Examples**:
```sql
SELECT ROUND(salary, 2) FROM employees;
SELECT TRUNCATE(45.9, 0) FROM dual;
SELECT ABS(-100) FROM dual;
SELECT CEIL(45.2), FLOOR(45.9) FROM dual;
SELECT MOD(emp_id, 2), emp_id % 2 FROM employees;
SELECT POWER(2, 3) FROM dual;
SELECT SQRT(16) FROM dual;
SELECT SIGN(salary - 50000) FROM employees;
SELECT GREATEST(sal1, sal2, sal3) FROM salaries;
```

---

## Conversion Functions Comparison

| Function | Oracle | MySQL | Purpose |
|----------|--------|-------|---------|
| **To Character** | `TO_CHAR(date/num, format)` | `DATE_FORMAT(date, format)` or `CAST(num AS CHAR)` | Convert to string |
| **To Date** | `TO_DATE(string, format)` | `STR_TO_DATE(string, format)` | Parse string to date |
| **To Number** | `TO_NUMBER(string, format)` | `CAST(string AS DECIMAL)` | Parse string to number |
| **CAST** | `CAST(expr AS type)` | `CAST(expr AS type)` | ANSI conversion |
| **CONVERT** | — | `CONVERT(expr, type)` | MySQL conversion |

**Oracle Examples**:
```sql
SELECT TO_CHAR(hire_date, 'DD-MON-YYYY') FROM employees;
SELECT TO_CHAR(salary, '$999,999.99') FROM employees;
SELECT TO_DATE('15-JAN-2020', 'DD-MON-YYYY') FROM dual;
SELECT TO_NUMBER('12345') FROM dual;
SELECT CAST(emp_id AS VARCHAR2(10)) FROM employees;
```

**MySQL Examples**:
```sql
SELECT DATE_FORMAT(hire_date, '%d-%b-%Y') FROM employees;
SELECT CAST(salary AS CHAR) FROM employees;
SELECT STR_TO_DATE('15-Jan-2020', '%d-%b-%Y') FROM dual;
SELECT CAST('12345' AS DECIMAL) FROM dual;
SELECT CAST(emp_id AS CHAR(10)) FROM employees;
```

---

## NULL Handling Comparison

| Function | Oracle | MySQL | Purpose |
|----------|--------|-------|---------|
| **COALESCE** | `COALESCE(col1, col2, ...)` | `COALESCE(col1, col2, ...)` | First non-NULL |
| **NVL** | `NVL(col, value)` | — | Replace NULL (Oracle only) |
| **NVL2** | `NVL2(col, not_null, null)` | — | Conditional NULL (Oracle only) |
| **NULLIF** | `NULLIF(col1, col2)` | `NULLIF(col1, col2)` | Return NULL if equal |
| **IFNULL** | — | `IFNULL(col, value)` | Replace NULL (MySQL only) |
| **IF** | — | `IF(condition, true, false)` | Conditional (MySQL only) |
| **IS NULL** | `col IS NULL` | `col IS NULL` | Test for NULL |
| **IS NOT NULL** | `col IS NOT NULL` | `col IS NOT NULL` | Test for not NULL |

**Oracle Examples**:
```sql
SELECT COALESCE(middle_name, last_name, 'Unknown') FROM employees;
SELECT NVL(commission_pct, 0) FROM employees;
SELECT NVL2(commission_pct, 'Has Comm', 'No Comm') FROM employees;
SELECT NULLIF(col1, col2) FROM table1;
SELECT * FROM employees WHERE commission_pct IS NULL;
```

**MySQL Examples**:
```sql
SELECT COALESCE(middle_name, last_name, 'Unknown') FROM employees;
SELECT IFNULL(commission_pct, 0) FROM employees;
SELECT IF(commission_pct IS NULL, 'No Comm', 'Has Comm') FROM employees;
SELECT NULLIF(col1, col2) FROM table1;
SELECT * FROM employees WHERE commission_pct IS NULL;
```

---

## Conditional Functions Comparison

| Feature | Oracle | MySQL |
|---------|--------|-------|
| **Simple Conditional** | `CASE expr WHEN val THEN ... END` | `CASE expr WHEN val THEN ... END` |
| **Searched Conditional** | `CASE WHEN cond THEN ... END` | `CASE WHEN cond THEN ... END` |
| **Oracle Legacy** | `DECODE(expr, val1, res1, ...)` | — |
| **MySQL Conditional** | — | `IF(condition, true, false)` |

**Oracle Examples**:
```sql
SELECT CASE WHEN salary > 100000 THEN 'High' ELSE 'Low' END FROM employees;
SELECT CASE department WHEN 'IT' THEN 'Technology' ELSE 'Other' END FROM employees;
SELECT DECODE(status, 'A', 'Active', 'I', 'Inactive', 'Unknown') FROM employees;
```

**MySQL Examples**:
```sql
SELECT CASE WHEN salary > 100000 THEN 'High' ELSE 'Low' END FROM employees;
SELECT CASE department WHEN 'IT' THEN 'Technology' ELSE 'Other' END FROM employees;
SELECT IF(status = 'A', 'Active', 'Inactive') FROM employees;
```

---

## System Tables and Queries

### Oracle System Views

```sql
-- List all tables in current schema
SELECT table_name FROM user_tables;

-- List all columns in a table
SELECT column_name, data_type, nullable FROM user_tab_columns WHERE table_name = 'EMPLOYEES';

-- List all indexes
SELECT index_name, table_name, uniqueness FROM user_indexes;

-- List all views
SELECT view_name FROM user_views;

-- List all procedures and functions
SELECT object_name, object_type FROM user_objects WHERE object_type IN ('PROCEDURE', 'FUNCTION');

-- View all constraints
SELECT constraint_name, constraint_type, table_name FROM user_constraints;

-- Check current user
SELECT user FROM dual;

-- Check database name
SELECT name FROM v$database;

-- Current session info
SELECT sid, serial#, username FROM v$session WHERE username = 'SCOTT';
```

### MySQL System Queries

```sql
-- List all databases
SHOW DATABASES;

-- Use specific database
USE database_name;

-- List all tables in current database
SHOW TABLES;

-- Show table structure
DESCRIBE employees;
-- or
SHOW COLUMNS FROM employees;

-- List all indexes on a table
SHOW INDEX FROM employees;

-- List all views
SELECT table_name FROM information_schema.tables WHERE table_type = 'VIEW';

-- Check current user
SELECT USER();

-- Check current database
SELECT DATABASE();

-- Show database version
SELECT VERSION();

-- Show table sizes
SELECT table_name, ROUND(((data_length + index_length) / 1024 / 1024), 2) AS size_mb
FROM information_schema.tables
WHERE table_schema = 'your_database_name';
```

---

## Key Differences Summary

| Aspect | Oracle | MySQL |
|--------|--------|-------|
| **Dual Table** | Required for SELECT without FROM | Optional (SELECT works without FROM) |
| **NLS Settings** | Significant impact on date/number formats | Less emphasis on locale settings |
| **String Concatenation** | `\|\|` operator preferred | `CONCAT()` function or `+` |
| **NULL Handling** | More NULL-aware functions (NVL, NVL2) | More flexible IF() usage |
| **Case Sensitivity** | Identifiers are case-insensitive by default | Table names case-sensitive on Unix/Linux |
| **Transactions** | Implicit transactions; explicit COMMIT | Autocommit on by default |
| **Comments** | `--` (line) or `/* */` (block) | `#`, `--`, or `/* */` |
| **Limit Results** | FETCH/OFFSET or ROWNUM | LIMIT clause |
| **Reserved Words** | Extensive list; can be used with quotes | Fewer reserved words |
| **User Variables** | Bind variables (`:var`) | Session variables (`@var`) |

---

**Course**: Oracle 19c SQL Workshop