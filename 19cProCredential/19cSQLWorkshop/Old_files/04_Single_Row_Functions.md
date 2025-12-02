# Single-Row Functions

## Table of Contents
1. [Character/String Functions](#characterstring-functions)
2. [Numeric Functions](#numeric-functions)
3. [Date Functions](#date-functions)
4. [Date Arithmetic](#date-arithmetic)

Single-row functions operate on individual rows and return one result per row. They can be used in SELECT, WHERE, and ORDER BY clauses.

### Character/String Functions

#### INITCAP()
- Purpose: Convert first letter of each word to uppercase, rest to lowercase
- Syntax: `INITCAP(column)`

```sql
SELECT INITCAP(first_name), INITCAP(last_name) FROM employees;
-- Result: John, Smith

SELECT INITCAP('john smith') FROM dual;
-- Result: John Smith

-- Handle hyphenated names
SELECT REPLACE(INITCAP(REPLACE(last_name, '-', ' ')), ' ', '-') AS formatted_last_name
FROM employees;
-- Example: 'mc-donald' -> 'Mc-Donald'
```

#### UPPER() / LOWER()
- Purpose: Convert strings to uppercase or lowercase
- Syntax: `UPPER(column)` / `LOWER(column)`

```sql
-- Convert to uppercase
SELECT UPPER(first_name), UPPER(last_name) FROM employees;
-- Result: JOHN, SMITH

-- Convert to lowercase
SELECT LOWER(email) FROM employees;
-- Result: john.smith@company.com

-- Use in WHERE clause for case-insensitive comparison
SELECT * FROM employees WHERE UPPER(last_name) = 'SMITH';
```

#### LENGTH() / LEN()
- Purpose: Return the number of characters in a string
- Syntax: `LENGTH(column)` (Oracle/PostgreSQL) or `LEN(column)` (SQL Server/MySQL)

```sql
-- Get string length
SELECT first_name, LENGTH(first_name) AS name_length FROM employees;
-- Result: John, 4

-- Filter by length
SELECT * FROM employees WHERE LENGTH(last_name) > 5;

-- Use in ORDER BY
SELECT first_name FROM employees ORDER BY LENGTH(first_name) DESC;
```

#### SUBSTR() / SUBSTRING()
- Purpose: Extract a substring from a string
- Syntax: `SUBSTR(string, start_position [, length])` (Oracle) / `SUBSTRING(string, start, length)` (MySQL/SQL Server)

```sql
-- Extract first 3 characters (Oracle/MySQL)
SELECT SUBSTR(first_name, 1, 3) AS first3 FROM employees;
-- MySQL equivalent: SELECT SUBSTRING(first_name, 1, 3) AS first3 FROM employees;

-- Extract from position 5 to end (Oracle)
SELECT SUBSTR(last_name, 5) AS tail FROM employees;

-- Extract last 3 characters (Oracle using negative start)
SELECT SUBSTR(first_name, -3) AS last3 FROM employees;

-- Extract username (part before '@') using INSTR/LOCATE
-- Oracle
SELECT SUBSTR(email, 1, INSTR(email, '@') - 1) AS username FROM employees;
-- MySQL
SELECT SUBSTRING(email, 1, LOCATE('@', email) - 1) AS username FROM employees;

-- Extract domain (part after '@')
SELECT SUBSTR(email, INSTR(email, '@') + 1) AS domain FROM employees;

-- Truncate description for report and append ellipsis
SELECT CASE WHEN LENGTH(description) > 100 THEN SUBSTR(description, 1, 100) || '...' ELSE description END AS short_desc
FROM products;

-- Use SUBSTR in WHERE to filter by pattern position
SELECT * FROM employees WHERE SUBSTR(phone, 1, 3) = '555';

-- Combine with other functions (e.g., trim and initcap)
SELECT INITCAP(TRIM(SUBSTR(full_name, 1, INSTR(full_name, ',') - 1))) AS last_name_formatted
FROM people;
```

#### CONCAT() / ||
- Purpose: Concatenate multiple strings
- Syntax: `CONCAT(arg1, arg2, ...)` or `str1 || str2` (Oracle)

```sql
-- Oracle: using concatenation operator
SELECT first_name || ' ' || last_name AS full_name FROM employees;
-- Result: John Smith

-- Oracle: CONCAT only accepts two arguments, nest for more
SELECT CONCAT(first_name, CONCAT(' ', last_name)) AS full_name FROM employees;

-- MySQL / ANSI: CONCAT accepts multiple arguments
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM employees;

-- MySQL: CONCAT_WS (concat with separator, skips NULLs)
SELECT CONCAT_WS(' ', first_name, middle_name, last_name) AS full_name FROM employees;
-- Example: 'John', NULL, 'Smith' -> 'John Smith'

-- Handle NULLs in Oracle using NVL/COALESCE
SELECT first_name || ' ' || NVL(last_name, '(No Last Name)') AS full_name FROM employees;

-- Combine with other expressions
SELECT first_name || ' (' || TO_CHAR(hire_date, 'YYYY') || ')' AS name_hire_year FROM employees;

-- Use in ORDER BY (by alias)
SELECT first_name || ' ' || last_name AS full_name, salary
FROM employees
ORDER BY full_name;
```

#### TRIM() / LTRIM() / RTRIM()
- Purpose: Remove leading/trailing (or both) characters (commonly whitespace)
- Syntax (Oracle):
  - `TRIM(string)` - remove spaces both sides
  - `TRIM([LEADING|TRAILING|BOTH] trim_char FROM string)`
  - `LTRIM(string [, trim_char])`
  - `RTRIM(string [, trim_char])`

```sql
-- Basic: remove leading and trailing spaces
SELECT TRIM('  hello world  ') AS trimmed FROM dual;
-- Result: 'hello world'

-- Remove specific character (both sides)
SELECT TRIM(BOTH '#' FROM '##value##') AS trimmed FROM dual;
-- Result: 'value'

-- Remove leading characters only
SELECT LTRIM('---note', '-') AS left_trimmed FROM dual;
-- Result: 'note'

-- Remove trailing characters only
SELECT RTRIM('note***', '*') AS right_trimmed FROM dual;
-- Result: 'note'

-- Common use: clean data before comparison
SELECT first_name, last_name
FROM employees
WHERE TRIM(last_name) = 'Smith';

-- Clean and concatenate for reports
SELECT TRIM(first_name) || ' ' || TRIM(last_name) AS full_name FROM employees;

-- Use TRIM in WHERE with wildcards
SELECT * FROM emails WHERE TRIM(subject) LIKE 'Urgent%';

-- Remove multiple different chars via nested REPLACE or REGEXP_REPLACE (Oracle)
SELECT REGEXP_REPLACE(email, '^[\s,]+|[\s,]+$','') AS email_clean FROM contacts;
```

#### LPAD() / RPAD()
- Purpose: Pad a string on the left (LPAD) or right (RPAD) to a fixed length with a specified fill string.
- Syntax: `LPAD(string, length, pad_string)` / `RPAD(string, length, pad_string)`

```sql
-- Pad numeric id with leading zeros (make 5 digits)
SELECT emp_id, LPAD(emp_id, 5, '0') AS emp_id_padded FROM employees;
-- Example result: 123 -> 00123

-- Right-pad name to fixed width for report output
SELECT first_name, RPAD(first_name || ' ' || last_name, 30, ' ') AS name_col
FROM employees
ORDER BY last_name;

-- Pad with dots for a dotted-line report field
SELECT first_name, RPAD(first_name, 20, '.') AS dotted_name FROM employees;

-- Use LPAD with VARCHAR conversion for numeric formatting
SELECT salary, LPAD(TO_CHAR(ROUND(salary)), 10, ' ') AS salary_col FROM employees;

-- Truncation behavior (source longer than length)
SELECT LPAD('ABCDEFGHI', 5, '*') AS left_trunc, RPAD('ABCDEFGHI', 5, '*') AS right_trunc FROM dual;
-- Results: left_trunc = 'ABCDE', right_trunc = 'ABCDE'

-- Multi-character pad string is repeated and then truncated as needed
SELECT LPAD('42', 7, '01') AS padded FROM dual;
-- Result: '01010142' truncated to length 7 -> '0101014'
```

#### REPLACE() / REGEXP_REPLACE()
- Purpose: Replace substring (REGEXP_REPLACE for regex in Oracle)
- Syntax (Oracle/MySQL): `REPLACE(string, search_string, replace_string)`

```sql
-- Simple replacement (Oracle/MySQL)
SELECT REPLACE(email, '@oldcompany.com', '@newcompany.com') AS email_new
FROM employees;

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

#### INSTR() / LOCATE()
- Purpose: Find position of substring within a string
- Syntax (Oracle): `INSTR(string, substring [, start_position [, occurrence]])`
- Returns: NUMBER (position) — returns 0 if not found

```sql
-- Simple: position of '@' in email
SELECT email, INSTR(email, '@') AS at_pos FROM employees;

-- Find the 3rd occurrence of 'a' in a string (Oracle)
SELECT INSTR('abracadabra', 'a', 1, 3) AS third_a FROM dual; -- returns 6

-- Start searching at a different position (Oracle)
SELECT INSTR(description, 'the', 10) AS pos_from_10 FROM products;

-- Use INSTR with SUBSTR to extract username (part before '@')
SELECT SUBSTR(email, 1, INSTR(email, '@') - 1) AS username FROM employees;

-- Test for presence (useful in WHERE)
SELECT * FROM employees WHERE INSTR(phone_number, '-') > 0;

-- MySQL: INSTR() returns position too; LOCATE() is an alternative
SELECT INSTR(email, '@') AS at_pos, LOCATE('@', email) AS at_pos_locate FROM employees;
```

### Numeric Functions

#### ROUND()
- Purpose: Round to specified number of decimal places
- Syntax: `ROUND(number [, decimal_places])`

```sql
-- Round to 2 decimal places
SELECT salary, ROUND(salary, 2) FROM employees;

-- Round to nearest integer
SELECT ROUND(45.7) FROM dual;
-- Result: 46

-- Round to hundreds place
SELECT ROUND(45678, -2) FROM dual;
-- Result: 45700
```

#### TRUNC()
- Purpose: Truncate to specified number of decimal places (no rounding)
- Syntax: `TRUNC(number [, decimal_places])`

```sql
-- Truncate to 2 decimal places
SELECT TRUNC(50000.567, 2) FROM dual;
-- Result: 50000.56

-- Truncate to integer
SELECT TRUNC(45.9) FROM dual;
-- Result: 45

-- Truncate to hundreds place
SELECT TRUNC(45678, -2) FROM dual;
-- Result: 45600
```

#### ABS(), CEIL(), FLOOR(), MOD(), POWER(), SQRT()
- Purpose: Absolute, ceiling, floor, modulus, power, square root

```sql
-- Absolute value
SELECT ABS(-100) FROM dual;
-- Result: 100

-- Ceiling (round up)
SELECT CEIL(45.2) FROM dual;
-- Result: 46

-- Floor (round down)
SELECT FLOOR(45.9) FROM dual;
-- Result: 45

-- Modulus (remainder)
SELECT MOD(emp_id, 2) AS rem FROM employees;

-- Find odd numbers
SELECT emp_id FROM employees WHERE MOD(emp_id, 2) = 1;

-- Find even numbers
SELECT emp_id FROM employees WHERE MOD(emp_id, 2) = 0;

-- Power
SELECT POWER(1.05, years) AS multiplier FROM investments;

-- Square root
SELECT SQRT(16) FROM dual;
-- Result: 4
```

### Date Functions

#### SYSDATE / NOW()
- Purpose: Current date/time
- Syntax: `SYSDATE` (Oracle) / `NOW()` (MySQL)

```sql
-- Get current date/time
SELECT SYSDATE FROM dual;        -- Oracle
SELECT NOW() FROM dual;          -- MySQL

-- Use in INSERT
INSERT INTO audit_log (action, timestamp) VALUES ('Login', SYSDATE);

-- Calculate days since hire
SELECT first_name, TRUNC(SYSDATE - hire_date) AS days_employed FROM employees;
```

#### ADD_MONTHS(), MONTHS_BETWEEN(), NEXT_DAY(), LAST_DAY()

**ADD_MONTHS**:
```sql
-- Probation end date (3 months after hire)
SELECT emp_id, hire_date, ADD_MONTHS(hire_date, 3) AS probation_end FROM employees;

-- Calculate 5-year maturity date
SELECT loan_id, start_date, ADD_MONTHS(start_date, 12 * 5) AS maturity_date FROM loans;
```

**MONTHS_BETWEEN**:
```sql
-- Fractional months employed and whole months
SELECT emp_id,
       MONTHS_BETWEEN(SYSDATE, hire_date) AS months_employed_exact,
       TRUNC(MONTHS_BETWEEN(SYSDATE, hire_date)) AS months_employed_whole,
       ROUND(MONTHS_BETWEEN(SYSDATE, hire_date) / 12, 2) AS years_employed_rounded
FROM employees;

-- Compare two dates (order matters: date1 - date2)
SELECT TO_CHAR(hire_date,'DD-MON-YYYY') d,
       TO_CHAR(SYSDATE,'DD-MON-YYYY') today,
       MONTHS_BETWEEN(SYSDATE, hire_date) AS months_diff
FROM employees
WHERE emp_id = 101;
```

**NEXT_DAY**:
```sql
-- Schedule next occurrence of a weekday (e.g., next Friday after SYSDATE)
SELECT SYSDATE AS today, NEXT_DAY(SYSDATE, 'FRIDAY') AS next_friday FROM dual;

-- Find next Monday after hire_date (useful for onboarding)
SELECT emp_id, hire_date, NEXT_DAY(hire_date, 'MONDAY') AS onboarding_monday
FROM employees;
```

**LAST_DAY**:
```sql
-- Last day of hire month and days remaining in hire month
SELECT emp_id, hire_date,
       LAST_DAY(hire_date) AS hire_month_end,
       TRUNC(LAST_DAY(hire_date) - hire_date) AS days_until_month_end
FROM employees;

-- Last day of next month
SELECT SYSDATE, LAST_DAY(ADD_MONTHS(SYSDATE, 1)) AS last_day_next_month FROM dual;

-- Give bonus to employees employed at least 12 months
SELECT emp_id, first_name, hire_date,
       FLOOR(MONTHS_BETWEEN(SYSDATE, hire_date) / 12) AS years_employed
FROM employees
WHERE FLOOR(MONTHS_BETWEEN(SYSDATE, hire_date) / 12) >= 1;
```

**TRUNC() for Dates**:
```sql
-- Remove time portion (midnight)
SELECT TRUNC(SYSDATE) FROM dual;
-- Result: 14-NOV-25

-- Truncate to first day of year
SELECT TRUNC(SYSDATE, 'YEAR') FROM dual;
-- Result: 01-JAN-25

-- Truncate to first day of month
SELECT TRUNC(SYSDATE, 'MONTH') FROM dual;
-- Result: 01-NOV-25

-- Truncate to hour
SELECT TRUNC(SYSDATE, 'HH') FROM dual;
```

**EXTRACT()**:
```sql
-- Extract year
SELECT EXTRACT(YEAR FROM hire_date) FROM employees;

-- Extract month
SELECT EXTRACT(MONTH FROM SYSDATE) FROM dual;
-- Result: 11 (November)

-- Extract day
SELECT EXTRACT(DAY FROM hire_date) FROM employees;

-- Find employees hired in November
SELECT * FROM employees WHERE EXTRACT(MONTH FROM hire_date) = 11;
```

### Date Arithmetic (Oracle vs MySQL)

- Notes: In Oracle adding a numeric value to a DATE adds days. Use INTERVAL or NUMTODSINTERVAL for finer units. MySQL uses DATE_ADD / DATE_SUB or TIMESTAMPADD / TIMESTAMPDIFF.

```sql
-- Oracle: add / subtract days
SELECT hire_date, hire_date + 30 AS hire_plus_30_days FROM employees;

-- Oracle: add hours/minutes using fractional days
SELECT SYSDATE, SYSDATE + (5/24) AS plus_5_hours FROM dual;

-- Oracle: add using INTERVAL (day / hour / minute)
SELECT hire_date,
       hire_date + INTERVAL '2' DAY AS plus_2_days,
       hire_date + INTERVAL '3' HOUR AS plus_3_hours,
       hire_date + INTERVAL '90' MINUTE AS plus_90_minutes
FROM employees;

-- Oracle: numeric difference (days) and convert to hours/minutes
SELECT emp_id,
       TRUNC(SYSDATE - hire_date) AS days_employed,
       ROUND((SYSDATE - hire_date) * 24, 2) AS hours_employed
FROM employees;

-- Oracle: NUMTODSINTERVAL example
SELECT hire_date, hire_date + NUMTODSINTERVAL(90, 'MINUTE') AS plus_90_min FROM employees;

-- Oracle: format result
SELECT TO_CHAR(hire_date + 30, 'DD-MON-YYYY') AS hire_plus_30_fmt FROM employees;

-- MySQL: add / subtract days
SELECT hire_date, DATE_ADD(hire_date, INTERVAL 30 DAY) AS hire_plus_30_days FROM employees;

-- MySQL: add hours/minutes
SELECT NOW(), DATE_ADD(NOW(), INTERVAL 5 HOUR) AS plus_5_hours, DATE_ADD(NOW(), INTERVAL 90 MINUTE) AS plus_90_min;

-- MySQL: date difference in days / hours
SELECT DATEDIFF(NOW(), hire_date) AS days_employed,
       TIMESTAMPDIFF(HOUR, hire_date, NOW()) AS hours_employed
FROM employees;

-- MySQL: subtract days
SELECT hire_date, DATE_SUB(hire_date, INTERVAL 7 DAY) AS hire_minus_7_days FROM employees;
```

---

**Last Updated**: November 17, 2025
**Course**: Oracle 19c SQL Workshop