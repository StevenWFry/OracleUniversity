# SQL Fundamentals

## Table of Contents
1. [Comparison Operators](#comparison-operators)
2. [WHERE Clause](#where-clause)
3. [Logical Operators: AND, OR, NOT](#logical-operators-and-or-not)
4. [Rules of Precedence](#rules-of-precedence)
5. [BETWEEN Operator](#between-operator)
6. [IN Operator](#in-operator)
7. [LIKE Operator](#like-operator)
8. [Escape Character](#escape-character)
9. [Q Operator](#q-operator-oracle-string-literals)

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
SELECT * FROM employees WHERE salary >= 50000 AND salary <= 100000;
SELECT * FROM employees WHERE department IN ('HR', 'IT', 'Finance');
SELECT * FROM employees WHERE name LIKE 'J%';
SELECT * FROM employees WHERE commission_pct IS NULL;
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
-- Simple condition
SELECT * FROM employees WHERE department = 'IT';

-- Multiple conditions with AND
SELECT * FROM employees WHERE department = 'IT' AND salary > 60000;

-- Multiple conditions with OR
SELECT * FROM employees WHERE department = 'HR' OR department = 'Finance';

-- NOT operator
SELECT * FROM employees WHERE NOT department = 'IT';
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

-- NOT with BETWEEN
SELECT * FROM employees WHERE NOT salary BETWEEN 50000 AND 100000;

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

-- Clarifying complex conditions
SELECT * FROM employees 
WHERE (department = 'IT' OR department = 'HR') 
  AND (salary BETWEEN 50000 AND 100000) 
  AND NOT status = 'Inactive';

-- Multiple levels of nesting
SELECT * FROM employees 
WHERE ((department = 'IT' AND salary > 70000) 
       OR (department = 'Finance' AND salary > 80000))
  AND status = 'Active';
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

-- LIKE with multiple patterns
SELECT * FROM employees WHERE (first_name LIKE 'J%' OR first_name LIKE 'M%') AND department = 'IT';

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
SELECT * FROM notes WHERE content = q'[It's working now]';

-- Nesting with different delimiters
SELECT * FROM logs WHERE message = q'<Can't stop won't stop>';
```

---

**Last Updated**: November 17, 2025
**Course**: Oracle 19c SQL Workshop