# Set Operators

## Table of Contents
1. [Introduction to Set Operators](#introduction-to-set-operators)
2. [UNION](#union)
3. [UNION ALL](#union-all)
4. [INTERSECT](#intersect)
5. [MINUS / EXCEPT](#minus--except)
6. [Set Operator Rules and Guidelines](#set-operator-rules-and-guidelines)
7. [Controlling Row Order](#controlling-row-order)
8. [Best Practices](#best-practices)
9. [Summary](#summary)

---

### Introduction to Set Operators

**Purpose**: Combine results from multiple queries into a single result set.

**Key Concepts**:
- Set operators work with complete result sets (rows and columns)
- Queries must have the same number of columns in the same order
- Data types should be compatible
- Null values are treated as equal to other null values in comparisons

**Types of Set Operators**:

| Operator | Purpose | Duplicates | Syntax |
|----------|---------|-----------|--------|
| **UNION** | Combine results, remove duplicates | No | ANSI standard |
| **UNION ALL** | Combine results, keep duplicates | Yes | ANSI standard |
| **INTERSECT** | Return only rows in both queries | No | ANSI standard |
| **MINUS** | Return rows in first but not second | No | Oracle |
| **EXCEPT** | Return rows in first but not second | No | MySQL/PostgreSQL/SQL Server |

**Basic Syntax**:
```sql
SELECT column1, column2, ... FROM table1
SET_OPERATOR
SELECT column1, column2, ... FROM table2
[ORDER BY column];
```

---

### UNION

**Purpose**: Combine results from multiple queries and remove duplicate rows.

**Characteristics**:
- Removes duplicate rows automatically
- Slower than UNION ALL (due to duplicate elimination)
- Performs implicit DISTINCT operation
- Columns must be in same order and compatible data types

**Syntax**:
```sql
SELECT column1, column2 FROM table1
UNION
SELECT column1, column2 FROM table2
[ORDER BY column];
```

**Examples**:

```sql
-- Combine employees and contractors
SELECT emp_id, first_name, last_name, 'Employee' AS emp_type
FROM employees
UNION
SELECT contractor_id, first_name, last_name, 'Contractor'
FROM contractors
ORDER BY last_name;
```

```sql
-- Find all unique cities from employees and departments
SELECT city FROM employees
UNION
SELECT city FROM departments
ORDER BY city;
```

```sql
-- Combine active and inactive employees (remove duplicates)
SELECT emp_id, first_name, last_name, 'Active' AS status
FROM employees
WHERE status = 'Active'
UNION
SELECT emp_id, first_name, last_name, 'Inactive'
FROM employees
WHERE status = 'Inactive'
ORDER BY last_name;
```

```sql
-- Get list of all departments that have either employees or budget allocations
SELECT department_name FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
UNION
SELECT department_name FROM budget_allocations b
INNER JOIN departments d ON b.department_id = d.department_id
ORDER BY department_name;
```

```sql
-- Combine sales by region (current year and previous year)
SELECT region, SUM(sales) AS total_sales
FROM sales_2024
GROUP BY region
UNION
SELECT region, SUM(sales)
FROM sales_2023
GROUP BY region
ORDER BY region;
```

**When UNION removes duplicates**:
```sql
-- Example: If John Smith appears in both queries, UNION returns only one row
SELECT first_name, last_name FROM employees WHERE department = 'IT'
UNION
SELECT first_name, last_name FROM employees WHERE salary > 100000;

-- If John Smith is in IT AND earns > 100000, UNION still returns only 1 row (not 2)
```

---

### UNION ALL

**Purpose**: Combine results from multiple queries and keep all duplicate rows.

**Characteristics**:
- Keeps duplicate rows
- Faster than UNION (no duplicate elimination needed)
- Useful when duplicates are expected and wanted
- Better performance for large datasets

**Syntax**:
```sql
SELECT column1, column2 FROM table1
UNION ALL
SELECT column1, column2 FROM table2
[ORDER BY column];
```

**Examples**:

```sql
-- Combine all salary changes from different time periods
SELECT emp_id, salary, 'Q1' AS quarter FROM salary_q1
UNION ALL
SELECT emp_id, salary, 'Q2' FROM salary_q2
UNION ALL
SELECT emp_id, salary, 'Q3' FROM salary_q3
UNION ALL
SELECT emp_id, salary, 'Q4' FROM salary_q4
ORDER BY emp_id, quarter;
```

```sql
-- Combine logs from multiple servers
SELECT log_id, message, 'Server1' AS source FROM server1_logs
UNION ALL
SELECT log_id, message, 'Server2' FROM server2_logs
UNION ALL
SELECT log_id, message, 'Server3' FROM server3_logs
ORDER BY log_id;
```

```sql
-- Get all transactions (including duplicates across sources)
SELECT transaction_id, amount, 'Internal' AS source
FROM internal_transactions
UNION ALL
SELECT transaction_id, amount, 'External'
FROM external_transactions
ORDER BY transaction_id;
```

```sql
-- Stack salary histories for analysis (keep all records)
SELECT emp_id, salary, change_date, 'Current' AS status
FROM employee_salaries
UNION ALL
SELECT emp_id, salary, change_date, 'Historical'
FROM salary_history
ORDER BY emp_id, change_date DESC;
```

**Performance comparison**:
```sql
-- UNION (slower: performs DISTINCT internally)
SELECT salary FROM employees WHERE department = 'IT'
UNION
SELECT salary FROM employees WHERE department = 'HR';

-- UNION ALL (faster: no duplicate removal)
SELECT salary FROM employees WHERE department = 'IT'
UNION ALL
SELECT salary FROM employees WHERE department = 'HR';
```

---

### INTERSECT

**Purpose**: Return only rows that appear in both query results.

**Characteristics**:
- Returns only common rows
- Removes duplicates automatically
- Both queries must have same structure
- Useful for finding overlapping data

**Syntax**:
```sql
SELECT column1, column2 FROM table1
INTERSECT
SELECT column1, column2 FROM table2
[ORDER BY column];
```

**Examples**:

```sql
-- Find employees who are both managers and developers
SELECT emp_id, first_name, last_name FROM employees
WHERE job_title = 'Manager'
INTERSECT
SELECT emp_id, first_name, last_name FROM employees
WHERE job_title = 'Developer'
ORDER BY last_name;
```

```sql
-- Find cities where we have both employees AND offices
SELECT DISTINCT city FROM employees
INTERSECT
SELECT city FROM offices
ORDER BY city;
```

```sql
-- Find products that are in both warehouse A and warehouse B
SELECT product_id, product_name FROM warehouse_a_inventory
INTERSECT
SELECT product_id, product_name FROM warehouse_b_inventory
ORDER BY product_name;
```

```sql
-- Find customers who made purchases in both 2023 and 2024
SELECT DISTINCT customer_id FROM orders
WHERE YEAR(order_date) = 2023
INTERSECT
SELECT DISTINCT customer_id FROM orders
WHERE YEAR(order_date) = 2024
ORDER BY customer_id;
```

```sql
-- Find employees with approved timesheets AND submitted expense reports
SELECT emp_id FROM timesheet_approvals
WHERE status = 'Approved'
INTERSECT
SELECT emp_id FROM expense_reports
WHERE status = 'Submitted'
ORDER BY emp_id;
```

**INTERSECT with multiple columns**:
```sql
-- Find department-location combinations that have both employees and offices
SELECT department_id, location_id FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id
INTERSECT
SELECT department_id, location_id FROM offices
ORDER BY department_id, location_id;
```

---

### MINUS / EXCEPT

**Purpose**: Return rows from first query that do NOT appear in second query.

**Characteristics**:
- Removes duplicates automatically
- Order matters: first query MINUS second query
- MINUS (Oracle) or EXCEPT (MySQL/PostgreSQL/SQL Server)
- Useful for finding differences between datasets

**Syntax**:
```sql
SELECT column1, column2 FROM table1
MINUS              -- Oracle
-- OR --
EXCEPT             -- MySQL/PostgreSQL/SQL Server
SELECT column1, column2 FROM table2
[ORDER BY column];
```

**Examples (Oracle - using MINUS)**:

```sql
-- Find employees NOT in the management team
SELECT emp_id, first_name, last_name FROM employees
MINUS
SELECT emp_id, first_name, last_name FROM managers
ORDER BY last_name;
```

```sql
-- Find customers who haven't made any purchases
SELECT customer_id FROM all_customers
MINUS
SELECT DISTINCT customer_id FROM orders
ORDER BY customer_id;
```

```sql
-- Find products that are in stock but have never been sold
SELECT product_id, product_name FROM inventory
WHERE quantity_on_hand > 0
MINUS
SELECT DISTINCT product_id, product_name FROM sales_orders
ORDER BY product_name;
```

```sql
-- Find employees who are NOT in the HR department
SELECT emp_id, first_name, last_name FROM employees
MINUS
SELECT emp_id, first_name, last_name FROM employees
WHERE department = 'HR'
ORDER BY last_name;
```

```sql
-- Find locations with offices but no employees assigned
SELECT location_id, city FROM locations
WHERE location_id IN (SELECT location_id FROM offices)
MINUS
SELECT location_id, city FROM locations
WHERE location_id IN (SELECT location_id FROM employees)
ORDER BY city;
```

**Examples (MySQL/PostgreSQL/SQL Server - using EXCEPT)**:

```sql
-- MySQL/PostgreSQL/SQL Server equivalent
SELECT emp_id, first_name, last_name FROM employees
EXCEPT
SELECT emp_id, first_name, last_name FROM managers
ORDER BY last_name;
```

```sql
-- Find vendors not currently under contract
SELECT vendor_id FROM all_vendors
EXCEPT
SELECT vendor_id FROM active_contracts
ORDER BY vendor_id;
```

---

### Set Operator Rules and Guidelines

**Rule 1: Same number of columns**
```sql
-- ❌ WRONG: Different number of columns
SELECT emp_id, first_name, last_name FROM employees
UNION
SELECT department_id, department_name FROM departments;
-- Error: inconsistent number of columns

-- ✅ CORRECT: Same number of columns
SELECT emp_id, first_name FROM employees
UNION
SELECT emp_id, first_name FROM contractors;
```

**Rule 2: Compatible data types**
```sql
-- ❌ WRONG: Incompatible data types (if emp_id is numeric, first_name is text)
SELECT emp_id FROM employees
UNION
SELECT first_name FROM employees;
-- Result may be unexpected or error

-- ✅ CORRECT: Convert to compatible types
SELECT CAST(emp_id AS VARCHAR2(10)) FROM employees
UNION
SELECT first_name FROM employees;
```

**Rule 3: Column names from first query**
```sql
-- Column names come from the first SELECT statement
SELECT emp_id AS employee_id, first_name AS emp_first_name FROM employees
UNION
SELECT contractor_id, first_name FROM contractors;
-- Result columns named: employee_id, emp_first_name (from first query)
```

**Rule 4: ORDER BY only at end**
```sql
-- ❌ WRONG: ORDER BY in middle query
SELECT emp_id FROM employees
ORDER BY emp_id
UNION
SELECT contractor_id FROM contractors;
-- Error or unexpected behavior

-- ✅ CORRECT: ORDER BY only at the end
SELECT emp_id FROM employees
UNION
SELECT contractor_id FROM contractors
ORDER BY emp_id;
```

**Rule 5: Use column position or alias in ORDER BY**
```sql
-- ✅ Using column position (1 = first column)
SELECT first_name, last_name FROM employees
UNION
SELECT first_name, last_name FROM contractors
ORDER BY 1, 2;

-- ✅ Using alias from first query
SELECT first_name AS fname, last_name AS lname FROM employees
UNION
SELECT first_name, last_name FROM contractors
ORDER BY fname, lname;
```

---

### Controlling Row Order

**ORDER BY with SET operators**:
```sql
-- Single column ordering
SELECT emp_id, first_name, last_name FROM active_employees
UNION
SELECT emp_id, first_name, last_name FROM inactive_employees
ORDER BY last_name;

-- Multiple column ordering
SELECT department, salary FROM current_employees
UNION
SELECT department, salary FROM former_employees
ORDER BY department, salary DESC;

-- Order by column position
SELECT emp_id, first_name, salary FROM employees
UNION
SELECT contractor_id, first_name, rate FROM contractors
ORDER BY 1;  -- Order by first column (emp_id / contractor_id)

-- Order by expression
SELECT emp_id, first_name, salary FROM employees
UNION
SELECT contractor_id, first_name, daily_rate * 250 FROM contractors
ORDER BY salary DESC;
```

**Sorting direction for each query independently**:
```sql
-- ❌ WRONG: Cannot apply ORDER BY to individual queries
SELECT emp_id FROM employees ORDER BY emp_id DESC
UNION
SELECT contractor_id FROM contractors ORDER BY contractor_id ASC;
-- Only the final ORDER BY applies to entire result

-- ✅ CORRECT: Use subqueries if need different sorting per query (then sort combined result)
SELECT * FROM (
  SELECT emp_id, 'Employee' AS type FROM employees
  UNION ALL
  SELECT contractor_id, 'Contractor' FROM contractors
)
ORDER BY emp_id;
```

---

### Best Practices

1. **Use UNION when removing duplicates is desired**
   ```sql
   -- Find all unique departments that have either employees or projects
   SELECT department_name FROM employees e
   JOIN departments d ON e.department_id = d.department_id
   UNION
   SELECT department_name FROM projects p
   JOIN departments d ON p.department_id = d.department_id;
   ```

2. **Use UNION ALL for better performance when duplicates don't matter**
   ```sql
   -- Combine sales data from multiple quarters (duplicates acceptable)
   SELECT * FROM q1_sales
   UNION ALL
   SELECT * FROM q2_sales
   UNION ALL
   SELECT * FROM q3_sales
   UNION ALL
   SELECT * FROM q4_sales;
   ```

3. **Document the purpose of set operations**
   ```sql
   -- Find active employees and contractors (all staff)
   SELECT emp_id, first_name, 'Employee' AS staff_type FROM employees
   WHERE status = 'Active'
   UNION
   SELECT contractor_id, first_name, 'Contractor' FROM contractors
   WHERE contract_active = 'Y'
   ORDER BY first_name;
   ```

4. **Use compatible data types**
   ```sql
   -- ✅ Good: Both return same data type
   SELECT CAST(emp_id AS NUMBER) AS id, first_name FROM employees
   UNION
   SELECT CAST(contractor_id AS NUMBER), first_name FROM contractors;
   ```

5. **Prefer INTERSECT/MINUS for finding overlaps or differences**
   ```sql
   -- Better than complex WHERE with NOT IN
   SELECT customer_id FROM 2023_customers
   MINUS
   SELECT customer_id FROM 2024_customers;
   -- Gets customers from 2023 not in 2024
   ```

6. **Use meaningful aliases and data source indicators**
   ```sql
   -- Add source identifier for clarity
   SELECT emp_id, first_name, 'Current Employee' AS status, SYSDATE AS snapshot_date
   FROM employees
   UNION ALL
   SELECT emp_id, first_name, 'Terminated', termination_date
   FROM former_employees
   ORDER BY emp_id;
   ```

7. **Test set operations with sample data**
   ```sql
   -- Verify expected result before using in production
   SELECT 'Test' AS data_source, COUNT(*) FROM (
     SELECT emp_id FROM employees
     UNION
     SELECT emp_id FROM contractors
   );
   ```

---

## Summary

In this lesson, you should have learned how to:

1. **Describe set operators**
   - UNION: Combines queries, removes duplicates
   - UNION ALL: Combines queries, keeps duplicates
   - INTERSECT: Returns rows common to both queries
   - MINUS/EXCEPT: Returns rows from first but not second query

2. **Use a set operator to combine multiple queries into a single query**
   - Align column count and data types
   - Use set operator between SELECT statements
   - Apply WHERE/GROUP BY/aggregate functions before combining
   - Column names from first query appear in result

3. **Control the order of rows returned**
   - Use ORDER BY at the end of entire set operation
   - Can reference column position (1, 2, 3...)
   - Can use column alias from first query
   - Can sort by expression or calculation
   - Only one ORDER BY clause for entire result set

**General Syntax Summary**:
```sql
-- Basic set operator pattern
SELECT col1, col2 FROM table1
[WHERE condition] [GROUP BY ...] [HAVING ...]
{UNION | UNION ALL | INTERSECT | MINUS}
SELECT col1, col2 FROM table2
[WHERE condition] [GROUP BY ...] [HAVING ...]
ORDER BY col1 [ASC|DESC];
```

---

**Course**: Oracle 19c SQL Workshop
**Last Updated**: November 18, 2025