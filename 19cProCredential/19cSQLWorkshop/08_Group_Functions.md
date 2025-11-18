# Group Functions

## Table of Contents
1. [Introduction to Group Functions](#introduction-to-group-functions)
2. [Common Group Functions](#common-group-functions)
3. [Using GROUP BY Clause](#using-group-by-clause)
4. [HAVING Clause](#having-clause)
5. [Detailed Examples](#detailed-examples)
6. [Advanced Group Function Techniques](#advanced-group-function-techniques)
7. [Best Practices](#best-practices)

---

### Introduction to Group Functions
Group functions, also known as aggregate functions, perform calculations on a set of values and return a single value. They are commonly used in conjunction with the `GROUP BY` clause to summarize data.

**Key Characteristics**:
- Operate on multiple rows
- Return a single result per group
- Can be used in SELECT, WHERE (with subqueries), and HAVING clauses
- Ignore NULL values (except COUNT(*))

---

### Common Group Functions

#### COUNT()
- **Purpose**: Count the number of rows or non-NULL values
- **Syntax**: `COUNT(*)` or `COUNT(column_name)` or `COUNT(DISTINCT column_name)`
- **Returns**: NUMBER

```sql
-- Count total rows
SELECT COUNT(*) AS total_employees FROM employees;

-- Count non-NULL values in a column
SELECT COUNT(commission_pct) AS employees_with_commission FROM employees;

-- Count distinct values
SELECT COUNT(DISTINCT department) AS unique_departments FROM employees;
```

#### SUM()
- **Purpose**: Calculate the sum of numeric values
- **Syntax**: `SUM(numeric_column)`
- **Returns**: NUMBER
- **Note**: Ignores NULL values

```sql
-- Sum all salaries
SELECT SUM(salary) AS total_salary FROM employees;

-- Sum with WHERE clause
SELECT SUM(salary) AS it_department_salary FROM employees WHERE department = 'IT';

-- Sum commission (NULL values ignored)
SELECT SUM(commission_pct) AS total_commission FROM employees;
```

#### AVG()
- **Purpose**: Calculate the average of numeric values
- **Syntax**: `AVG(numeric_column)`
- **Returns**: NUMBER
- **Note**: Ignores NULL values; divides by count of non-NULL values

```sql
-- Average salary across all employees
SELECT AVG(salary) AS average_salary FROM employees;

-- Average with rounding
SELECT ROUND(AVG(salary), 2) AS average_salary FROM employees;

-- Average commission (excludes NULLs)
SELECT AVG(commission_pct) AS average_commission FROM employees;
```

#### MIN()
- **Purpose**: Find the minimum value in a set
- **Syntax**: `MIN(column_name)`
- **Returns**: Same data type as the column
- **Works with**: Numbers, dates, strings

```sql
-- Minimum salary
SELECT MIN(salary) AS lowest_salary FROM employees;

-- Earliest hire date
SELECT MIN(hire_date) AS earliest_hire_date FROM employees;

-- Alphabetically first name
SELECT MIN(first_name) AS first_alphabetically FROM employees;
```

#### MAX()
- **Purpose**: Find the maximum value in a set
- **Syntax**: `MAX(column_name)`
- **Returns**: Same data type as the column
- **Works with**: Numbers, dates, strings

```sql
-- Maximum salary
SELECT MAX(salary) AS highest_salary FROM employees;

-- Latest hire date
SELECT MAX(hire_date) AS most_recent_hire FROM employees;

-- Alphabetically last name
SELECT MAX(last_name) AS last_alphabetically FROM employees;
```

---

### Using GROUP BY Clause

The `GROUP BY` clause groups rows with identical values in specified column(s).

**Syntax**:
```sql
SELECT column1, aggregate_function(column2)
FROM table_name
WHERE condition
GROUP BY column1
ORDER BY aggregate_function(column2);
```

**Rules for GROUP BY**:
1. Any column in SELECT not in an aggregate function must be in GROUP BY
2. GROUP BY is executed before HAVING
3. Multiple columns can be grouped together

```sql
-- Single column grouping
SELECT department, COUNT(*) AS emp_count
FROM employees
GROUP BY department;

-- Multiple column grouping
SELECT department, job_title, COUNT(*) AS emp_count
FROM employees
GROUP BY department, job_title;

-- Group with WHERE (filters rows before grouping)
SELECT department, COUNT(*) AS emp_count
FROM employees
WHERE salary > 50000
GROUP BY department;
```

---

### HAVING Clause

The `HAVING` clause filters group results (applied after GROUP BY), similar to WHERE but for aggregates.

**Syntax**:
```sql
SELECT column1, aggregate_function(column2)
FROM table_name
GROUP BY column1
HAVING aggregate_condition
ORDER BY aggregate_function(column2);
```

**WHERE vs HAVING**:
- **WHERE**: Filters individual rows before grouping
- **HAVING**: Filters groups after aggregation

```sql
-- HAVING example: departments with more than 5 employees
SELECT department, COUNT(*) AS emp_count
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;

-- Combining WHERE and HAVING
SELECT department, AVG(salary) AS avg_salary
FROM employees
WHERE hire_date >= TO_DATE('2020-01-01', 'YYYY-MM-DD')
GROUP BY department
HAVING AVG(salary) > 60000;
```

---

### Detailed Examples

#### 1. Employee Count by Department
```sql
SELECT 
  department,
  COUNT(*) AS total_employees,
  COUNT(DISTINCT job_title) AS unique_positions
FROM employees
GROUP BY department
ORDER BY total_employees DESC;
```

#### 2. Salary Statistics by Department
```sql
SELECT 
  department,
  COUNT(*) AS emp_count,
  SUM(salary) AS total_salary,
  ROUND(AVG(salary), 2) AS avg_salary,
  MIN(salary) AS min_salary,
  MAX(salary) AS max_salary
FROM employees
GROUP BY department
ORDER BY avg_salary DESC;
```

#### 3. Commission Analysis
```sql
SELECT 
  CASE 
    WHEN commission_pct IS NULL THEN 'No Commission'
    WHEN commission_pct < 0.1 THEN '< 10%'
    WHEN commission_pct < 0.2 THEN '10-20%'
    ELSE '> 20%'
  END AS commission_level,
  COUNT(*) AS employee_count,
  ROUND(AVG(salary), 2) AS avg_salary
FROM employees
GROUP BY CASE 
    WHEN commission_pct IS NULL THEN 'No Commission'
    WHEN commission_pct < 0.1 THEN '< 10%'
    WHEN commission_pct < 0.2 THEN '10-20%'
    ELSE '> 20%'
  END
ORDER BY employee_count DESC;
```

#### 4. Departments with High Average Salary
```sql
SELECT 
  department,
  COUNT(*) AS emp_count,
  ROUND(AVG(salary), 2) AS avg_salary,
  MAX(salary) - MIN(salary) AS salary_range
FROM employees
GROUP BY department
HAVING AVG(salary) > 70000
ORDER BY avg_salary DESC;
```

#### 5. Tenure Analysis by Department
```sql
-- Oracle syntax
SELECT 
  department,
  COUNT(*) AS emp_count,
  ROUND(AVG(MONTHS_BETWEEN(SYSDATE, hire_date) / 12), 1) AS avg_years_employed,
  ROUND(MIN(MONTHS_BETWEEN(SYSDATE, hire_date) / 12), 1) AS min_years,
  ROUND(MAX(MONTHS_BETWEEN(SYSDATE, hire_date) / 12), 1) AS max_years
FROM employees
GROUP BY department
ORDER BY avg_years_employed DESC;

-- MySQL syntax
SELECT 
  department,
  COUNT(*) AS emp_count,
  ROUND(AVG(TIMESTAMPDIFF(YEAR, hire_date, NOW())), 1) AS avg_years_employed,
  ROUND(MIN(TIMESTAMPDIFF(YEAR, hire_date, NOW())), 1) AS min_years,
  ROUND(MAX(TIMESTAMPDIFF(YEAR, hire_date, NOW())), 1) AS max_years
FROM employees
GROUP BY department
ORDER BY avg_years_employed DESC;
```

#### 6. Multi-level Grouping: Department and Job Title
```sql
SELECT 
  department,
  job_title,
  COUNT(*) AS emp_count,
  ROUND(AVG(salary), 2) AS avg_salary,
  SUM(salary) AS total_salary
FROM employees
GROUP BY department, job_title
HAVING COUNT(*) >= 2
ORDER BY department, avg_salary DESC;
```

#### 7. Salary Bracket Distribution
```sql
SELECT 
  CASE 
    WHEN salary < 30000 THEN 'Entry Level (< $30K)'
    WHEN salary < 60000 THEN 'Mid Level ($30K-$60K)'
    WHEN salary < 90000 THEN 'Senior ($60K-$90K)'
    ELSE 'Executive ($90K+)'
  END AS salary_bracket,
  COUNT(*) AS employee_count,
  ROUND(AVG(salary), 2) AS avg_salary
FROM employees
GROUP BY CASE 
    WHEN salary < 30000 THEN 'Entry Level (< $30K)'
    WHEN salary < 60000 THEN 'Mid Level ($30K-$60K)'
    WHEN salary < 90000 THEN 'Senior ($60K-$90K)'
    ELSE 'Executive ($90K+)'
  END
ORDER BY employee_count DESC;
```

#### 8. Departments with Salary Variance Analysis
```sql
SELECT 
  department,
  COUNT(*) AS emp_count,
  ROUND(AVG(salary), 2) AS avg_salary,
  MAX(salary) - MIN(salary) AS salary_range,
  ROUND((MAX(salary) - MIN(salary)) / AVG(salary) * 100, 2) AS variance_percent
FROM employees
GROUP BY department
HAVING MAX(salary) - MIN(salary) > 50000
ORDER BY variance_percent DESC;
```

#### 9. Hire Date Year Distribution
```sql
-- Oracle syntax
SELECT 
  TO_CHAR(hire_date, 'YYYY') AS hire_year,
  COUNT(*) AS employees_hired,
  ROUND(AVG(salary), 2) AS avg_salary
FROM employees
GROUP BY TO_CHAR(hire_date, 'YYYY')
ORDER BY hire_year DESC;

-- MySQL syntax
SELECT 
  YEAR(hire_date) AS hire_year,
  COUNT(*) AS employees_hired,
  ROUND(AVG(salary), 2) AS avg_salary
FROM employees
GROUP BY YEAR(hire_date)
ORDER BY hire_year DESC;
```

#### 10. Top Earners by Department
```sql
SELECT 
  department,
  COUNT(*) AS emp_count,
  SUM(salary) AS total_salary,
  ROUND(AVG(salary), 2) AS avg_salary,
  ROUND(MAX(salary), 2) AS top_salary,
  ROUND(MIN(salary), 2) AS bottom_salary
FROM employees
GROUP BY department
HAVING SUM(salary) > 500000
ORDER BY total_salary DESC;
```

---

### Advanced Group Function Techniques

#### Nested Group Functions (Aggregate of Aggregates)
```sql
-- Find the average of department averages
SELECT ROUND(AVG(dept_avg), 2) AS overall_avg_salary
FROM (
  SELECT AVG(salary) AS dept_avg
  FROM employees
  GROUP BY department
);
```

#### Conditional Aggregation with CASE
```sql
-- Count employees by salary level
SELECT 
  department,
  COUNT(CASE WHEN salary < 50000 THEN 1 END) AS junior_staff,
  COUNT(CASE WHEN salary >= 50000 AND salary < 100000 THEN 1 END) AS mid_level,
  COUNT(CASE WHEN salary >= 100000 THEN 1 END) AS senior_staff,
  COUNT(*) AS total
FROM employees
GROUP BY department;
```

#### DISTINCT within Aggregates
```sql
-- Count distinct job titles per department
SELECT 
  department,
  COUNT(DISTINCT job_title) AS unique_positions,
  COUNT(*) AS total_employees
FROM employees
GROUP BY department
ORDER BY unique_positions DESC;
```

#### Group with NULL Values
```sql
-- Show NULL grouping behavior
SELECT 
  department,
  commission_pct,
  COUNT(*) AS emp_count
FROM employees
GROUP BY department, commission_pct
ORDER BY department, commission_pct;
```

---

### Best Practices

1. **Always include non-aggregated columns in GROUP BY**
   ```sql
   -- ❌ WRONG: job_title not in GROUP BY (may error or return unexpected results)
   SELECT department, job_title, AVG(salary) FROM employees GROUP BY department;
   
   -- ✅ CORRECT: Both columns in GROUP BY
   SELECT department, job_title, AVG(salary) FROM employees GROUP BY department, job_title;
   ```

2. **Use WHERE to filter rows before grouping, HAVING to filter groups after**
   ```sql
   -- ✅ CORRECT: WHERE filters before grouping (more efficient)
   SELECT department, AVG(salary) AS avg_sal
   FROM employees
   WHERE hire_date >= TRUNC(SYSDATE) - 365  -- Oracle
   GROUP BY department
   HAVING AVG(salary) > 60000;
   ```

3. **Use aliases for readability**
   ```sql
   -- ✅ CORRECT: Clear aliases
   SELECT 
     department,
     COUNT(*) AS emp_count,
     ROUND(AVG(salary), 2) AS avg_salary
   FROM employees
   GROUP BY department;
   ```

4. **Be aware of NULL handling**
   ```sql
   -- NULL values are excluded from aggregate calculations
   SELECT 
     COUNT(*) AS total_rows,
     COUNT(commission_pct) AS non_null_commissions,
     SUM(commission_pct) AS total_commission
   FROM employees;
   -- total_rows might be 100, but COUNT(commission_pct) might be 35
   ```

5. **Use ROUND() for numeric aggregates**
   ```sql
   -- ✅ CORRECT: Rounding for currency/decimals
   SELECT 
     department,
     ROUND(SUM(salary), 2) AS total_salary,
     ROUND(AVG(salary), 2) AS avg_salary
   FROM employees
   GROUP BY department;
   ```

6. **Order GROUP BY results for reports**
   ```sql
   -- ✅ CORRECT: Ordered results for better readability
   SELECT department, COUNT(*) AS emp_count
   FROM employees
   GROUP BY department
   ORDER BY emp_count DESC;
   ```

---

**Course**: Oracle 19c SQL Workshop
**Last Updated**: November 18, 2025