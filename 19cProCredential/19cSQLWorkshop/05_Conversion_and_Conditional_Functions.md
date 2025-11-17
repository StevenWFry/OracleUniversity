# Conversion Functions and Conditional Expressions

## Table of Contents
1. [Implicit vs Explicit Conversion](#implicit-vs-explicit-conversion)
2. [Conversion Functions](#conversion-functions)
3. [Conditional Expressions](#conditional-expressions)
4. [Best Practices](#best-practices)

### Implicit vs Explicit Conversion

- **Implicit conversion**: Database automatically converts between types (risky — depends on NLS settings)
- **Explicit conversion**: You call conversion functions (TO_CHAR, TO_DATE, TO_NUMBER, CAST, etc.)

**Implicit Examples (Oracle)**:
```sql
-- Numeric column compared to string literal: Oracle will attempt to convert the literal to NUMBER
SELECT * FROM employees WHERE emp_id = '101';

-- Date compared to string literal: Oracle will use NLS_DATE_FORMAT to convert the string to DATE
-- (behavior depends on session NLS settings; avoid relying on this)
SELECT * FROM employees WHERE hire_date = '01-JAN-2020';
```

**Explicit Examples (Preferred)**:
```sql
-- Convert literal to number explicitly
SELECT * FROM employees WHERE emp_id = TO_NUMBER('101');

-- Convert string to date explicitly (use correct format mask)
SELECT * FROM employees WHERE hire_date = TO_DATE('01-JAN-2020','DD-MON-YYYY');

-- Format date as string for display
SELECT first_name, TO_CHAR(hire_date,'DD-MON-YYYY') AS hire_fmt FROM employees;

-- Convert number to formatted string
SELECT TO_CHAR(salary,'FM999,999.00') AS salary_fmt FROM employees;

-- Use CAST for ANSI-style conversion
SELECT CAST(emp_id AS VARCHAR2(10)) AS emp_id_str FROM employees;
```

**Notes**:
- Prefer explicit conversions to avoid ORA-01722 invalid number and NLS-dependent date mismatches.
- Explicit conversions make intent clear and improve portability and maintainability.

---

### Conversion Functions

#### TO_CHAR() (Oracle)
- **Purpose**: Convert date or number to character string
- **Syntax**: `TO_CHAR(date [, format])` or `TO_CHAR(number [, format])`
- **Returns**: VARCHAR2

```sql
-- Format date
SELECT TO_CHAR(hire_date, 'DD-MON-YYYY') FROM employees;
-- Result: 15-JAN-2020

-- Format with time
SELECT TO_CHAR(SYSDATE, 'DD/MM/YYYY HH:MI:SS') FROM dual;
-- Result: 14/11/2025 14:30:45

-- Format number
SELECT TO_CHAR(salary, '999,999.99') FROM employees;
-- Result: 50,000.00

-- Format with currency
SELECT TO_CHAR(salary, '$999,999.99') FROM employees;
-- Result: $50,000.00
```

**Number Format Model Elements (TO_CHAR)**:

| Format Element | Meaning |
|----------------|---------|
| 9              | Represents a number |
| 0              | Forces a zero to be displayed |
| $              | Places a floating dollar sign |
| L              | Uses the floating local currency symbol |
| .              | Prints a decimal point |
| ,              | Prints a comma as a thousands indicator |

```sql
-- Example: format salary with currency and thousands separator
SELECT TO_CHAR(salary, '$99,999.99') AS salary_fmt FROM employees;
-- Result: $50,000.00
```

#### TO_DATE() (Oracle)
- **Purpose**: Convert character string to date
- **Syntax**: `TO_DATE(string, format)`
- **Returns**: DATE

```sql
-- Convert string to date
SELECT TO_DATE('15-JAN-2020', 'DD-MON-YYYY') FROM dual;

-- Use in WHERE clause
SELECT * FROM employees WHERE hire_date > TO_DATE('2020-01-01', 'YYYY-MM-DD');

-- Parse user input
SELECT * FROM employees WHERE hire_date = TO_DATE('&hire_date', 'DD-MON-YYYY');
```

#### RR (two-digit year) format example
- **Purpose**: Allow two-digit year input to be resolved to the appropriate century.
- **Behavior**: When using the RR format for two-digit years, Oracle maps:
  - 00–49 -> 2000–2049
  - 50–99 -> 1950–1999
- **Use case**: Helpful when accepting legacy two-digit year input but needing sensible century resolution.

```sql
-- Two-digit year interpreted with RR

-- '30' -> 2030
SELECT TO_CHAR(TO_DATE('01-JAN-30', 'DD-MON-RR'), 'YYYY-MM-DD') AS resolved_date FROM dual;
-- Result: 2030-01-01

-- '70' -> 1970
SELECT TO_CHAR(TO_DATE('01-JAN-70', 'DD-MON-RR'), 'YYYY-MM-DD') AS resolved_date FROM dual;
-- Result: 1970-01-01

-- Round-trip: show RR output formatting
SELECT TO_CHAR(TO_DATE('01-JAN-2030', 'DD-MON-YYYY'), 'DD-MON-RR') AS two_digit_year FROM dual;
-- Result: 01-JAN-30

-- Real-world example: Find employees hired before 2010 using RR format
SELECT last_name, TO_CHAR(hire_date, 'DD-MON-YYYY') AS hire_date_fmt
FROM employees
WHERE hire_date < TO_DATE('01-JAN-10', 'DD-MON-RR');
-- Result: Shows employees hired before 2010 (e.g., Kochhar 21-Sep-2009, De Haan 13-Jan-2009)
```

#### TO_NUMBER() (Oracle)
- **Purpose**: Convert character string to number
- **Syntax**: `TO_NUMBER(string [, format])`
- **Returns**: NUMBER

```sql
-- Convert string to number
SELECT TO_NUMBER('12345') FROM dual;

-- Parse formatted number
SELECT TO_NUMBER('$1,234.56', '$9,999.99') FROM dual;

-- Use in calculation
SELECT salary + TO_NUMBER(bonus_text) AS total_comp FROM employees;

-- Convert with scientific notation
SELECT TO_NUMBER('1.5E3') FROM dual;
-- Result: 1500
```

#### CAST() (ANSI Standard)
- **Purpose**: Convert between data types using standard ANSI syntax
- **Syntax**: `CAST(expression AS target_data_type)`
- **Returns**: Specified target data type
- **Advantage**: Portable across Oracle, MySQL, SQL Server, PostgreSQL

```sql
-- Convert number to character/string
SELECT CAST(emp_id AS VARCHAR2(10)) AS emp_id_str FROM employees;

-- Convert string to number
SELECT CAST('12345' AS NUMBER) FROM dual;

-- Convert date to character
SELECT CAST(hire_date AS VARCHAR2(20)) FROM employees;

-- Convert string to date (Oracle may still need TO_DATE for format control)
SELECT CAST('2020-01-15' AS DATE) FROM dual;

-- Use CAST in WHERE clause for comparison
SELECT * FROM employees WHERE CAST(emp_id AS VARCHAR2(5)) = '00101';

-- CAST with arithmetic (convert and calculate)
SELECT emp_id, CAST(salary AS DECIMAL(10,2)) * 1.1 AS salary_increase FROM employees;

-- Chain CAST operations
SELECT CAST(CAST(employee_code AS NUMBER) AS VARCHAR2(10)) AS formatted_code FROM employees;
```

#### DATE_FORMAT() (MySQL)
- **Purpose**: Format date as string (MySQL equivalent to TO_CHAR)
- **Syntax**: `DATE_FORMAT(date, format_string)`
- **Returns**: VARCHAR

```sql
-- Format date (MySQL)
SELECT DATE_FORMAT(hire_date, '%d-%b-%Y') FROM employees;
-- Result: 15-Jan-2020

-- Format with time
SELECT DATE_FORMAT(NOW(), '%d/%m/%Y %H:%i:%S');
-- Result: 14/11/2025 14:30:45

-- Custom formatting for reports
SELECT emp_id, DATE_FORMAT(hire_date, '%W, %M %d, %Y') AS hire_date_formatted FROM employees;
-- Result: Wednesday, January 15, 2020
```

#### STR_TO_DATE() (MySQL)
- **Purpose**: Convert string to date (MySQL equivalent to TO_DATE)
- **Syntax**: `STR_TO_DATE(string, format)`
- **Returns**: DATE

```sql
-- Convert string to date (MySQL)
SELECT STR_TO_DATE('15-Jan-2020', '%d-%b-%Y') FROM dual;

-- Use in WHERE clause
SELECT * FROM employees WHERE hire_date > STR_TO_DATE('2020-01-01', '%Y-%m-%d');

-- Parse user input
SELECT * FROM employees WHERE hire_date = STR_TO_DATE('&hire_date', '%d-%b-%Y');
```

---

### Conditional Expressions

#### CASE
- **Purpose**: Conditional logic returning different values based on conditions
- **Syntax**: 
  - Simple: `CASE expression WHEN value1 THEN result1 WHEN value2 THEN result2 ELSE result_default END`
  - Searched: `CASE WHEN condition1 THEN result1 WHEN condition2 THEN result2 ELSE result_default END`
- **Returns**: Specified data type

```sql
-- Simple CASE (compare one expression)
SELECT first_name, 
       CASE department
         WHEN 'IT' THEN 'Information Technology'
         WHEN 'HR' THEN 'Human Resources'
         WHEN 'Finance' THEN 'Financial Services'
         ELSE 'Other'
       END AS department_name
FROM employees;

-- Searched CASE (complex conditions)
SELECT first_name, salary,
       CASE
         WHEN salary >= 100000 THEN 'Executive'
         WHEN salary >= 75000 THEN 'Senior'
         WHEN salary >= 50000 THEN 'Mid-level'
         ELSE 'Junior'
       END AS salary_grade
FROM employees;

-- CASE with aggregate functions
SELECT department,
       COUNT(CASE WHEN salary > 60000 THEN 1 END) AS high_earners,
       COUNT(CASE WHEN salary <= 60000 THEN 1 END) AS regular_staff
FROM employees
GROUP BY department;

-- CASE in ORDER BY
SELECT first_name, department
FROM employees
ORDER BY CASE department
           WHEN 'IT' THEN 1
           WHEN 'HR' THEN 2
           WHEN 'Finance' THEN 3
           ELSE 4
         END;

-- Nested CASE
SELECT first_name, salary, department,
       CASE
         WHEN department = 'IT' THEN
           CASE WHEN salary > 80000 THEN 'Senior IT' ELSE 'Junior IT' END
         WHEN department = 'HR' THEN
           CASE WHEN salary > 70000 THEN 'Senior HR' ELSE 'Junior HR' END
         ELSE 'Other'
       END AS position_level
FROM employees;
```

#### DECODE() (Oracle specific)
- **Purpose**: Oracle's proprietary conditional function (similar to CASE but older syntax)
- **Syntax**: `DECODE(expression, search1, result1, [search2, result2, ...], default_result)`
- **Returns**: Specified data type
- **Note**: Less readable than CASE; CASE is preferred for new code

```sql
-- Simple decode
SELECT first_name,
       DECODE(department,
              'IT', 'Information Technology',
              'HR', 'Human Resources',
              'Finance', 'Financial Services',
              'Other') AS department_name
FROM employees;

-- Decode with conditions (not recommended for complex logic)
SELECT emp_id,
       DECODE(SIGN(salary - 50000),
              1, 'Above Average',
              0, 'Average',
             -1, 'Below Average') AS salary_category
FROM employees;

-- Decode with aggregates
SELECT DECODE(gender, 'M', 'Male', 'F', 'Female') AS gender,
       COUNT(*) AS employee_count
FROM employees
GROUP BY DECODE(gender, 'M', 'Male', 'F', 'Female');

-- Decode vs CASE comparison
-- DECODE (less readable)
SELECT DECODE(status, 'A', 'Active', 'I', 'Inactive', 'Unknown') FROM employees;

-- CASE (more readable)
SELECT CASE WHEN status = 'A' THEN 'Active' WHEN status = 'I' THEN 'Inactive' ELSE 'Unknown' END FROM employees;
```

#### NULL Handling Functions

##### COALESCE()
- **Purpose**: Return first non-NULL value from list of expressions
- **Syntax**: `COALESCE(expression1, expression2, ..., expression_n)`
- **Returns**: Data type of first non-NULL expression
- **Advantage**: ANSI standard, works in Oracle, MySQL, SQL Server, PostgreSQL

```sql
-- Return first non-NULL commission or 0
SELECT first_name, COALESCE(commission_pct, 0) AS commission FROM employees;

-- Use multiple columns for fallback
SELECT emp_id, 
       COALESCE(middle_name, last_name, 'Unknown') AS name_fallback
FROM employees;

-- Calculate with NULL handling
SELECT salary, bonus, COALESCE(bonus, salary * 0.1) AS final_bonus FROM employees;

-- Multiple fallback levels
SELECT emp_id,
       COALESCE(department, work_location, 'Unassigned') AS assignment
FROM employees;
```

##### NVL() (Oracle)
- **Purpose**: Return alternative value if expression is NULL
- **Syntax**: `NVL(expression, null_value_replacement)`
- **Returns**: Data type of first expression
- **Note**: Oracle-specific; only accepts 2 arguments

```sql
-- Handle NULL commission
SELECT first_name, NVL(commission_pct, 0) AS commission FROM employees;

-- Handle NULL department
SELECT emp_id, NVL(department, 'Unassigned') AS dept FROM employees;

-- Calculate with NULL values
SELECT salary, NVL(bonus, salary * 0.1) AS total_compensation FROM employees;

-- Use in ORDER BY
SELECT first_name FROM employees ORDER BY NVL(middle_name, last_name);
```

##### NVL2() (Oracle)
- **Purpose**: Return one value if expression is NOT NULL, another if it IS NULL
- **Syntax**: `NVL2(expression, not_null_result, null_result)`
- **Returns**: Data type of not_null_result or null_result
- **Note**: Oracle-specific

```sql
-- Conditional based on NULL
SELECT emp_id,
       NVL2(commission_pct, 'Has Commission', 'No Commission') AS comm_status
FROM employees;

-- Different calculations based on NULL
SELECT salary,
       NVL2(bonus, salary + bonus, salary * 1.05) AS total_comp
FROM employees;

-- Mark active vs inactive
SELECT emp_id,
       NVL2(termination_date, 'Inactive', 'Active') AS status
FROM employees;

-- String manipulation with NVL2
SELECT first_name, 
       NVL2(middle_name, first_name || ' ' || middle_name || ' ' || last_name, first_name || ' ' || last_name) AS full_name
FROM employees;
```

##### NULLIF()
- **Purpose**: Return NULL if two expressions are equal; otherwise, return the first expression
- **Syntax**: `NULLIF(expression1, expression2)`
- **Returns**: Data type of expression1
- **Common use**: Avoid division by zero errors

```sql
-- Avoid division by zero
SELECT revenue, costs, NULLIF(costs, 0) AS costs_nonzero,
       revenue / NULLIF(costs, 0) AS profit_margin
FROM quarters;

-- Conditional logic with NULLIF
SELECT first_name, salary,
       NULLIF(salary, 0) * 12 AS annual_salary
FROM employees;

-- Mark unchanged records
SELECT old_value, new_value,
       CASE WHEN NULLIF(old_value, new_value) IS NULL THEN 'No Change' ELSE 'Changed' END AS change_status
FROM audit_log;

-- Filter out specific values by returning NULL
SELECT emp_id,
       NULLIF(department, 'Temp') AS permanent_dept
FROM employees;
```

##### IFNULL() / IF() (MySQL)
- **Purpose**: MySQL-specific functions to handle NULLs or perform conditional logic
- **Syntax**: 
  - `IFNULL(expression, alt_value)` — equivalent to NVL
  - `IF(condition, true_value, false_value)` — equivalent to CASE
- **Returns**: Specified data type

```sql
-- IFNULL example (MySQL equivalent to NVL)
SELECT first_name, IFNULL(commission_pct, 0) AS commission FROM employees;

-- IFNULL with calculations
SELECT emp_id, IFNULL(bonus, salary * 0.05) AS bonus_amount FROM employees;

-- IF example (MySQL equivalent to CASE)
SELECT first_name, salary,
       IF(salary > 50000, 'High', 'Low') AS salary_level
FROM employees;

-- Nested IF
SELECT emp_id,
       IF(salary > 100000, 'Executive',
          IF(salary > 75000, 'Senior',
             IF(salary > 50000, 'Mid-level', 'Junior'))) AS grade
FROM employees;

-- IF with multiple conditions
SELECT emp_id,
       IF(status = 'Active' AND salary > 60000, 'Eligible for Bonus', 'Not Eligible') AS bonus_status
FROM employees;
```

---

### Best Practices

1. **Always use explicit conversions** for clarity and portability
   - Avoid relying on implicit conversions
   - Use TO_CHAR, TO_DATE, CAST explicitly

2. **Use CASE over DECODE** for new code
   - More readable and ANSI standard
   - Easier to maintain and debug
   - Works across multiple database platforms

3. **Use COALESCE for NULL handling** over database-specific functions
   - COALESCE is ANSI standard
   - Works in Oracle, MySQL, SQL Server, PostgreSQL
   - More readable than multiple nested conditions

4. **Use appropriate format models**
   - Understand your data and use correct format masks
   - Use RR for legacy two-digit year data
   - Document complex formatting patterns

5. **Protect calculations with NULLIF**
   - Use NULLIF(denominator, 0) to avoid division by zero
   - Prevents unexpected errors in reports

6. **Test conversions with sample data**
   - Verify format strings work as expected
   - Check NULL handling behavior
   - Validate date century resolution with RR format

7. **Performance considerations**
   - Use explicit conversions in WHERE clauses for index usage
   - Convert in SELECT clause when filtering on index columns
   - Avoid functions on indexed columns in WHERE if possible

---

**Course**: Oracle 19c SQL Workshop