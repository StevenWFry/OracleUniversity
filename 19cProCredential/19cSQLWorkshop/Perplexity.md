# Oracle Database 19c: SQL Workshop - Complete Study Notes

## Course Overview

**Course Duration:** 14 hours 41 minutes  
**Number of Modules:** 22  
**Audience:** All Users (Beginner-friendly, ideal for developers)  
**Next Recommended Course:** Oracle Database 19c: PL/SQL Workshop

***

## Module 1: Course Overview

### Course Objectives
After completing this course, you will be able to:
- Identify major components of Oracle Database and MySQL
- Retrieve row and column data from tables with SELECT statement
- Create rows of sorted and restricted data
- Employ SQL functions to generate and retrieve customized data
- Run complex queries to retrieve data from multiple tables
- Run DML (Data Manipulation Language) statements to update data
- Run DDL (Data Definition Language) statements to create and manage schema objects

### Course Structure (7 Units)

**Unit 1:** Retrieving, Restricting, and Sorting Data
- Basic SQL SELECT statement
- WHERE and ORDER BY clauses
- Single-row functions and conversion functions
- Conditional expressions

**Unit 2:** Joins, Subqueries, and Set Operators
- Group functions for aggregating data
- Table joins (displaying data from multiple tables)
- Subqueries to solve queries
- Set operators

**Unit 3:** DML and DDL Statements
- Data Manipulation Language for managing table data
- Data Definition Language for creating objects (tables, views)

**Unit 4:** Data Dictionary, Sequences, and Views
- Data dictionary views
- Sequences, synonyms, and indexes
- Creating views

**Unit 5:** Schema Objects and Subqueries
- Managing schema objects
- Retrieving data using subqueries
- Manipulating data using subqueries

**Unit 6:** Controlling User Access
- GRANT and REVOKE statements

**Unit 7:** Advanced Queries and Time Zones
- Manipulating data using advanced queries
- Managing data in different time zones

***

## Module 2: Introduction

### Learning Objectives
After completing this lesson, you should be able to:
- Define the goals of the course
- List the features of Oracle Database 19c
- Discuss theoretical and physical aspects of a relational database
- Describe Oracle server's implementation of RDBMS and ORDBMS
- Identify development environments that can be used for the course
- Describe the database and schema used in this course

### Oracle Database 19c Features

**Focus Areas:**
- Information Management
- Application Development
- Infrastructure Grids
- Oracle Cloud

**Key Features:**
- Manageability
- Information Integration
- Security
- Performance
- High Availability

### MySQL Overview

**MySQL (also written as MySQL):**
- Modern database for the digital age
- High scalability support
- Examples of scale:
  - Mobile networks with 800+ million subscribers
  - Booking.com: 2 billion events per day
  - Government: IDs for 1 billion citizens
  - Social media: 1.7 billion active users
  - PayPal: 100 terabytes of user data
  - Candy Crush: 850 million gameplays per day

**Supported Operating Systems:**
- Windows
- Linux
- Oracle Solaris
- macOS

**MySQL Enterprise Edition Features:**
- **Scalability:** Thread Pool, Group Replication, InnoDB Cluster
- **High Availability:** MySQL Enterprise HA
- **Security:** Authentication, Audit, Encryption (TDE), Firewall
- **Management Tools:** Enterprise Monitor, Enterprise Backup
- **Development:** MySQL Connectors
- **Administration:** MySQL Workbench, MySQL Utilities

**Support:**
- Oracle Premier Support for MySQL
- World-class support in 29 languages
- 24/7/365 availability
- Unlimited incidents
- Global scale and reach

### Relational Database Concepts

**Relational Database Definition:**
- Proposed by Dr. E.F. Codd in 1970
- Basis for RDBMS
- Collection of objects or relations
- Set of operators to act on relations
- Data integrity for accuracy and consistency

**Relational Database:**
- Collection of two-dimensional tables controlled by the server
- Data stored in table-like format with columns and rows

### Entity Relationship Modeling

**Entity Modeling Convention:**
- **Entity** (Table name): Singular, unique, UPPERCASE, in soft box (rounded corners)
- **Attribute** (Column): Singular, lowercase
  - Mandatory: marked with *
  - Optional: marked with o
  - Primary identifier: marked with #
  - Secondary unique identifier: marked with (#)

**Keys:**
- **Primary Key:** Uniquely identifies each row in a table
- **Foreign Key:** Creates relationships between tables (parent-child relationship)
- **Composite Key:** Combination of two or more columns forming unique identifier

### Relational Database Terminology

1. **Row:** Horizontal stretch across table (also called tuple or record)
2. **Column:** Vertical stretch down table (also called attribute or field name)
3. **Primary Key Column:** First column typically, contains unique, non-null values
4. **Foreign Key Column:** Contains values that reference primary key in another table
5. **Field:** Intersection of row and column (also called cell in spreadsheets)
6. **Null:** Absence of a value (has four different meanings)

### HR Schema Tables Used in Course

**Main Tables:**
- **EMPLOYEES** (Primary key: EMPLOYEE_ID)
  - Contains employee information
  - First 11 chapters: 20 employees
  - From chapter 12 onwards: 107 employees
  - Has hierarchical relationship (employee_id to manager_id)

- **DEPARTMENTS** (Primary key: DEPARTMENT_ID)
  - Department information
  - Relationship to EMPLOYEES table

- **JOBS** (Primary key: JOB_ID)
  - Job titles and salary ranges

- **JOB_HISTORY** (Composite primary key: EMPLOYEE_ID, START_DATE)
  - Historical job assignments

- **LOCATIONS** (Primary key: LOCATION_ID)
  - Geographic locations

- **COUNTRIES** (Primary key: COUNTRY_ID)
  - Country information

- **REGIONS** (Primary key: REGION_ID)
  - Regional groupings

- **JOB_GRADES**
  - Salary grade levels (no defined relationships)

**Table Relationships:**
- EMPLOYEES ↔ DEPARTMENTS (bidirectional)
- DEPARTMENTS → LOCATIONS
- LOCATIONS → COUNTRIES
- COUNTRIES → REGIONS
- EMPLOYEES → JOBS
- EMPLOYEES → JOB_HISTORY
- EMPLOYEES → EMPLOYEES (hierarchical: manager relationship)

### Introduction to SQL

**SQL (Structured Query Language):**
- ANSI standard language for operating relational databases
- Efficient and easy to learn and use
- Functionally complete: define objects, retrieve data, manipulate data

**SQL Statement Categories:**

1. **DML (Data Manipulation Language):**
   - SELECT: Retrieve data
   - INSERT: Add new rows
   - UPDATE: Modify existing data
   - DELETE: Remove rows
   - MERGE: Conditional insert/update

2. **DDL (Data Definition Language):**
   - CREATE: Create objects
   - ALTER: Modify object structure
   - DROP: Remove objects
   - RENAME: Rename objects
   - TRUNCATE: Remove all rows quickly
   - COMMENT: Add comments

3. **DCL (Data Control Language):**
   - GRANT: Give privileges
   - REVOKE: Remove privileges

4. **Transaction Control:**
   - COMMIT: Save changes permanently
   - ROLLBACK: Undo changes
   - SAVEPOINT: Create rollback points

### Development Environments

**For Oracle Database:**
- **SQL Developer** (Recommended for beginners - graphical interface)
- **SQL*Plus** (Command-line tool)
- **Oracle Live SQL** (Free online environment for practice)

**For MySQL:**
- **MySQL Workbench** (Primary tool - graphical interface)
- **mysql** command-line tool

### Course Icons
- **Database cylinder icon:** Output from Oracle database
- **Dolphin icon:** Output from MySQL/MySQL

***

## Module 3: Retrieving Data Using the SQL SELECT Statement

### Key Topics:
- Basic SELECT statement syntax
- Selecting all columns (*)
- Selecting specific columns
- Using column aliases
- Arithmetic expressions
- NULL values in calculations
- Concatenation operator
- DISTINCT keyword to eliminate duplicates
- DESCRIBE command for table structure

***

## Module 4: Restricting and Sorting Data

### Key Topics:
- WHERE clause for filtering rows
- Comparison operators (=, >, <, >=, <=, <>)
- BETWEEN...AND operator
- IN operator
- LIKE operator with wildcards (%, _)
- IS NULL and IS NOT NULL
- Logical operators (AND, OR, NOT)
- ORDER BY clause for sorting
- Ascending (ASC) and descending (DESC) sorting
- Sorting by multiple columns

***

## Module 5: Using Single-Row Functions to Customize Output

### Key Topics:
- Character functions (UPPER, LOWER, INITCAP, CONCAT, SUBSTR, LENGTH, etc.)
- Number functions (ROUND, TRUNC, MOD)
- Date functions (SYSDATE, MONTHS_BETWEEN, ADD_MONTHS, NEXT_DAY, LAST_DAY)
- Working with dates

***

## Module 6: Using Conversion Functions and Conditional Expressions

### Key Topics:
- Implicit and explicit data type conversion
- TO_CHAR function (converting numbers and dates to strings)
- TO_NUMBER function
- TO_DATE function
- Format models
- CASE expressions
- DECODE function
- NVL, NVL2, NULLIF, COALESCE functions

***

## Module 7: Reporting Aggregated Data Using the Group Functions

### Key Topics:
- Group functions: AVG, COUNT, MAX, MIN, SUM
- GROUP BY clause
- HAVING clause (filtering groups)
- Nesting group functions
- Working with multiple columns in GROUP BY

***

## Module 8: Displaying Data from Multiple Tables Using Joins

### Key Topics:
- Natural joins
- INNER JOIN with ON clause
- INNER JOIN with USING clause
- LEFT OUTER JOIN
- RIGHT OUTER JOIN
- FULL OUTER JOIN
- CROSS JOIN (Cartesian product)
- Self-joins

***

## Module 9: Using Subqueries to Solve Queries

### Key Topics:
- Single-row subqueries
- Multiple-row subqueries
- IN, ANY, ALL operators
- Subqueries in WHERE, HAVING, FROM clauses

***

## Module 10: Using Set Operators

### Key Topics:
- UNION: Combines results, eliminates duplicates
- UNION ALL: Combines results, keeps duplicates
- INTERSECT: Returns common rows
- MINUS: Returns rows from first query not in second

***

## Module 11: Managing Tables Using DML Statements

### Key Topics:
- INSERT statement (single row, multiple rows)
- UPDATE statement
- DELETE statement
- MERGE statement
- Transaction control (COMMIT, ROLLBACK, SAVEPOINT)
- Read consistency

***

## Module 12: Introduction to Data Definition Language

### Key Topics:
- CREATE TABLE statement
- Data types (VARCHAR2, NUMBER, DATE, TIMESTAMP, CLOB, BLOB, etc.)
- Constraints (PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK, NOT NULL)
- ALTER TABLE statement
- DROP TABLE statement
- TRUNCATE TABLE statement

***

## Module 13: Introduction to Data Dictionary Views

### Key Topics:
- USER_* views (objects owned by user)
- ALL_* views (objects accessible to user)
- DBA_* views (all database objects)
- Key views: USER_TABLES, USER_CONSTRAINTS, USER_OBJECTS
- Querying data dictionary

***

## Module 14: Creating Sequences, Synonyms, and Indexes

### Key Topics:
- **Sequences:** Generate unique numbers automatically
  - CREATE SEQUENCE syntax
  - NEXTVAL and CURRVAL
- **Indexes:** Improve query performance
  - CREATE INDEX statement
  - When to create indexes
- **Synonyms:** Alternative names for objects
  - Public and private synonyms

***

## Module 15: Creating Views

### Key Topics:
- CREATE VIEW statement
- Simple and complex views
- Benefits of views (security, simplicity, data independence)
- Inline views
- WITH CHECK OPTION
- WITH READ ONLY option

***

## Module 16: Managing Schema Objects

### Key Topics:
- Adding, modifying, dropping columns
- Adding and dropping constraints
- Enabling and disabling constraints
- Renaming objects
- Truncating tables
- Creating and using external tables

***

## Module 17: Retrieving Data by Using Subqueries

### Key Topics (Advanced):
- Multiple-column subqueries
- Pairwise and non-pairwise comparisons
- Scalar subqueries
- Correlated subqueries
- EXISTS and NOT EXISTS operators

***

## Module 18: Manipulating Data by Using Subqueries

### Key Topics:
- Using subqueries to INSERT
- Using subqueries to UPDATE
- Using subqueries to DELETE
- WITH clause (subquery factoring)

***

## Module 19: Controlling User Access

### Key Topics:
- System privileges vs. object privileges
- GRANT statement
- REVOKE statement
- Role-based access control
- Creating and managing roles

***

## Module 20: Manipulating Data Using Advanced Queries

### Key Topics:
- Multi-table INSERT statements
- INSERT ALL
- INSERT FIRST
- Pivoting INSERT
- Advanced UPDATE and DELETE

***

## Module 21: Managing Data in Different Time Zones

### Key Topics:
- TIMESTAMP WITH TIME ZONE
- TIMESTAMP WITH LOCAL TIME ZONE
- TIME ZONE conversions
- CURRENT_DATE vs. SYSDATE
- EXTRACT function for datetime components

***

## Module 22: Conclusion

### Course Summary:
- SQL is powerful and essential for database developers
- Covered SELECT, DML, DDL, joins, subqueries, functions
- Learned to create and manage database objects
- Mastered data retrieval and manipulation techniques

***

## Additional Resources

### Documentation:
- Oracle Database New Features Guide
- Oracle Database Reference
- Oracle Database SQL Language Reference
- Oracle Database Concepts
- SQL Developer User's Guide

### Training Paths:
- **Introductory:** Oracle Database 19c: SQL Workshop (this course)
- **Intermediate:** PL/SQL Fundamentals
- **Advanced:** SQL Tuning for Developers

### Certifications:
- Oracle Database SQL Certified Associate
- Oracle Database PL/SQL Certified Associate
- Oracle Database PL/SQL Certified Professional
- MySQL 5.6 Database Administrator
- MySQL 5.6 Developer

### Online Resources:
- Oracle Learning Library
- Oracle Cloud
- SQL Developer Home Page
- Oracle Live SQL (free practice environment)
- MySQL.com (product information, white papers, webinars)
- dev.mysql.com (Developer Zone, documentation, downloads)
- GitHub (MySQL source code)

***

## Key Takeaways

1. **SQL is functionally complete** - You can define, retrieve, and manipulate data
2. **Relational databases use tables** with rows and columns
3. **Primary keys uniquely identify rows**, foreign keys create relationships
4. **SELECT statement** is fundamental for data retrieval
5. **DML statements** (INSERT, UPDATE, DELETE, MERGE) manipulate data
6. **DDL statements** (CREATE, ALTER, DROP) define database structure
7. **Joins combine data** from multiple related tables
8. **Subqueries** allow complex nested queries
9. **Functions** transform and calculate data
10. **Views** provide security and simplify complex queries
11. **Indexes** improve query performance
12. **Transaction control** ensures data consistency

***

*These study notes cover all 22 modules of the Oracle Database 19c: SQL Workshop course, providing a comprehensive reference for learning SQL fundamentals and advanced techniques.*


I'll expand the study notes with concrete examples for each teaching point across all modules. Let me create a comprehensive version with practical examples.

***

# Oracle Database 19c: SQL Workshop - Complete Study Notes with Examples

## Module 1: Course Overview

### Course Structure Example
```
Timeline:
Week 1-2: Units 1-2 (Basic queries, functions, joins)
Week 3-4: Units 3-4 (DML, DDL, schema objects)
Week 5-6: Units 5-7 (Advanced topics, access control)
```

***

## Module 2: Introduction

### Entity Relationship Model Example
```
Entity: EMPLOYEE
Attributes:
  * employee_id      # (Primary Key - mandatory)
  * first_name       * (mandatory)
  * last_name        * (mandatory)
  o email            o (optional)
  o phone_number     o (optional)
  * hire_date        * (mandatory)
  * job_id           * (mandatory)
  o salary           o (optional)
  o department_id    o (optional - foreign key)
```

### Primary Key and Foreign Key Example
```sql
-- DEPARTMENTS table (Parent)
DEPARTMENT_ID | DEPARTMENT_NAME
90           | Executive
60           | IT
50           | Shipping

-- EMPLOYEES table (Child)
EMPLOYEE_ID | FIRST_NAME | DEPARTMENT_ID (FK)
100         | Steven     | 90
103         | Alexander  | 60
120         | Matthew    | 50

-- The DEPARTMENT_ID in EMPLOYEES must exist in DEPARTMENTS first
```

### Relational Database Terminology Visual
```
EMPLOYEES Table:

     Column 1    Column 2     Column 3      Column 4
     (PK)        
Row 1: 100    |  Steven    |  24000     |  90  (FK)
Row 2: 101    |  Neena     |  17000     |  90
Row 3: 102    |  Lex       |  17000     |  90
Row 4: 103    |  Alexander |  9000      |  60
Row 5: 104    |  Bruce     |  6000      |  60
Row 6: 107    |  Diana     |  4200      |  NULL

Field example: Intersection of Row 1, Column 3 = 24000
Null example: Row 6, Column 4 = NULL (no department assigned)
```

***

## Module 3: Retrieving Data Using the SQL SELECT Statement

### Basic SELECT Statement
```sql
-- Select all columns from a table
SELECT * FROM employees;

-- Select specific columns
SELECT first_name, last_name, salary 
FROM employees;

-- Result:
FIRST_NAME | LAST_NAME | SALARY
Steven     | King      | 24000
Neena      | Kochhar   | 17000
Lex        | De Haan   | 17000
```

### Column Aliases
```sql
-- Using AS keyword
SELECT first_name AS "First Name", 
       last_name AS "Last Name",
       salary AS "Monthly Salary"
FROM employees;

-- Without AS keyword
SELECT first_name "First Name",
       last_name "Last Name",
       salary "Monthly Salary"
FROM employees;

-- Result:
First Name | Last Name | Monthly Salary
Steven     | King      | 24000
Neena      | Kochhar   | 17000
```

### Arithmetic Expressions
```sql
-- Calculate annual salary
SELECT first_name, salary, salary * 12 AS annual_salary
FROM employees;

-- Result:
FIRST_NAME | SALARY | ANNUAL_SALARY
Steven     | 24000  | 288000
Neena      | 17000  | 204000

-- Calculate with commission
SELECT first_name, salary, commission_pct,
       salary * 12 * (1 + commission_pct) AS total_comp
FROM employees;
```

### NULL Values in Calculations
```sql
-- NULL in arithmetic returns NULL
SELECT first_name, salary, commission_pct,
       salary * commission_pct AS commission
FROM employees;

-- Result:
FIRST_NAME | SALARY | COMMISSION_PCT | COMMISSION
Steven     | 24000  | NULL          | NULL
John       | 8200   | 0.25          | 2050
Karen      | 6500   | 0.20          | 1300

-- Any arithmetic with NULL results in NULL
```

### Concatenation Operator
```sql
-- Concatenate with ||
SELECT first_name || ' ' || last_name AS full_name
FROM employees;

-- Result:
FULL_NAME
Steven King
Neena Kochhar
Lex De Haan

-- Complex concatenation
SELECT first_name || ' earns $' || salary || ' per month' AS employee_info
FROM employees;

-- Result:
EMPLOYEE_INFO
Steven earns $24000 per month
Neena earns $17000 per month
```

### DISTINCT Keyword
```sql
-- Show all department IDs (with duplicates)
SELECT department_id FROM employees;
-- Result: 90, 90, 90, 60, 60, 60, 50, 50, 50...

-- Show unique department IDs only
SELECT DISTINCT department_id FROM employees;
-- Result: 90, 60, 50, 80, 70, 100, NULL

-- DISTINCT with multiple columns
SELECT DISTINCT department_id, job_id FROM employees;
-- Shows unique combinations of both columns
```

### DESCRIBE Command
```sql
-- View table structure
DESCRIBE employees;

-- Result:
Name             Null?    Type
EMPLOYEE_ID      NOT NULL NUMBER(6)
FIRST_NAME                VARCHAR2(20)
LAST_NAME        NOT NULL VARCHAR2(25)
EMAIL            NOT NULL VARCHAR2(25)
PHONE_NUMBER              VARCHAR2(20)
HIRE_DATE        NOT NULL DATE
JOB_ID           NOT NULL VARCHAR2(10)
SALARY                    NUMBER(8,2)
COMMISSION_PCT            NUMBER(2,2)
MANAGER_ID                NUMBER(6)
DEPARTMENT_ID             NUMBER(4)
```

***

## Module 4: Restricting and Sorting Data

### WHERE Clause - Comparison Operators
```sql
-- Equal to (=)
SELECT first_name, salary 
FROM employees 
WHERE salary = 9000;

-- Result:
FIRST_NAME | SALARY
Alexander  | 9000

-- Greater than (>)
SELECT first_name, salary 
FROM employees 
WHERE salary > 15000;

-- Result:
FIRST_NAME | SALARY
Steven     | 24000
Neena      | 17000
Lex        | 17000

-- Not equal (<> or !=)
SELECT first_name, department_id 
FROM employees 
WHERE department_id <> 60;
```

### BETWEEN Operator
```sql
-- Find salaries between 5000 and 10000
SELECT first_name, salary 
FROM employees 
WHERE salary BETWEEN 5000 AND 10000;

-- Result:
FIRST_NAME | SALARY
Alexander  | 9000
Bruce      | 6000
David      | 4800
Valli      | 4800

-- NOT BETWEEN
SELECT first_name, salary 
FROM employees 
WHERE salary NOT BETWEEN 5000 AND 10000;
```

### IN Operator
```sql
-- Select specific departments
SELECT first_name, department_id 
FROM employees 
WHERE department_id IN (60, 90, 100);

-- Result:
FIRST_NAME | DEPARTMENT_ID
Steven     | 90
Neena      | 90
Alexander  | 60
Bruce      | 60
Daniel     | 100

-- NOT IN
SELECT first_name, job_id 
FROM employees 
WHERE job_id NOT IN ('IT_PROG', 'ST_CLERK');
```

### LIKE Operator with Wildcards
```sql
-- % matches zero or more characters
-- Names starting with 'S'
SELECT first_name FROM employees 
WHERE first_name LIKE 'S%';

-- Result: Steven, Susan, Sarah

-- Names ending with 'en'
SELECT first_name FROM employees 
WHERE first_name LIKE '%en';

-- Result: Steven, Karen

-- Names containing 'ar'
SELECT first_name FROM employees 
WHERE first_name LIKE '%ar%';

-- Result: Karen, Sarah, Barbara, Maria

-- _ matches exactly one character
-- Names with 'a' as second letter
SELECT first_name FROM employees 
WHERE first_name LIKE '_a%';

-- Result: David, Daniel, Karen, Sarah

-- Four-letter names starting with 'J'
SELECT first_name FROM employees 
WHERE first_name LIKE 'J___';

-- Result: John, Jose
```

### IS NULL / IS NOT NULL
```sql
-- Find employees without commission
SELECT first_name, commission_pct 
FROM employees 
WHERE commission_pct IS NULL;

-- Result:
FIRST_NAME | COMMISSION_PCT
Steven     | NULL
Neena      | NULL
Lex        | NULL

-- Find employees with commission
SELECT first_name, commission_pct 
FROM employees 
WHERE commission_pct IS NOT NULL;

-- Result:
FIRST_NAME | COMMISSION_PCT
John       | 0.25
Karen      | 0.20
Alberto    | 0.30
```

### Logical Operators
```sql
-- AND - Both conditions must be true
SELECT first_name, salary, department_id 
FROM employees 
WHERE salary > 10000 AND department_id = 90;

-- Result:
FIRST_NAME | SALARY | DEPARTMENT_ID
Steven     | 24000  | 90
Neena      | 17000  | 90
Lex        | 17000  | 90

-- OR - At least one condition must be true
SELECT first_name, salary, department_id 
FROM employees 
WHERE salary > 15000 OR department_id = 60;

-- Result includes high earners OR IT dept employees

-- NOT - Negates condition
SELECT first_name, job_id 
FROM employees 
WHERE NOT job_id = 'IT_PROG';

-- Complex combination
SELECT first_name, salary, department_id, job_id
FROM employees
WHERE (salary > 10000 AND department_id = 90)
   OR (job_id = 'IT_PROG' AND salary > 8000);
```

### ORDER BY Clause
```sql
-- Sort ascending (default)
SELECT first_name, salary 
FROM employees 
ORDER BY salary;

-- Result (lowest to highest):
FIRST_NAME | SALARY
Diana      | 4200
Valli      | 4800
David      | 4800
Bruce      | 6000
...

-- Sort descending
SELECT first_name, salary 
FROM employees 
ORDER BY salary DESC;

-- Result (highest to lowest):
FIRST_NAME | SALARY
Steven     | 24000
Neena      | 17000
Lex        | 17000
...

-- Sort by multiple columns
SELECT first_name, department_id, salary 
FROM employees 
ORDER BY department_id ASC, salary DESC;

-- Result: Groups by dept, then highest salary first in each dept

-- Sort by column alias
SELECT first_name, salary, salary * 12 AS annual 
FROM employees 
ORDER BY annual DESC;

-- Sort by column position (not recommended)
SELECT first_name, salary 
FROM employees 
ORDER BY 2 DESC;  -- Sorts by 2nd column (salary)
```

### Combined Example
```sql
-- Complex query with WHERE and ORDER BY
SELECT first_name, last_name, salary, department_id
FROM employees
WHERE department_id IN (60, 90, 100)
  AND salary BETWEEN 5000 AND 20000
  AND first_name LIKE 'A%'
ORDER BY department_id, salary DESC;

-- Result: Shows employees whose:
-- - Department is 60, 90, or 100
-- - Salary is between 5000 and 20000
-- - Name starts with 'A'
-- Sorted by department, then highest salary first
```

***

## Module 5: Using Single-Row Functions to Customize Output

### Character Functions - Case Conversion
```sql
-- UPPER - Convert to uppercase
SELECT UPPER(first_name) FROM employees;
-- Result: STEVEN, NEENA, LEX

-- LOWER - Convert to lowercase
SELECT LOWER(first_name) FROM employees;
-- Result: steven, neena, lex

-- INITCAP - Capitalize first letter of each word
SELECT INITCAP('steven KING') FROM dual;
-- Result: Steven King

-- Practical use in WHERE clause
SELECT first_name, last_name 
FROM employees 
WHERE UPPER(last_name) = 'KING';
-- Finds 'King', 'KING', 'king', etc.
```

### Character Functions - Manipulation
```sql
-- CONCAT - Join two strings
SELECT CONCAT(first_name, last_name) FROM employees;
-- Result: StevenKing, NeenaKochhar

SELECT CONCAT(CONCAT(first_name, ' '), last_name) FROM employees;
-- Result: Steven King, Neena Kochhar

-- SUBSTR - Extract substring
SELECT SUBSTR('Hello World', 1, 5) FROM dual;
-- Result: Hello
-- SUBSTR(string, start_position, length)

SELECT first_name, SUBSTR(first_name, 1, 3) AS nickname
FROM employees;
-- Result: Steven -> Ste, Alexander -> Ale

-- LENGTH - Get string length
SELECT first_name, LENGTH(first_name) AS name_length
FROM employees;

-- Result:
FIRST_NAME | NAME_LENGTH
Steven     | 6
Alexander  | 9
Lex        | 3

-- INSTR - Find position of substring
SELECT INSTR('Hello World', 'World') FROM dual;
-- Result: 7 (position where 'World' starts)

-- LPAD - Left pad with characters
SELECT LPAD(salary, 10, '*') FROM employees;
-- Result: ****24000, ****17000
-- LPAD(string, total_length, pad_character)

-- RPAD - Right pad
SELECT RPAD(first_name, 15, '.') FROM employees;
-- Result: Steven........., Neena..........

-- TRIM - Remove leading/trailing characters
SELECT TRIM('  Hello  ') FROM dual;
-- Result: Hello

SELECT TRIM('x' FROM 'xxxHelloxxx') FROM dual;
-- Result: Hello

-- REPLACE - Replace substring
SELECT REPLACE(phone_number, '.', '-') FROM employees;
-- Result: 515.123.4567 -> 515-123-4567
```

### Number Functions
```sql
-- ROUND - Round to specified decimal places
SELECT ROUND(45.926, 2) FROM dual;
-- Result: 45.93

SELECT ROUND(45.926, 0) FROM dual;
-- Result: 46

SELECT ROUND(45.926, -1) FROM dual;
-- Result: 50 (rounds to nearest 10)

-- TRUNC - Truncate to specified decimal places
SELECT TRUNC(45.926, 2) FROM dual;
-- Result: 45.92

SELECT TRUNC(45.926, 0) FROM dual;
-- Result: 45

-- MOD - Modulus (remainder)
SELECT MOD(10, 3) FROM dual;
-- Result: 1 (10 divided by 3 = 3 remainder 1)

-- Practical: Find odd/even employee IDs
SELECT employee_id, 
       CASE WHEN MOD(employee_id, 2) = 0 
            THEN 'Even' 
            ELSE 'Odd' 
       END AS id_type
FROM employees;

-- CEIL - Round up
SELECT CEIL(45.1) FROM dual;
-- Result: 46

-- FLOOR - Round down
SELECT FLOOR(45.9) FROM dual;
-- Result: 45

-- ABS - Absolute value
SELECT ABS(-45) FROM dual;
-- Result: 45

-- POWER - Raise to power
SELECT POWER(2, 3) FROM dual;
-- Result: 8 (2^3)

-- SQRT - Square root
SELECT SQRT(16) FROM dual;
-- Result: 4
```

### Date
## Module 5: Using Single-Row Functions (Continued)

### Date Functions
```sql
-- SYSDATE - Current date and time
SELECT SYSDATE FROM dual;
-- Result: 21-NOV-25 (current date)

-- Date arithmetic - add/subtract days
SELECT hire_date, 
       hire_date + 7 AS "One Week Later",
       hire_date - 30 AS "30 Days Before"
FROM employees;

-- Result:
HIRE_DATE  | One Week Later | 30 Days Before
17-JUN-03  | 24-JUN-03     | 18-MAY-03
21-SEP-05  | 28-SEP-05     | 22-AUG-05

-- Subtract dates to get difference in days
SELECT first_name, 
       ROUND(SYSDATE - hire_date) AS days_employed
FROM employees;

-- Result:
FIRST_NAME | DAYS_EMPLOYED
Steven     | 8192
Neena      | 7396

-- MONTHS_BETWEEN - Difference in months
SELECT first_name,
       ROUND(MONTHS_BETWEEN(SYSDATE, hire_date)) AS months_employed
FROM employees;

-- Result:
FIRST_NAME | MONTHS_EMPLOYED
Steven     | 269
Neena      | 243

-- ADD_MONTHS - Add months to date
SELECT hire_date,
       ADD_MONTHS(hire_date, 6) AS "6 Months Later"
FROM employees;

-- Result:
HIRE_DATE  | 6 Months Later
17-JUN-03  | 17-DEC-03
21-SEP-05  | 21-MAR-06

-- NEXT_DAY - Next occurrence of specified day
SELECT SYSDATE,
       NEXT_DAY(SYSDATE, 'MONDAY') AS "Next Monday"
FROM dual;

-- Result:
SYSDATE    | Next Monday
21-NOV-25  | 24-NOV-25

-- LAST_DAY - Last day of month
SELECT hire_date,
       LAST_DAY(hire_date) AS "End of Month"
FROM employees;

-- Result:
HIRE_DATE  | End of Month
17-JUN-03  | 30-JUN-03
21-SEP-05  | 30-SEP-05

-- ROUND - Round date
SELECT SYSDATE,
       ROUND(SYSDATE, 'MONTH') AS rounded_to_month,
       ROUND(SYSDATE, 'YEAR') AS rounded_to_year
FROM dual;

-- TRUNC - Truncate date
SELECT SYSDATE,
       TRUNC(SYSDATE, 'MONTH') AS first_of_month,
       TRUNC(SYSDATE, 'YEAR') AS first_of_year
FROM dual;
```

***

## Module 6: Using Conversion Functions and Conditional Expressions

### Implicit Data Type Conversion
```sql
-- Oracle automatically converts in some cases
SELECT * FROM employees 
WHERE employee_id = '100';  -- String '100' converted to number

SELECT * FROM employees 
WHERE hire_date = '17-JUN-03';  -- String converted to date

-- Can lead to performance issues and unexpected results
```

### TO_CHAR Function - Numbers
```sql
-- Convert number to formatted string
SELECT first_name,
       TO_CHAR(salary, '$99,999.99') AS formatted_salary
FROM employees;

-- Result:
FIRST_NAME | FORMATTED_SALARY
Steven     |  $24,000.00
Neena      |  $17,000.00

-- Format elements:
-- 9 = digit position
-- 0 = display zero
-- $ = dollar sign
-- , = comma
-- . = decimal point

SELECT TO_CHAR(1234.56, '00000.00') FROM dual;
-- Result: 01234.56

SELECT TO_CHAR(1234.56, 'L9,999.99') FROM dual;
-- Result: $1,234.56 (L = local currency symbol)

SELECT TO_CHAR(1234.56, '9999.99PR') FROM dual;
-- Result: 1234.56 (PR shows negative in <angle brackets>)

SELECT TO_CHAR(-1234.56, '9999.99PR') FROM dual;
-- Result: <1234.56>
```

### TO_CHAR Function - Dates
```sql
-- Basic date formatting
SELECT first_name,
       TO_CHAR(hire_date, 'DD-MON-YYYY') AS formatted_date
FROM employees;

-- Result:
FIRST_NAME | FORMATTED_DATE
Steven     | 17-JUN-2003
Neena      | 21-SEP-2005

-- Common format elements:
-- YYYY = 4-digit year
-- YY = 2-digit year
-- MM = 2-digit month
-- MON = 3-letter month abbreviation
-- MONTH = full month name
-- DD = day of month
-- DY = 3-letter day abbreviation
-- DAY = full day name

SELECT TO_CHAR(SYSDATE, 'Day, Month DD, YYYY') FROM dual;
-- Result: Friday, November 21, 2025

SELECT TO_CHAR(SYSDATE, 'DD/MM/YYYY HH24:MI:SS') FROM dual;
-- Result: 21/11/2025 11:00:00

SELECT TO_CHAR(hire_date, 'fmDay, fmMonth DD, YYYY') 
FROM employees;
-- fm removes padding spaces
-- Result: Friday, November 21, 2025 (no extra spaces)

-- Time formats
SELECT TO_CHAR(SYSDATE, 'HH:MI:SS AM') FROM dual;
-- Result: 11:00:00 AM

SELECT TO_CHAR(SYSDATE, 'HH24:MI:SS') FROM dual;
-- Result: 11:00:00

-- Day of year
SELECT TO_CHAR(SYSDATE, 'DDD') FROM dual;
-- Result: 325 (day 325 of the year)

-- Week of year
SELECT TO_CHAR(SYSDATE, 'WW') FROM dual;
-- Result: 47 (week 47 of the year)

-- Quarter
SELECT TO_CHAR(SYSDATE, 'Q') FROM dual;
-- Result: 4 (4th quarter)
```

### TO_NUMBER Function
```sql
-- Convert string to number
SELECT TO_NUMBER('1234.56') FROM dual;
-- Result: 1234.56

SELECT TO_NUMBER('$1,234.56', '$9,999.99') FROM dual;
-- Result: 1234.56

-- Use in calculations
SELECT TO_NUMBER('100') + TO_NUMBER('200') FROM dual;
-- Result: 300

-- Practical example
SELECT first_name, salary
FROM employees
WHERE salary > TO_NUMBER('10000');
```

### TO_DATE Function
```sql
-- Convert string to date
SELECT TO_DATE('21-NOV-2025', 'DD-MON-YYYY') FROM dual;
-- Result: 21-NOV-25

SELECT TO_DATE('November 21, 2025', 'Month DD, YYYY') FROM dual;
-- Result: 21-NOV-25

SELECT TO_DATE('2025/11/21', 'YYYY/MM/DD') FROM dual;
-- Result: 21-NOV-25

-- Use in WHERE clause
SELECT first_name, hire_date
FROM employees
WHERE hire_date > TO_DATE('01-JAN-2005', 'DD-MON-YYYY');

-- With time component
SELECT TO_DATE('21-11-2025 14:30:00', 'DD-MM-YYYY HH24:MI:SS') 
FROM dual;
```

### NVL Function
```sql
-- Replace NULL with specified value
SELECT first_name, 
       commission_pct,
       NVL(commission_pct, 0) AS commission_with_default
FROM employees;

-- Result:
FIRST_NAME | COMMISSION_PCT | COMMISSION_WITH_DEFAULT
Steven     | NULL          | 0
John       | 0.25          | 0.25
Karen      | 0.20          | 0.20

-- Calculate total compensation
SELECT first_name,
       salary,
       salary * 12 * (1 + NVL(commission_pct, 0)) AS annual_comp
FROM employees;

-- NVL with strings
SELECT first_name,
       department_id,
       NVL(TO_CHAR(department_id), 'No Dept') AS dept_display
FROM employees;
```

### NVL2 Function
```sql
-- NVL2(expr, value_if_not_null, value_if_null)
SELECT first_name,
       commission_pct,
       NVL2(commission_pct, 'Has Commission', 'No Commission') AS status
FROM employees;

-- Result:
FIRST_NAME | COMMISSION_PCT | STATUS
Steven     | NULL          | No Commission
John       | 0.25          | Has Commission

-- Complex calculation
SELECT first_name,
       salary,
       NVL2(commission_pct, 
            salary + (salary * commission_pct),
            salary) AS total_salary
FROM employees;
```

### NULLIF Function
```sql
-- NULLIF(expr1, expr2) - Returns NULL if equal, else returns expr1
SELECT first_name,
       LENGTH(first_name) AS name_length,
       LENGTH(last_name) AS last_length,
       NULLIF(LENGTH(first_name), LENGTH(last_name)) AS length_diff
FROM employees;

-- Result:
FIRST_NAME | NAME_LENGTH | LAST_LENGTH | LENGTH_DIFF
Steven     | 6          | 4           | 6
John       | 4          | 4           | NULL (same length)
Alexander  | 9          | 5           | 9
```

### COALESCE Function
```sql
-- COALESCE returns first non-null value
SELECT first_name,
       COALESCE(commission_pct, manager_id, department_id, 0) AS first_non_null
FROM employees;

-- Practical: Build full address with available parts
SELECT COALESCE(street_address, city, state_province, 'No Address') 
AS address
FROM locations;

-- Multiple fallback values
SELECT employee_id,
       COALESCE(email, phone_number, 'No Contact Info') AS contact
FROM employees;
```

### CASE Expression
```sql
-- Simple CASE
SELECT first_name,
       department_id,
       CASE department_id
           WHEN 90 THEN 'Executive'
           WHEN 60 THEN 'IT'
           WHEN 50 THEN 'Shipping'
           WHEN 80 THEN 'Sales'
           ELSE 'Other'
       END AS department_name
FROM employees;

-- Result:
FIRST_NAME | DEPARTMENT_ID | DEPARTMENT_NAME
Steven     | 90           | Executive
Alexander  | 60           | IT
Matthew    | 50           | Shipping

-- Searched CASE (more flexible)
SELECT first_name,
       salary,
       CASE
           WHEN salary < 5000 THEN 'Low'
           WHEN salary BETWEEN 5000 AND 10000 THEN 'Medium'
           WHEN salary BETWEEN 10001 AND 15000 THEN 'High'
           ELSE 'Very High'
       END AS salary_grade
FROM employees;

-- Result:
FIRST_NAME | SALARY | SALARY_GRADE
Steven     | 24000  | Very High
Diana      | 4200   | Low
Bruce      | 6000   | Medium

-- Complex CASE
SELECT first_name,
       hire_date,
       salary,
       CASE
           WHEN MONTHS_BETWEEN(SYSDATE, hire_date) > 240 
                AND salary > 10000 THEN 'Senior High Paid'
           WHEN MONTHS_BETWEEN(SYSDATE, hire_date) > 240 THEN 'Senior'
           WHEN salary > 10000 THEN 'High Paid'
           ELSE 'Regular'
       END AS employee_category
FROM employees;

-- Use in ORDER BY
SELECT first_name, department_id
FROM employees
ORDER BY CASE department_id
             WHEN 90 THEN 1
             WHEN 60 THEN 2
             ELSE 3
         END;
```

### DECODE Function (Oracle-specific)
```sql
-- DECODE(expression, search1, result1, search2, result2, ..., default)
SELECT first_name,
       department_id,
       DECODE(department_id,
              90, 'Executive',
              60, 'IT',
              50, 'Shipping',
              80, 'Sales',
              'Other') AS department_name
FROM employees;

-- Same as the simple CASE example above

-- With calculations
SELECT first_name,
       job_id,
       salary,
       DECODE(job_id,
              'IT_PROG', salary * 1.10,
              'ST_CLERK', salary * 1.15,
              'SA_REP', salary * 1.20,
              salary) AS new_salary
FROM employees;

-- Result:
FIRST_NAME | JOB_ID    | SALARY | NEW_SALARY
Alexander  | IT_PROG   | 9000   | 9900
Kevin      | ST_CLERK  | 3100   | 3565
John       | SA_REP    | 8200   | 9840
Steven     | AD_PRES   | 24000  | 24000
```

***

## Module 7: Reporting Aggregated Data Using the Group Functions

### COUNT Function
```sql
-- Count all rows
SELECT COUNT(*) AS total_employees
FROM employees;
-- Result: 20 (or 107 in full dataset)

-- Count non-null values in a column
SELECT COUNT(commission_pct) AS employees_with_commission
FROM employees;
-- Result: 6 (only counts non-null commission values)

-- Count distinct values
SELECT COUNT(DISTINCT department_id) AS number_of_departments
FROM employees;
-- Result: 7 (unique departments)

-- Comparison
SELECT COUNT(*) AS all_rows,
       COUNT(commission_pct) AS with_commission,
       COUNT(DISTINCT department_id) AS unique_depts
FROM employees;

-- Result:
ALL_ROWS | WITH_COMMISSION | UNIQUE_DEPTS
20       | 6              | 7
```

### SUM Function
```sql
-- Total of all salaries
SELECT SUM(salary) AS total_salary_cost
FROM employees;
-- Result: 156400

-- Sum with condition (using CASE)
SELECT SUM(CASE WHEN department_id = 60 THEN salary END) AS it_dept_total
FROM employees;
-- Result: 19200

-- Multiple sums
SELECT SUM(salary) AS total_salary,
       SUM(commission_pct * salary) AS total_commission
FROM employees;
```

### AVG Function
```sql
-- Average salary
SELECT AVG(salary) AS average_salary
FROM employees;
-- Result: 7820 (total/count)

-- AVG ignores NULL values
SELECT AVG(commission_pct) AS avg_commission
FROM employees;
-- Result: 0.225 (average of non-null values only)

-- Average with ROUND
SELECT ROUND(AVG(salary), 2) AS avg_salary
FROM employees;
-- Result: 7820.00

-- Compare with including NULLs as zero
SELECT AVG(commission_pct) AS avg_excluding_null,
       AVG(NVL(commission_pct, 0)) AS avg_including_null
FROM employees;

-- Result:
AVG_EXCLUDING_NULL | AVG_INCLUDING_NULL
0.225             | 0.0675
```

### MAX and MIN Functions
```sql
-- Highest and lowest salaries
SELECT MAX(salary) AS highest_salary,
       MIN(salary) AS lowest_salary
FROM employees;

--
## Module 7: Reporting Aggregated Data (Continued)

### MAX and MIN Functions (Continued)
```sql
-- Highest and lowest salaries
SELECT MAX(salary) AS highest_salary,
       MIN(salary) AS lowest_salary
FROM employees;

-- Result:
HIGHEST_SALARY | LOWEST_SALARY
24000         | 4200

-- Works with dates
SELECT MIN(hire_date) AS first_hired,
       MAX(hire_date) AS last_hired
FROM employees;

-- Result:
FIRST_HIRED | LAST_HIRED
17-JUN-03   | 21-APR-08

-- Works with strings (alphabetical)
SELECT MIN(first_name) AS first_alphabetically,
       MAX(first_name) AS last_alphabetically
FROM employees;

-- Result:
FIRST_ALPHABETICALLY | LAST_ALPHABETICALLY
Alexander           | William
```

### GROUP BY Clause
```sql
-- Average salary by department
SELECT department_id,
       AVG(salary) AS avg_dept_salary
FROM employees
GROUP BY department_id;

-- Result:
DEPARTMENT_ID | AVG_DEPT_SALARY
90           | 19333.33
60           | 6400.00
50           | 3500.00
80           | 10033.33

-- Count employees by department
SELECT department_id,
       COUNT(*) AS employee_count
FROM employees
GROUP BY department_id;

-- Result:
DEPARTMENT_ID | EMPLOYEE_COUNT
90           | 3
60           | 3
50           | 5
80           | 6
NULL         | 1

-- Multiple group columns
SELECT department_id, 
       job_id,
       COUNT(*) AS emp_count,
       AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id, job_id
ORDER BY department_id, job_id;

-- Result:
DEPARTMENT_ID | JOB_ID    | EMP_COUNT | AVG_SALARY
50           | ST_CLERK  | 5         | 3500
60           | IT_PROG   | 3         | 6400
80           | SA_MAN    | 1         | 14000
80           | SA_REP    | 5         | 9350
90           | AD_PRES   | 1         | 24000
90           | AD_VP     | 2         | 17000

-- All aggregate functions together
SELECT department_id,
       COUNT(*) AS emp_count,
       SUM(salary) AS total_salary,
       AVG(salary) AS avg_salary,
       MIN(salary) AS min_salary,
       MAX(salary) AS max_salary
FROM employees
GROUP BY department_id
ORDER BY department_id;
```

### HAVING Clause
```sql
-- Filter groups (not individual rows)
-- Show departments with average salary > 10000
SELECT department_id,
       AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 10000;

-- Result:
DEPARTMENT_ID | AVG_SALARY
90           | 19333.33
80           | 10033.33

-- Show departments with more than 3 employees
SELECT department_id,
       COUNT(*) AS employee_count
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 3;

-- Result:
DEPARTMENT_ID | EMPLOYEE_COUNT
50           | 5
80           | 6

-- Combine WHERE and HAVING
-- WHERE filters before grouping, HAVING filters after
SELECT department_id,
       AVG(salary) AS avg_salary
FROM employees
WHERE job_id != 'ST_CLERK'  -- Filter rows before grouping
GROUP BY department_id
HAVING AVG(salary) > 8000   -- Filter groups after aggregation
ORDER BY avg_salary DESC;

-- Multiple conditions in HAVING
SELECT department_id,
       COUNT(*) AS emp_count,
       AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
HAVING COUNT(*) >= 3 AND AVG(salary) > 5000;

-- Using aggregate functions in HAVING
SELECT job_id,
       SUM(salary) AS total_salary,
       COUNT(*) AS emp_count
FROM employees
GROUP BY job_id
HAVING SUM(salary) > 20000 AND COUNT(*) > 2
ORDER BY total_salary DESC;
```

### Nesting Group Functions
```sql
-- Maximum of the average salaries
SELECT MAX(AVG(salary)) AS highest_avg_salary
FROM employees
GROUP BY department_id;

-- Result: 19333.33 (the highest average among all departments)

-- Minimum of the sums
SELECT MIN(SUM(salary)) AS lowest_dept_total
FROM employees
GROUP BY department_id;

-- Average of the counts
SELECT AVG(COUNT(*)) AS avg_employees_per_dept
FROM employees
GROUP BY department_id;

-- Cannot nest more than two levels
-- This is INVALID:
-- SELECT MAX(AVG(MIN(salary))) FROM employees;
```

### Complete GROUP BY Example
```sql
-- Comprehensive query showing multiple concepts
SELECT 
    d.department_name,
    COUNT(e.employee_id) AS total_employees,
    COUNT(e.commission_pct) AS employees_with_commission,
    SUM(e.salary) AS total_salary,
    ROUND(AVG(e.salary), 2) AS avg_salary,
    MIN(e.salary) AS min_salary,
    MAX(e.salary) AS max_salary,
    ROUND(AVG(MONTHS_BETWEEN(SYSDATE, e.hire_date))) AS avg_months_employed
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE e.salary > 3000
GROUP BY d.department_name
HAVING COUNT(e.employee_id) > 2
ORDER BY avg_salary DESC;

-- Result shows departments with detailed statistics
```

***

## Module 8: Displaying Data from Multiple Tables Using Joins

### Natural Join
```sql
-- Automatically joins on columns with same name
SELECT e.first_name, e.salary, d.department_name
FROM employees e
NATURAL JOIN departments d;

-- Oracle finds DEPARTMENT_ID in both tables and joins on it
-- Result:
FIRST_NAME | SALARY | DEPARTMENT_NAME
Steven     | 24000  | Executive
Neena      | 17000  | Executive
Alexander  | 9000   | IT
```

### INNER JOIN with USING Clause
```sql
-- Explicitly specify the join column
SELECT e.first_name, e.salary, d.department_name
FROM employees e
JOIN departments d USING (department_id);

-- Same result as Natural Join but more explicit
-- Note: Don't use table alias with column in USING clause

-- Multiple table join with USING
SELECT e.first_name, d.department_name, l.city
FROM employees e
JOIN departments d USING (department_id)
JOIN locations l USING (location_id);

-- Result:
FIRST_NAME | DEPARTMENT_NAME | CITY
Steven     | Executive      | Seattle
Neena      | Executive      | Seattle
Alexander  | IT             | Southlake
```

### INNER JOIN with ON Clause
```sql
-- Most flexible join syntax
SELECT e.first_name, e.salary, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id;

-- Result:
FIRST_NAME | SALARY | DEPARTMENT_NAME
Steven     | 24000  | Executive
Neena      | 17000  | Executive
Alexander  | 9000   | IT
Bruce      | 6000   | IT

-- Join with additional conditions
SELECT e.first_name, e.salary, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE e.salary > 10000;

-- Join condition in ON, filter in WHERE
SELECT e.first_name, e.salary, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id 
                   AND d.department_name = 'IT';

-- Three-table join
SELECT e.first_name, 
       d.department_name,
       l.city,
       c.country_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id
JOIN locations l ON d.location_id = l.location_id
JOIN countries c ON l.country_id = c.country_id;

-- Result:
FIRST_NAME | DEPARTMENT_NAME | CITY      | COUNTRY_NAME
Steven     | Executive      | Seattle   | United States
Alexander  | IT             | Southlake | United States
```

### LEFT OUTER JOIN
```sql
-- Returns all rows from left table, matching rows from right
SELECT e.first_name, e.department_id, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id;

-- Result includes employees WITHOUT departments:
FIRST_NAME | DEPARTMENT_ID | DEPARTMENT_NAME
Steven     | 90           | Executive
Diana      | NULL         | NULL
Alexander  | 60           | IT

-- Show all departments, even those without employees
SELECT d.department_name, e.first_name
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
ORDER BY d.department_name;

-- Result:
DEPARTMENT_NAME | FIRST_NAME
Accounting      | Shelley
Admin           | Jennifer
Executive       | Steven
Executive       | Neena
IT              | Alexander
IT              | Bruce
Marketing       | NULL  -- No employees in Marketing
Treasury        | NULL  -- No employees in Treasury
```

### RIGHT OUTER JOIN
```sql
-- Returns all rows from right table, matching rows from left
SELECT e.first_name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id;

-- Same as LEFT JOIN with tables reversed
-- Result shows all departments, even without employees

-- Practical example: Find departments with no employees
SELECT d.department_id, d.department_name, COUNT(e.employee_id) AS emp_count
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.department_id
GROUP BY d.department_id, d.department_name
HAVING COUNT(e.employee_id) = 0;

-- Result:
DEPARTMENT_ID | DEPARTMENT_NAME | EMP_COUNT
120          | Treasury        | 0
130          | Corporate Tax   | 0
```

### FULL OUTER JOIN
```sql
-- Returns all rows from both tables
SELECT e.first_name, e.department_id, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.department_id;

-- Result includes:
-- - Employees without departments
-- - Departments without employees
-- - Matched rows

FIRST_NAME | DEPARTMENT_ID | DEPARTMENT_NAME
Steven     | 90           | Executive
Diana      | NULL         | NULL  -- Employee without dept
NULL       | 120          | Treasury  -- Dept without employees
Alexander  | 60           | IT

-- Find orphaned records
SELECT 
    CASE 
        WHEN e.employee_id IS NULL THEN 'Dept without employees'
        WHEN d.department_id IS NULL THEN 'Employee without dept'
        ELSE 'Matched'
    END AS record_status,
    e.first_name,
    d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.department_id;
```

### CROSS JOIN (Cartesian Product)
```sql
-- Every row from first table matched with every row from second
SELECT e.first_name, d.department_name
FROM employees e
CROSS JOIN departments d;

-- If employees has 20 rows and departments has 27 rows
-- Result has 20 × 27 = 540 rows

-- Practical use: Generate combinations
SELECT 
    e.first_name,
    j.job_title
FROM employees e
CROSS JOIN jobs j
WHERE e.employee_id <= 3  -- Limit for demonstration
ORDER BY e.first_name, j.job_title;

-- Shows all possible job assignments for first 3 employees

-- Alternative syntax (without CROSS JOIN keyword)
SELECT e.first_name, d.department_name
FROM employees e, departments d;
-- Same as CROSS JOIN (old Oracle syntax)
```

### Self-Join
```sql
-- Join table to itself
-- Find employees and their managers
SELECT 
    e.first_name AS employee_name,
    e.job_id AS employee_job,
    m.first_name AS manager_name,
    m.job_id AS manager_job
FROM employees e
JOIN employees m ON e.manager_id = m.employee_id;

-- Result:
EMPLOYEE_NAME | EMPLOYEE_JOB | MANAGER_NAME | MANAGER_JOB
Neena        | AD_VP        | Steven       | AD_PRES
Lex          | AD_VP        | Steven       | AD_PRES
Alexander    | IT_PROG      | Bruce        | IT_PROG
Bruce        | IT_PROG      | Lex          | AD_VP

-- Show organizational hierarchy
SELECT 
    LPAD(' ', 2 * LEVEL - 2) || e.first_name AS hierarchy,
    e.job_id,
    LEVEL AS org_level
FROM employees e
START WITH e.manager_id IS NULL  -- Start with CEO
CONNECT BY PRIOR e.employee_id = e.manager_id
ORDER SIBLINGS BY e.first_name;

-- Result shows tree structure:
HIERARCHY        | JOB_ID   | ORG_LEVEL
Steven          | AD_PRES  | 1
  Neena         | AD_VP    | 2
    Nancy       | FI_MGR   | 3
      Daniel    | FI_ACCOUNT | 4
  Lex           | AD_VP    | 2
    Alexander   | IT_PROG  | 3
```

### Complex Join Example
```sql
-- Comprehensive multi-table join
SELECT 
    e.employee_id,
    e.first_name || ' ' || e.last_name AS employee_name,
    e.salary,
    j.job_title,
    d.department_name,
    l.city,
    c.country_name,
    r.region_name,
    m.first_name || ' ' || m.last_name AS manager_name
FROM employees e
JOIN jobs j ON e.job_id = j.job_id
LEFT JOIN departments d ON e.department_id = d.department_id
LEFT JOIN locations l ON d.location_id = l.location_id
LEFT JOIN countries c ON l.country_id = c.country_id
LEFT JOIN regions r ON c.region_id = r.region_id
LEFT JOIN employees m ON e.manager_id = m.employee_id
WHERE e.salary BETWEEN 5000 AND 15000
ORDER BY e.salary DESC, e.last_name;

-- Result shows complete employee information with geographic details
```

### Non-Equijoins
```sql
-- Join without equality operator
-- Match employees to salary grades
SELECT 
    e.first_name,
    e.salary,
    jg.grade_level
FROM employees e
JOIN job_grades jg ON e.salary BETWEEN jg.lowest_sal AND jg.highest_sal;

-- Result:
FIRST_NAME | SALARY | GRADE_LEVEL
Steven     | 24000  | E
Neena      | 17000  | E
Alexander  | 9000   | C
Diana      | 4200   | A

-- Another example: Compare salaries
SELECT 
    e1.first_name AS employee1,
    e1.salary AS salary1,
    e2.first_name AS employee2,
    e2.salary AS salary2
FROM employees e1
JOIN employees e2 ON e1.salary < e2.salary
WHERE e1.department_id = e2.department_id
  AND e1.employee_id < e2.employee_id  -- Avoid duplicates
ORDER BY e1.first_name;
```

***

## Module 9: Using Subqueries to Solve Queries

### Single-Row Subqueries
```sql
-- Find employees who earn more than the average salary
SELECT first_name
## Module 9: Using Subqueries to Solve Queries (Continued)

### Single-Row Subqueries (Continued)
```sql
-- Find employees who earn more than the average salary
SELECT first_name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Result:
FIRST_NAME | SALARY
Steven     | 24000
Neena      | 17000
Lex        | 17000
Nancy      | 12008
Daniel     | 11000

-- Find employee with same job as employee 141
SELECT first_name, job_id
FROM employees
WHERE job_id = (SELECT job_id 
                FROM employees 
                WHERE employee_id = 141);

-- Find employees in same department as 'Steven'
SELECT first_name, last_name, department_id
FROM employees
WHERE department_id = (SELECT department_id
                       FROM employees
                       WHERE first_name = 'Steven');

-- Result:
FIRST_NAME | LAST_NAME | DEPARTMENT_ID
Steven     | King      | 90
Neena      | Kochhar   | 90
Lex        | De Haan   | 90

-- Subquery with aggregate function
SELECT first_name, hire_date
FROM employees
WHERE hire_date = (SELECT MIN(hire_date) FROM employees);

-- Result shows earliest hired employee

-- Using GROUP BY in subquery
SELECT first_name, salary, department_id
FROM employees
WHERE salary > (SELECT AVG(salary) 
                FROM employees 
                WHERE department_id = 50)
ORDER BY salary DESC;
```

### Multiple-Row Subqueries with IN
```sql
-- Find employees who work in IT or Sales departments
SELECT first_name, last_name, department_id
FROM employees
WHERE department_id IN (SELECT department_id
                        FROM departments
                        WHERE department_name IN ('IT', 'Sales'));

-- Result:
FIRST_NAME | LAST_NAME | DEPARTMENT_ID
Alexander  | Hunold    | 60
Bruce      | Ernst     | 60
David      | Austin    | 60
John       | Russell   | 80
Karen      | Partners  | 80

-- Find employees with job titles that exist in department 90
SELECT first_name, job_id, department_id
FROM employees
WHERE job_id IN (SELECT job_id
                 FROM employees
                 WHERE department_id = 90);

-- Find employees in departments located in 'Seattle'
SELECT first_name, department_id
FROM employees
WHERE department_id IN (SELECT department_id
                        FROM departments
                        WHERE location_id IN (SELECT location_id
                                            FROM locations
                                            WHERE city = 'Seattle'));
-- Note: This is a nested subquery (subquery within subquery)
```

### Multiple-Row Subqueries with ANY
```sql
-- ANY means "any one of the values"
-- Find employees earning less than ANY manager in dept 90
SELECT first_name, salary, job_id
FROM employees
WHERE salary < ANY (SELECT salary
                    FROM employees
                    WHERE department_id = 90)
  AND job_id != 'AD_PRES';

-- Result shows employees earning less than at least one person in dept 90

-- Using ANY with comparison operators
SELECT first_name, salary
FROM employees
WHERE salary > ANY (SELECT salary
                    FROM employees
                    WHERE job_id = 'IT_PROG');

-- Result: Employees earning more than ANY IT programmer
-- (more than the lowest IT_PROG salary)

-- Equivalent to:
SELECT first_name, salary
FROM employees
WHERE salary > (SELECT MIN(salary)
                FROM employees
                WHERE job_id = 'IT_PROG');
```

### Multiple-Row Subqueries with ALL
```sql
-- ALL means "all of the values"
-- Find employees earning more than ALL IT programmers
SELECT first_name, salary, job_id
FROM employees
WHERE salary > ALL (SELECT salary
                    FROM employees
                    WHERE job_id = 'IT_PROG');

-- Result:
FIRST_NAME | SALARY | JOB_ID
Steven     | 24000  | AD_PRES
Neena      | 17000  | AD_VP
Lex        | 17000  | AD_VP
Nancy      | 12008  | FI_MGR

-- Equivalent to using MAX:
SELECT first_name, salary
FROM employees
WHERE salary > (SELECT MAX(salary)
                FROM employees
                WHERE job_id = 'IT_PROG');

-- Using ALL with less than
SELECT first_name, salary
FROM employees
WHERE salary < ALL (SELECT salary
                    FROM employees
                    WHERE department_id = 90);

-- Result shows employees earning less than ALL dept 90 employees
-- (less than the lowest dept 90 salary)
```

### NOT IN with Subqueries
```sql
-- Find employees NOT in specific departments
SELECT first_name, department_id
FROM employees
WHERE department_id NOT IN (SELECT department_id
                            FROM departments
                            WHERE department_name IN ('IT', 'Sales'));

-- Result shows employees not in IT or Sales

-- Find employees whose job is not held by anyone in dept 50
SELECT first_name, job_id
FROM employees
WHERE job_id NOT IN (SELECT job_id
                     FROM employees
                     WHERE department_id = 50);

-- WARNING: NOT IN with NULL values can cause issues
SELECT first_name
FROM employees
WHERE department_id NOT IN (SELECT department_id
                            FROM employees
                            WHERE manager_id = 100);
-- If subquery returns NULL, entire query returns no rows!

-- Safe version using IS NOT NULL:
SELECT first_name
FROM employees
WHERE department_id NOT IN (SELECT department_id
                            FROM employees
                            WHERE manager_id = 100
                              AND department_id IS NOT NULL);
```

### Subqueries in FROM Clause (Inline Views)
```sql
-- Use subquery as a temporary table
SELECT e.first_name, e.salary, dept_avg.avg_salary
FROM employees e,
     (SELECT department_id, AVG(salary) AS avg_salary
      FROM employees
      GROUP BY department_id) dept_avg
WHERE e.department_id = dept_avg.department_id
  AND e.salary > dept_avg.avg_salary;

-- Result shows employees earning above their department average

-- Complex inline view example
SELECT dept_stats.department_name,
       dept_stats.emp_count,
       dept_stats.total_salary
FROM (SELECT d.department_name,
             COUNT(e.employee_id) AS emp_count,
             SUM(e.salary) AS total_salary
      FROM departments d
      JOIN employees e ON d.department_id = e.department_id
      GROUP BY d.department_name) dept_stats
WHERE dept_stats.emp_count > 3;

-- Result:
DEPARTMENT_NAME | EMP_COUNT | TOTAL_SALARY
Shipping        | 5         | 17500
Sales           | 6         | 60200
```

### Subqueries in HAVING Clause
```sql
-- Find departments with minimum salary greater than dept 50's minimum
SELECT department_id, MIN(salary) AS min_salary
FROM employees
GROUP BY department_id
HAVING MIN(salary) > (SELECT MIN(salary)
                      FROM employees
                      WHERE department_id = 50);

-- Result:
DEPARTMENT_ID | MIN_SALARY
90           | 17000
60           | 4200
80           | 6100

-- Find departments with more employees than average
SELECT department_id, COUNT(*) AS emp_count
FROM employees
GROUP BY department_id
HAVING COUNT(*) > (SELECT AVG(COUNT(*))
                   FROM employees
                   GROUP BY department_id);
```

### Multiple Column Subqueries
```sql
-- Compare multiple columns simultaneously (pairwise)
SELECT first_name, department_id, salary
FROM employees
WHERE (department_id, salary) IN 
      (SELECT department_id, MAX(salary)
       FROM employees
       GROUP BY department_id);

-- Result shows highest paid employee in each department:
FIRST_NAME | DEPARTMENT_ID | SALARY
Steven     | 90           | 24000
Nancy      | 100          | 12008
Alexander  | 60           | 9000

-- Non-pairwise comparison
SELECT first_name, department_id, salary
FROM employees
WHERE department_id IN (SELECT department_id
                        FROM employees
                        WHERE first_name = 'Steven')
  AND salary IN (SELECT MAX(salary)
                 FROM employees
                 GROUP BY department_id);
```

### Correlated Subqueries
```sql
-- Inner query references outer query
-- Find employees earning above their department average
SELECT e.first_name, e.salary, e.department_id
FROM employees e
WHERE e.salary > (SELECT AVG(salary)
                  FROM employees
                  WHERE department_id = e.department_id);

-- Result:
FIRST_NAME | SALARY | DEPARTMENT_ID
Steven     | 24000  | 90
Neena      | 17000  | 90
Lex        | 17000  | 90
Alexander  | 9000   | 60

-- Execution: For EACH row in outer query, inner query runs

-- Find employees who earn more than average in their job
SELECT e.first_name, e.job_id, e.salary
FROM employees e
WHERE e.salary > (SELECT AVG(salary)
                  FROM employees
                  WHERE job_id = e.job_id);

-- Show department name with employee count
SELECT d.department_name,
       (SELECT COUNT(*)
        FROM employees e
        WHERE e.department_id = d.department_id) AS emp_count
FROM departments d;

-- Result:
DEPARTMENT_NAME | EMP_COUNT
Administration  | 1
Marketing       | 2
Purchasing      | 6
IT              | 5
```

### EXISTS Operator
```sql
-- EXISTS returns TRUE if subquery returns at least one row
-- Find employees who are managers
SELECT e.first_name, e.last_name
FROM employees e
WHERE EXISTS (SELECT 1
              FROM employees
              WHERE manager_id = e.employee_id);

-- Result shows only employees who manage others

-- Find departments that have employees
SELECT d.department_name
FROM departments d
WHERE EXISTS (SELECT 1
              FROM employees e
              WHERE e.department_id = d.department_id);

-- Result:
DEPARTMENT_NAME
Administration
Marketing
Purchasing
IT
Sales
Executive

-- EXISTS is efficient - stops at first match
-- Equivalent to IN but often faster for large datasets
```

### NOT EXISTS Operator
```sql
-- Find employees who are NOT managers
SELECT e.first_name, e.last_name
FROM employees e
WHERE NOT EXISTS (SELECT 1
                  FROM employees
                  WHERE manager_id = e.employee_id);

-- Result shows employees with no direct reports

-- Find departments without employees
SELECT d.department_id, d.department_name
FROM departments d
WHERE NOT EXISTS (SELECT 1
                  FROM employees e
                  WHERE e.department_id = d.department_id);

-- Result:
DEPARTMENT_ID | DEPARTMENT_NAME
120          | Treasury
130          | Corporate Tax
140          | Control And Credit
150          | Shareholder Services

-- Find employees not in any job history
SELECT e.first_name, e.employee_id
FROM employees e
WHERE NOT EXISTS (SELECT 1
                  FROM job_history jh
                  WHERE jh.employee_id = e.employee_id);
```

***

## Module 10: Using Set Operators

### UNION - Combine and Remove Duplicates
```sql
-- Combine results from two queries, eliminate duplicates
SELECT employee_id, job_id
FROM employees
WHERE department_id = 60
UNION
SELECT employee_id, job_id
FROM job_history
WHERE department_id = 60;

-- Result: Unique combinations from both tables

-- Must have same number of columns
-- Data types must be compatible

-- Example: Current and past IT programmers
SELECT first_name, last_name, 'Current Employee' AS status
FROM employees
WHERE job_id = 'IT_PROG'
UNION
SELECT first_name, last_name, 'Past Employee' AS status
FROM employees e
JOIN job_history jh ON e.employee_id = jh.employee_id
WHERE jh.job_id = 'IT_PROG';

-- Result shows all IT programmers (current and former)
```

### UNION ALL - Combine and Keep Duplicates
```sql
-- Faster than UNION (no duplicate removal)
SELECT employee_id, job_id
FROM employees
WHERE department_id = 60
UNION ALL
SELECT employee_id, job_id
FROM job_history
WHERE department_id = 60;

-- Result includes duplicates if any

-- Practical: Combine salary data from multiple sources
SELECT 'Employee' AS source, first_name, salary
FROM employees
WHERE salary > 10000
UNION ALL
SELECT 'Bonus' AS source, first_name, commission_pct * 1000
FROM employees
WHERE commission_pct IS NOT NULL;

-- Use case: Audit trail or historical reporting
SELECT employee_id, salary, hire_date AS change_date, 'HIRE' AS event_type
FROM employees
UNION ALL
SELECT employee_id, salary, sysdate, 'CURRENT' AS event_type
FROM employees;
```

### INTERSECT - Common Rows
```sql
-- Returns only rows that appear in BOTH queries
SELECT employee_id
FROM employees
WHERE department_id = 60
INTERSECT
SELECT employee_id
FROM job_history
WHERE job_id = 'IT_PROG';

-- Result: Employees currently in dept 60 who were IT_PROG before

-- Find employees who worked in both Sales and IT
SELECT employee_id
FROM job_history
WHERE department_id = 80  -- Sales
INTERSECT
SELECT employee_id
FROM job_history
WHERE department_id = 60;  -- IT

-- Find common job titles between two departments
SELECT job_id
FROM employees
WHERE department_id = 50
INTERSECT
SELECT job_id
FROM employees
WHERE department_id = 80;

-- Result shows jobs that exist in both departments
```

### MINUS (or EXCEPT in some databases)
```sql
-- Returns rows from first query NOT in second query
SELECT employee_id
FROM employees
MINUS
SELECT employee_id
FROM job_history;

-- Result: Employees who never changed jobs

-- Find departments without employees
SELECT department_id
FROM departments
MINUS
SELECT department_id
FROM employees;

-- Result:
DEPARTMENT_ID
120
130
140
150

-- Find job titles not currently assigned
SELECT job_id
FROM jobs
MINUS
SELECT job_id
FROM employees;

-- Find employees who never had commission
SELECT employee_id, first_name
FROM employees
MINUS
SELECT employee_id, first_name
FROM employees
WHERE commission_pct IS NOT NULL;
```

### Set Operator Rules and ORDER BY
```sql
-- Number of columns must match
-- Data types must be compatible
-- Column names from first query are used
-- ORDER BY only at the end (not in individual queries)

-- Valid:
SELECT employee_id, first_name, salary
FROM employees
WHERE department_id = 60
UNION
SELECT employee_id, first_name, salary
FROM employees
WHERE department_id = 90
ORDER BY salary DESC;

-- ORDER BY with column position
SELECT employee_id, first_name
FROM employees
WHERE department_id = 60
UNION
SELECT employee_id, first_name
FROM employees
WHERE department_id = 90
ORDER BY 2;  -- Orders by second column (first_name)

-- ORDER BY with alias from first query
SELECT employee_id AS emp_id, first_name AS name
FROM employees
WHERE department_id = 60
UNION
SELECT employee_id, first_name
FROM employees
WHERE department_id = 90
ORDER BY name;  -- Uses alias from first SELECT
```

### Complex Set Operations
```sql
-- Combine multiple set operators
-- Use parentheses to control precedence
(SELECT employee_id FROM employees WHERE department_id = 60
 UNION
 SELECT employee_id FROM employees WHERE department_id = 90)
MINUS
SELECT employee_id FROM job_history;

-- Result: Employees in dept 60 or 90 who never changed jobs

## Module 10: Using Set Operators (Continued)

### Complex Set Operations (Continued)
```sql
-- Create comprehensive employee report
SELECT employee_id, first_name, job_id, 'Active' AS status, salary
FROM employees
WHERE hire_date > TO_DATE('01-JAN-2005', 'DD-MON-YYYY')
UNION ALL
SELECT employee_id, first_name, job_id, 'Senior' AS status, salary
FROM employees
WHERE MONTHS_BETWEEN(SYSDATE, hire_date) > 240
UNION ALL
SELECT employee_id, first_name, job_id, 'High Earner' AS status, salary
FROM employees
WHERE salary > 15000
ORDER BY employee_id, status;

-- Combines different employee categories with labels
```

***

## Module 11: Managing Tables Using DML Statements

### INSERT - Single Row
```sql
-- Insert with all columns specified
INSERT INTO departments (department_id, department_name, manager_id, location_id)
VALUES (280, 'Data Science', 103, 1700);

-- Result: 1 row inserted

-- Insert without column list (must match ALL columns in order)
INSERT INTO departments
VALUES (290, 'Cybersecurity', 104, 1700);

-- Insert with some columns (others default to NULL)
INSERT INTO departments (department_id, department_name)
VALUES (300, 'AI Research');

-- Result: manager_id and location_id will be NULL

-- Insert with explicit NULL
INSERT INTO employees (employee_id, first_name, last_name, email, 
                       hire_date, job_id, salary, commission_pct)
VALUES (207, 'John', 'Smith', 'JSMITH',
        SYSDATE, 'IT_PROG', 8000, NULL);

-- Insert with functions
INSERT INTO employees (employee_id, first_name, last_name, email,
                       hire_date, job_id, salary)
VALUES (208, 'Jane', 'Doe', UPPER('jdoe'),
        TO_DATE('15-NOV-2025', 'DD-MON-YYYY'), 'IT_PROG', 8500);

-- Insert with expressions
INSERT INTO employees (employee_id, first_name, last_name, email,
                       hire_date, job_id, salary)
VALUES (209, 'Bob', 'Wilson', 'BWILSON',
        SYSDATE, 'SA_REP', 6000 * 1.1);  -- Calculated salary
```

### INSERT - Multiple Rows with Subquery
```sql
-- Copy data from another table
INSERT INTO sales_reps (employee_id, first_name, last_name, salary)
SELECT employee_id, first_name, last_name, salary
FROM employees
WHERE job_id = 'SA_REP';

-- Result: All sales reps copied to new table

-- Insert with calculations
INSERT INTO employee_bonuses (employee_id, bonus_amount, bonus_date)
SELECT employee_id, salary * 0.10, SYSDATE
FROM employees
WHERE commission_pct IS NOT NULL;

-- Insert with JOIN
INSERT INTO dept_summary (dept_id, dept_name, emp_count, total_salary)
SELECT d.department_id, d.department_name, 
       COUNT(e.employee_id), SUM(e.salary)
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_id, d.department_name;

-- No VALUES clause when using subquery
```

### UPDATE - Modify Existing Data
```sql
-- Update single column
UPDATE employees
SET salary = 10000
WHERE employee_id = 107;

-- Result: 1 row updated

-- Update multiple columns
UPDATE employees
SET salary = 11000,
    commission_pct = 0.15
WHERE employee_id = 107;

-- Update with expression
UPDATE employees
SET salary = salary * 1.10  -- 10% raise
WHERE department_id = 60;

-- Result: All IT employees get 10% raise

-- Update with function
UPDATE employees
SET hire_date = ADD_MONTHS(hire_date, -1)
WHERE employee_id = 208;

-- Update multiple columns with calculation
UPDATE employees
SET salary = salary * 1.05,
    commission_pct = commission_pct + 0.02
WHERE job_id = 'SA_REP'
  AND salary < 10000;

-- Update with CASE
UPDATE employees
SET salary = CASE 
    WHEN department_id = 60 THEN salary * 1.15
    WHEN department_id = 90 THEN salary * 1.10
    WHEN department_id = 50 THEN salary * 1.08
    ELSE salary * 1.05
END
WHERE salary < 15000;
```

### UPDATE with Subquery
```sql
-- Update based on another table
UPDATE employees
SET department_id = (SELECT department_id
                     FROM departments
                     WHERE department_name = 'IT')
WHERE job_id = 'IT_PROG'
  AND department_id IS NULL;

-- Update salary to department average
UPDATE employees e
SET salary = (SELECT AVG(salary)
              FROM employees
              WHERE department_id = e.department_id)
WHERE employee_id = 999;

-- Update multiple columns with subquery
UPDATE employees
SET (salary, commission_pct) = 
    (SELECT MAX(salary), MAX(commission_pct)
     FROM employees
     WHERE department_id = 80)
WHERE employee_id = 210;

-- Correlated update
UPDATE employees e
SET salary = (SELECT AVG(salary) * 1.1
              FROM employees
              WHERE job_id = e.job_id)
WHERE department_id = 50;
```

### DELETE - Remove Rows
```sql
-- Delete specific row
DELETE FROM departments
WHERE department_id = 280;

-- Result: 1 row deleted

-- Delete multiple rows
DELETE FROM employees
WHERE department_id = 50
  AND salary < 3000;

-- Delete all rows (keeps table structure)
DELETE FROM temp_employees;

-- Result: All rows deleted

-- Delete with subquery
DELETE FROM employees
WHERE department_id IN (SELECT department_id
                        FROM departments
                        WHERE location_id = 1700);

-- Delete based on correlated subquery
DELETE FROM employees e
WHERE salary < (SELECT AVG(salary)
                FROM employees
                WHERE department_id = e.department_id);
-- Deletes employees earning below their department average

-- Delete with NOT EXISTS
DELETE FROM departments d
WHERE NOT EXISTS (SELECT 1
                  FROM employees e
                  WHERE e.department_id = d.department_id);
-- Deletes departments with no employees
```

### MERGE Statement
```sql
-- Insert or update based on condition
MERGE INTO employees_copy ec
USING employees e
ON (ec.employee_id = e.employee_id)
WHEN MATCHED THEN
    UPDATE SET
        ec.first_name = e.first_name,
        ec.salary = e.salary,
        ec.department_id = e.department_id
WHEN NOT MATCHED THEN
    INSERT (employee_id, first_name, last_name, email, 
            hire_date, job_id, salary, department_id)
    VALUES (e.employee_id, e.first_name, e.last_name, e.email,
            e.hire_date, e.job_id, e.salary, e.department_id);

-- Result: Updates existing rows, inserts new rows

-- MERGE with WHERE clause
MERGE INTO bonuses b
USING (SELECT employee_id, salary * 0.1 AS bonus_amt
       FROM employees
       WHERE department_id = 80) e
ON (b.employee_id = e.employee_id)
WHEN MATCHED THEN
    UPDATE SET b.bonus_amount = e.bonus_amt
WHEN NOT MATCHED THEN
    INSERT (employee_id, bonus_amount)
    VALUES (e.employee_id, e.bonus_amt);

-- MERGE with DELETE clause
MERGE INTO employee_archive ea
USING employees e
ON (ea.employee_id = e.employee_id)
WHEN MATCHED THEN
    UPDATE SET ea.status = 'UPDATED'
    DELETE WHERE ea.salary < 5000  -- Delete after update if condition met
WHEN NOT MATCHED THEN
    INSERT VALUES (e.employee_id, e.first_name, 'NEW');
```

### Transaction Control - COMMIT
```sql
-- Save changes permanently
INSERT INTO departments (department_id, department_name)
VALUES (310, 'Research');

COMMIT;
-- Changes are now permanent and visible to other users

-- Multiple statements before commit
UPDATE employees SET salary = salary * 1.05 WHERE department_id = 60;
INSERT INTO salary_history 
SELECT employee_id, salary, SYSDATE FROM employees WHERE department_id = 60;
COMMIT;
-- Both statements saved together
```

### Transaction Control - ROLLBACK
```sql
-- Undo changes since last COMMIT
UPDATE employees
SET salary = 99999
WHERE department_id = 90;

ROLLBACK;
-- Changes are undone, data restored to last COMMIT

-- Example transaction
INSERT INTO departments VALUES (320, 'Test Dept', NULL, NULL);
DELETE FROM employees WHERE employee_id = 999;
-- Oops, made a mistake!
ROLLBACK;
-- Both INSERT and DELETE are undone
```

### Transaction Control - SAVEPOINT
```sql
-- Create intermediate rollback points
UPDATE employees SET salary = salary * 1.10 WHERE department_id = 60;
SAVEPOINT after_it_raises;

UPDATE employees SET salary = salary * 1.05 WHERE department_id = 90;
SAVEPOINT after_exec_raises;

UPDATE employees SET salary = salary * 1.08 WHERE department_id = 50;
-- Oops, wrong amount for shipping!

ROLLBACK TO SAVEPOINT after_exec_raises;
-- Only the last UPDATE is undone

UPDATE employees SET salary = salary * 1.07 WHERE department_id = 50;
-- Correct it

COMMIT;
-- All changes except the rolled-back one are saved

-- Multiple savepoints example
SAVEPOINT sp1;
INSERT INTO test_table VALUES (1, 'First');
SAVEPOINT sp2;
INSERT INTO test_table VALUES (2, 'Second');
SAVEPOINT sp3;
INSERT INTO test_table VALUES (3, 'Third');

ROLLBACK TO sp2;  -- Removes only third insert
COMMIT;  -- Saves first and second inserts
```

### Read Consistency
```sql
-- Oracle provides read consistency
-- User A:
UPDATE employees SET salary = 10000 WHERE employee_id = 107;
-- Not committed yet

-- User B (in different session):
SELECT salary FROM employees WHERE employee_id = 107;
-- Sees OLD value (before User A's change)

-- User A:
COMMIT;

-- User B:
SELECT salary FROM employees WHERE employee_id = 107;
-- Now sees NEW value (10000)

-- This ensures users always see consistent data
```

***

## Module 12: Introduction to Data Definition Language

### CREATE TABLE - Basic Syntax
```sql
-- Create simple table
CREATE TABLE departments_new (
    department_id   NUMBER(4),
    department_name VARCHAR2(30),
    manager_id      NUMBER(6),
    location_id     NUMBER(4)
);

-- Table created successfully

-- Create table with constraints
CREATE TABLE employees_new (
    employee_id    NUMBER(6) PRIMARY KEY,
    first_name     VARCHAR2(20) NOT NULL,
    last_name      VARCHAR2(25) NOT NULL,
    email          VARCHAR2(25) UNIQUE NOT NULL,
    hire_date      DATE DEFAULT SYSDATE NOT NULL,
    job_id         VARCHAR2(10) NOT NULL,
    salary         NUMBER(8,2) CHECK (salary > 0),
    department_id  NUMBER(4)
);

-- With all column-level constraints
```

### Common Data Types
```sql
-- Character data types
CREATE TABLE string_examples (
    fixed_char      CHAR(10),        -- Fixed length, padded with spaces
    variable_char   VARCHAR2(100),   -- Variable length, up to 100
    large_text      CLOB             -- Character Large Object, up to 4GB
);

-- Examples:
-- CHAR(10) with 'ABC' stores 'ABC       ' (padded)
-- VARCHAR2(10) with 'ABC' stores 'ABC' (no padding)

-- Numeric data types
CREATE TABLE number_examples (
    integer_col     NUMBER(5),       -- Up to 5 digits
    decimal_col     NUMBER(8,2),     -- 8 total digits, 2 after decimal
    float_col       FLOAT,           -- Floating point
    precise_num     NUMBER           -- Unlimited precision
);

-- Examples:
-- NUMBER(8,2) can store 999999.99
-- NUMBER(5) can store 99999

-- Date and time data types
CREATE TABLE date_examples (
    simple_date     DATE,            -- Date and time (second precision)
    timestamp_col   TIMESTAMP,       -- Date and time (fractional seconds)
    timestamp_tz    TIMESTAMP WITH TIME ZONE,  -- With time zone
    timestamp_ltz   TIMESTAMP WITH LOCAL TIME ZONE,  -- Adjusted to session TZ
    interval_ym     INTERVAL YEAR TO MONTH,  -- Period of years/months
    interval_ds     INTERVAL DAY TO SECOND   -- Period of days/seconds
);

-- Binary data types
CREATE TABLE binary_examples (
    small_binary    RAW(2000),       -- Small binary, up to 2000 bytes
    large_binary    BLOB,            -- Binary Large Object, up to 4GB
    binary_file     BFILE            -- Reference to external file
);

-- Examples with sample data:
INSERT INTO date_examples (simple_date, timestamp_col)
VALUES (SYSDATE, SYSTIMESTAMP);

-- Result:
-- simple_date: 21-NOV-25
-- timestamp_col: 21-NOV-25 11.00.00.000000 AM
```

### CREATE TABLE with Subquery
```sql
-- Create table and copy data
CREATE TABLE it_employees AS
SELECT employee_id, first_name, last_name, salary, hire_date
FROM employees
WHERE department_id = 60;

-- Table created with structure and data

-- Create table with modified structure
CREATE TABLE high_earners AS
SELECT employee_id AS emp_id,
       first_name || ' ' || last_name AS full_name,
       salary * 12 AS annual_salary
FROM employees
WHERE salary > 10000;

-- Create empty table (copy structure only)
CREATE TABLE employees_backup AS
SELECT *
FROM employees
WHERE 1 = 2;  -- Condition always false, no rows copied

-- Result: Table created with same structure, no data
```

### PRIMARY KEY Constraint
```sql
-- Column-level primary key
CREATE TABLE departments_pk (
    department_id   NUMBER(4) PRIMARY KEY,
    department_name VARCHAR2(30)
);

-- Table-level primary key
CREATE TABLE departments_pk2 (
    department_id   NUMBER(4),
    department_name VARCHAR2(30),
    CONSTRAINT dept_pk PRIMARY KEY (department_id)
);

-- Composite primary key (multiple columns)
CREATE TABLE job_history_new (
    employee_id   NUMBER(6),
    start_date    DATE,
    end_date      DATE,
    job_id        VARCHAR2(10),
    department_id NUMBER(4),
    CONSTRAINT jh_pk PRIMARY KEY (employee_id, start_date)
);

-- Primary key ensures:
-- 1. NOT NULL
-- 2. UNIQUE
-- 3. Only one per table
```

### FOREIGN KEY Constraint
```sql
-- Column-level foreign key
CREATE TABLE employees_fk (
    employee_id   NUMBER(6) PRIMARY KEY,
    first_name    VARCHAR2(20),
    department_id NUMBER(4) REFERENCES departments(department_id)
);

-- Table-level foreign key with name
CREATE TABLE employees_fk2 (
    employee_id   NUMBER(6) PRIMARY KEY,
    first_name    VARCHAR2(20),
    department_id NUMBER(4),
    CONSTRAINT emp_dept_fk FOREIGN KEY (department_id)

## Module 12: Introduction to Data Definition Language (Continued)

### FOREIGN KEY Constraint (Continued)
```sql
-- Table-level foreign key with name
CREATE TABLE employees_fk2 (
    employee_id   NUMBER(6) PRIMARY KEY,
    first_name    VARCHAR2(20),
    department_id NUMBER(4),
    CONSTRAINT emp_dept_fk FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
);

-- Foreign key with ON DELETE CASCADE
CREATE TABLE employees_cascade (
    employee_id   NUMBER(6) PRIMARY KEY,
    first_name    VARCHAR2(20),
    department_id NUMBER(4),
    CONSTRAINT emp_dept_fk_cascade FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
        ON DELETE CASCADE
);
-- When parent department is deleted, child employees are also deleted

-- Foreign key with ON DELETE SET NULL
CREATE TABLE employees_setnull (
    employee_id   NUMBER(6) PRIMARY KEY,
    first_name    VARCHAR2(20),
    department_id NUMBER(4),
    CONSTRAINT emp_dept_fk_null FOREIGN KEY (department_id)
        REFERENCES departments(department_id)
        ON DELETE SET NULL
);
-- When parent department is deleted, department_id set to NULL

-- Example behavior:
DELETE FROM departments WHERE department_id = 60;
-- With CASCADE: All employees in dept 60 are deleted
-- With SET NULL: Employees remain but department_id becomes NULL
-- Without option: Error - cannot delete parent with children
```

### UNIQUE Constraint
```sql
-- Column-level unique
CREATE TABLE employees_unique (
    employee_id NUMBER(6) PRIMARY KEY,
    email       VARCHAR2(25) UNIQUE,
    phone       VARCHAR2(20)
);

-- Table-level unique with name
CREATE TABLE employees_unique2 (
    employee_id NUMBER(6) PRIMARY KEY,
    email       VARCHAR2(25),
    ssn         VARCHAR2(11),
    CONSTRAINT emp_email_uk UNIQUE (email),
    CONSTRAINT emp_ssn_uk UNIQUE (ssn)
);

-- Composite unique constraint
CREATE TABLE project_assignments (
    employee_id NUMBER(6),
    project_id  NUMBER(6),
    role        VARCHAR2(20),
    CONSTRAINT proj_emp_uk UNIQUE (employee_id, project_id)
);
-- Same employee can't be assigned to same project twice

-- UNIQUE allows NULL (unlike PRIMARY KEY)
INSERT INTO employees_unique VALUES (1, NULL, '555-1234');
INSERT INTO employees_unique VALUES (2, NULL, '555-5678');
-- Both succeed - multiple NULLs allowed
```

### CHECK Constraint
```sql
-- Column-level check
CREATE TABLE employees_check (
    employee_id NUMBER(6) PRIMARY KEY,
    first_name  VARCHAR2(20),
    salary      NUMBER(8,2) CHECK (salary > 0)
);

-- Table-level check with name
CREATE TABLE employees_check2 (
    employee_id NUMBER(6) PRIMARY KEY,
    first_name  VARCHAR2(20),
    salary      NUMBER(8,2),
    hire_date   DATE,
    CONSTRAINT emp_salary_ck CHECK (salary > 0),
    CONSTRAINT emp_hire_ck CHECK (hire_date <= SYSDATE)
);

-- Multiple conditions in check
CREATE TABLE employees_advanced_check (
    employee_id   NUMBER(6) PRIMARY KEY,
    salary        NUMBER(8,2),
    commission_pct NUMBER(2,2),
    job_id        VARCHAR2(10),
    CONSTRAINT emp_sal_ck CHECK (salary BETWEEN 2000 AND 100000),
    CONSTRAINT emp_comm_ck CHECK (commission_pct BETWEEN 0 AND 0.5),
    CONSTRAINT emp_job_ck CHECK (job_id IN ('SA_REP', 'SA_MAN', 'IT_PROG'))
);

-- Examples:
INSERT INTO employees_advanced_check VALUES (1, 50000, 0.2, 'SA_REP');
-- Success

INSERT INTO employees_advanced_check VALUES (2, 150000, 0.2, 'SA_REP');
-- Error: salary violates check constraint (> 100000)

INSERT INTO employees_advanced_check VALUES (3, 50000, 0.8, 'SA_REP');
-- Error: commission violates check constraint (> 0.5)
```

### NOT NULL Constraint
```sql
-- Column-level NOT NULL
CREATE TABLE employees_notnull (
    employee_id NUMBER(6) PRIMARY KEY,
    first_name  VARCHAR2(20) NOT NULL,
    last_name   VARCHAR2(25) NOT NULL,
    email       VARCHAR2(25) NOT NULL,
    phone       VARCHAR2(20)  -- Can be NULL
);

-- NOT NULL with constraint name
CREATE TABLE employees_notnull2 (
    employee_id NUMBER(6) PRIMARY KEY,
    first_name  VARCHAR2(20) CONSTRAINT emp_fname_nn NOT NULL,
    last_name   VARCHAR2(25) CONSTRAINT emp_lname_nn NOT NULL
);

-- Examples:
INSERT INTO employees_notnull VALUES (1, 'John', 'Smith', 'JSMITH', NULL);
-- Success - phone can be NULL

INSERT INTO employees_notnull VALUES (2, NULL, 'Doe', 'JDOE', '555-1234');
-- Error: first_name cannot be NULL
```

### DEFAULT Values
```sql
-- Column with default value
CREATE TABLE employees_defaults (
    employee_id NUMBER(6) PRIMARY KEY,
    first_name  VARCHAR2(20) NOT NULL,
    hire_date   DATE DEFAULT SYSDATE,
    status      VARCHAR2(10) DEFAULT 'ACTIVE',
    salary      NUMBER(8,2) DEFAULT 5000
);

-- Insert without specifying default columns
INSERT INTO employees_defaults (employee_id, first_name)
VALUES (1, 'John');
-- hire_date = today, status = 'ACTIVE', salary = 5000

-- Insert explicitly overriding defaults
INSERT INTO employees_defaults (employee_id, first_name, hire_date, status)
VALUES (2, 'Jane', TO_DATE('01-JAN-2023', 'DD-MON-YYYY'), 'INACTIVE');

-- Default with expressions
CREATE TABLE orders (
    order_id     NUMBER PRIMARY KEY,
    order_date   DATE DEFAULT SYSDATE,
    ship_date    DATE DEFAULT SYSDATE + 7,  -- 7 days from now
    order_total  NUMBER DEFAULT 0
);
```

### ALTER TABLE - Add Column
```sql
-- Add single column
ALTER TABLE employees_new
ADD (middle_name VARCHAR2(20));

-- Table modified, existing rows have NULL in new column

-- Add multiple columns
ALTER TABLE employees_new
ADD (birth_date DATE,
     gender VARCHAR2(1),
     marital_status VARCHAR2(10));

-- Add column with default
ALTER TABLE employees_new
ADD (bonus NUMBER(8,2) DEFAULT 0);
-- Existing rows get 0 for bonus

-- Add column with constraint
ALTER TABLE employees_new
ADD (social_security VARCHAR2(11) UNIQUE);
```

### ALTER TABLE - Modify Column
```sql
-- Increase column size
ALTER TABLE employees_new
MODIFY (first_name VARCHAR2(50));

-- Change data type (only if column is empty or compatible)
ALTER TABLE employees_new
MODIFY (middle_name CHAR(20));

-- Add NOT NULL constraint
ALTER TABLE employees_new
MODIFY (email NOT NULL);

-- Add default value
ALTER TABLE employees_new
MODIFY (status DEFAULT 'ACTIVE');

-- Multiple modifications
ALTER TABLE employees_new
MODIFY (first_name VARCHAR2(50),
        last_name VARCHAR2(50),
        salary NUMBER(10,2));

-- Cannot decrease size if data exists:
ALTER TABLE employees_new
MODIFY (first_name VARCHAR2(5));
-- Error if any first_name > 5 characters exists
```

### ALTER TABLE - Drop Column
```sql
-- Drop single column
ALTER TABLE employees_new
DROP COLUMN middle_name;

-- Drop multiple columns
ALTER TABLE employees_new
DROP (birth_date, gender, marital_status);

-- Drop column with dependencies (CASCADE)
ALTER TABLE employees_new
DROP COLUMN department_id CASCADE CONSTRAINTS;
-- Also drops foreign key constraints referencing this column

-- Mark column unused (faster, delayed physical removal)
ALTER TABLE employees_new
SET UNUSED (bonus);

-- Later, physically remove unused columns
ALTER TABLE employees_new
DROP UNUSED COLUMNS;
```

### ALTER TABLE - Add Constraints
```sql
-- Add primary key
ALTER TABLE employees_new
ADD CONSTRAINT emp_new_pk PRIMARY KEY (employee_id);

-- Add foreign key
ALTER TABLE employees_new
ADD CONSTRAINT emp_dept_fk 
    FOREIGN KEY (department_id)
    REFERENCES departments(department_id);

-- Add unique constraint
ALTER TABLE employees_new
ADD CONSTRAINT emp_email_uk UNIQUE (email);

-- Add check constraint
ALTER TABLE employees_new
ADD CONSTRAINT emp_salary_ck CHECK (salary > 0);

-- Add NOT NULL (use MODIFY, not ADD)
ALTER TABLE employees_new
MODIFY (phone_number NOT NULL);
```

### ALTER TABLE - Drop Constraints
```sql
-- Drop constraint by name
ALTER TABLE employees_new
DROP CONSTRAINT emp_email_uk;

-- Drop primary key
ALTER TABLE employees_new
DROP PRIMARY KEY;

-- Drop primary key with cascade (also drops dependent foreign keys)
ALTER TABLE departments
DROP PRIMARY KEY CASCADE;

-- Drop foreign key
ALTER TABLE employees_new
DROP CONSTRAINT emp_dept_fk;

-- Drop check constraint
ALTER TABLE employees_new
DROP CONSTRAINT emp_salary_ck;
```

### ALTER TABLE - Enable/Disable Constraints
```sql
-- Disable constraint (temporarily bypass validation)
ALTER TABLE employees_new
DISABLE CONSTRAINT emp_dept_fk;

-- Now can insert invalid foreign key values
INSERT INTO employees_new (employee_id, department_id)
VALUES (999, 9999);  -- Department 9999 doesn't exist, but allowed

-- Re-enable constraint
ALTER TABLE employees_new
ENABLE CONSTRAINT emp_dept_fk;
-- Error if invalid data exists

-- Enable with NOVALIDATE (enable for future, ignore existing violations)
ALTER TABLE employees_new
ENABLE NOVALIDATE CONSTRAINT emp_dept_fk;

-- Disable all constraints on table
ALTER TABLE employees_new
DISABLE ALL CONSTRAINTS;
```

### DROP TABLE
```sql
-- Drop table permanently
DROP TABLE employees_new;

-- Table dropped, cannot be recovered (unless Flashback enabled)

-- Drop with CASCADE CONSTRAINTS
DROP TABLE departments CASCADE CONSTRAINTS;
-- Also drops foreign keys in other tables referencing this table

-- Drop table and purge from recycle bin
DROP TABLE employees_new PURGE;
-- Cannot be recovered from recycle bin
```

### TRUNCATE TABLE
```sql
-- Remove all rows, keep structure
TRUNCATE TABLE temp_employees;

-- Faster than DELETE (no undo generated)
-- Cannot be rolled back
-- Resets high water mark

-- Comparison with DELETE:
DELETE FROM temp_employees;  -- Can rollback, slower, generates undo
TRUNCATE TABLE temp_employees;  -- Cannot rollback, faster, no undo

-- TRUNCATE with CASCADE
TRUNCATE TABLE departments CASCADE;
-- Also truncates child tables with ON DELETE CASCADE foreign keys
```

### RENAME Table
```sql
-- Rename table
RENAME employees_new TO employees_archive;

-- Table renamed

-- Rename column (using ALTER TABLE)
ALTER TABLE employees_archive
RENAME COLUMN first_name TO fname;

-- Rename constraint
ALTER TABLE employees_archive
RENAME CONSTRAINT emp_pk TO employees_archive_pk;
```

### COMMENT on Table and Columns
```sql
-- Add comment to table
COMMENT ON TABLE employees IS 
'Contains employee information including salary and department assignment';

-- Add comment to column
COMMENT ON COLUMN employees.salary IS 
'Monthly salary in USD';

-- View comments
SELECT table_name, comments
FROM user_tab_comments
WHERE table_name = 'EMPLOYEES';

-- Result:
TABLE_NAME | COMMENTS
EMPLOYEES  | Contains employee information...

SELECT column_name, comments
FROM user_col_comments
WHERE table_name = 'EMPLOYEES';

-- Result:
COLUMN_NAME | COMMENTS
SALARY      | Monthly salary in USD
```

***

## Module 13: Introduction to Data Dictionary Views

### USER_* Views (Objects Owned by User)
```sql
-- View all tables owned by current user
SELECT table_name
FROM user_tables
ORDER BY table_name;

-- Result:
TABLE_NAME
COUNTRIES
DEPARTMENTS
EMPLOYEES
JOB_HISTORY
JOBS
LOCATIONS
REGIONS

-- View table with details
SELECT table_name, tablespace_name, num_rows
FROM user_tables
WHERE table_name = 'EMPLOYEES';

-- Result:
TABLE_NAME | TABLESPACE_NAME | NUM_ROWS
EMPLOYEES  | USERS          | 107

-- View all your indexes
SELECT index_name, table_name, uniqueness
FROM user_indexes
WHERE table_name = 'EMPLOYEES';

-- Result:
INDEX_NAME  | TABLE_NAME | UNIQUENESS
EMP_EMP_ID_PK | EMPLOYEES | UNIQUE
EMP_NAME_IX   | EMPLOYEES | NONUNIQUE
```

### Viewing Constraints
```sql
-- View all constraints on your tables
SELECT constraint_name, constraint_type, table_name
FROM user_constraints
WHERE table_name = 'EMPLOYEES';

-- Result:
CONSTRAINT_NAME    | CONSTRAINT_TYPE | TABLE_NAME
EMP_EMP_ID_PK     | P              | EMPLOYEES
EMP_DEPT_FK       | R              | EMPLOYEES
EMP_EMAIL_UK      | U              | EMPLOYEES
EMP_SALARY_MIN    | C              | EMPLOYEES

-- Constraint types:
-- P = Primary Key
-- R = Foreign Key (Referential)
-- U = Unique
-- C = Check (and NOT NULL)

-- View constraint details
SELECT constraint_name, search_condition
FROM user_constraints
WHERE table_name = 'EMPLOYEES'
  AND constraint_type = 'C';

-- Result shows CHECK constraint conditions

-- View constraint columns
SELECT constraint_name, column_name, position
FROM user_cons_columns
WHERE table_name = 'EMPLOYEES'
ORDER BY constraint_name, position;

-- Result:
CONSTRAINT_NAME | COLUMN_NAME   | POSITION
EMP_EMP_ID_PK  | EMPLOYEE_ID   | 1
EMP_DEPT_FK    | DEPARTMENT_ID | 1
EMP_EMAIL_UK   | EMAIL         | 1
```

### Viewing Columns
```sql
-- View all columns in a table
SELECT column_name, data_type, data_length, nullable
FROM user_tab_columns
WHERE table_name = 'EMPLOYEES'
ORDER BY column_id;

-- Result:
COLUMN_NAME    | DATA_TYPE   | DATA_LENGTH | NULLABLE
EMPLOYEE_ID    | NUMBER      | 22          | N
FIRST_NAME     | VARCHAR2    | 20          | Y
LAST_NAME      | VARCHAR2    | 25          | N
EMAIL          | VARCHAR2    | 25          | N
PHONE_NUMBER   | VARCHAR2    | 20          | Y
HIRE_DATE      | DATE        | 7           | N
JOB_ID         | VARCHAR2    | 10          | N
SALARY         | NUMBER      | 22          | Y

-- View columns with precision/scale
SELECT column_name, data_type, data_precision, data_scale
FROM user_tab_columns
WHERE table_name = 'EMPLOYEES'
  AND data_type = 'NUMBER';

-- Result:
COLUMN_NAME  | DATA_TYPE | DATA_PRECISION | DATA_SCALE
EMPLOYEE_ID  | NUMBER    | 6             | 0
SALARY       | NUMBER    | 8             | 2
COMMISSION_PCT | NUMBER  | 2             | 2
```

### ALL_* Views (Objects Accessible to User)
```sql
-- View all tables you can access (owned + granted access)
SELECT owner, table_name
FROM all_tables
WHERE table_name LIKE 'EMP%'
ORDER BY owner, table_name;

-- Result:
OWNER  | TABLE_NAME
HR     | EMPLOYEES
HR     | EMP_DETAILS_VIEW
 ## Module 13: Introduction to Data Dictionary Views (Continued)

### ALL_* Views (Continued)
```sql
-- View all tables you can access (owned + granted access)
SELECT owner, table_name
FROM all_tables
WHERE table_name LIKE 'EMP%'
ORDER BY owner, table_name;

-- Result:
OWNER  | TABLE_NAME
HR     | EMPLOYEES
HR     | EMP_DETAILS_VIEW
SCOTT  | EMP
PUBLIC | EMPLOYEES_PUBLIC

-- View accessible objects
SELECT owner, object_name, object_type
FROM all_objects
WHERE object_name LIKE '%DEPT%'
ORDER BY object_type, owner, object_name;

-- Result:
OWNER | OBJECT_NAME  | OBJECT_TYPE
HR    | DEPARTMENTS  | TABLE
HR    | DEPT_VIEW    | VIEW
HR    | DEPT_SEQ     | SEQUENCE
```

### DBA_* Views (All Database Objects - DBA Only)
```sql
-- View all tables in database (requires DBA privileges)
SELECT owner, table_name, num_rows
FROM dba_tables
WHERE owner = 'HR'
ORDER BY table_name;

-- Result (if you have DBA access):
OWNER | TABLE_NAME   | NUM_ROWS
HR    | COUNTRIES    | 25
HR    | DEPARTMENTS  | 27
HR    | EMPLOYEES    | 107

-- View all users in database
SELECT username, account_status, created
FROM dba_users
ORDER BY username;

-- View all tablespaces
SELECT tablespace_name, status, contents
FROM dba_tablespaces;
```

### Viewing Sequences
```sql
-- View your sequences
SELECT sequence_name, min_value, max_value, increment_by, last_number
FROM user_sequences;

-- Result:
SEQUENCE_NAME | MIN_VALUE | MAX_VALUE | INCREMENT_BY | LAST_NUMBER
DEPT_SEQ      | 1         | 9999999   | 10           | 280
EMP_SEQ       | 1         | 9999999   | 1            | 210

-- Detailed sequence information
SELECT sequence_name, cache_size, cycle_flag, order_flag
FROM user_sequences
WHERE sequence_name = 'EMP_SEQ';

-- Result:
SEQUENCE_NAME | CACHE_SIZE | CYCLE_FLAG | ORDER_FLAG
EMP_SEQ       | 20         | N          | N
```

### Viewing Views
```sql
-- View all your views
SELECT view_name, text_length
FROM user_views;

-- Result:
VIEW_NAME       | TEXT_LENGTH
EMP_DETAILS_VW  | 245
DEPT_SUMMARY_VW | 189

-- View the SQL text that defines a view
SELECT view_name, text
FROM user_views
WHERE view_name = 'EMP_DETAILS_VW';

-- Result shows the SELECT statement that creates the view

-- Check if view is updateable
SELECT view_name, updatable, insertable, deletable
FROM user_updatable_columns
WHERE table_name = 'EMP_DETAILS_VW'
  AND column_name = 'SALARY';
```

### Viewing Synonyms
```sql
-- View your synonyms
SELECT synonym_name, table_owner, table_name
FROM user_synonyms;

-- Result:
SYNONYM_NAME | TABLE_OWNER | TABLE_NAME
EMP          | HR          | EMPLOYEES
DEPT         | HR          | DEPARTMENTS

-- View public synonyms available to all users
SELECT synonym_name, table_owner, table_name
FROM all_synonyms
WHERE table_owner = 'HR'
ORDER BY synonym_name;
```

### USER_OBJECTS - All Object Types
```sql
-- View all objects you own
SELECT object_name, object_type, status, created
FROM user_objects
ORDER BY object_type, object_name;

-- Result:
OBJECT_NAME    | OBJECT_TYPE | STATUS  | CREATED
DEPT_SEQ       | SEQUENCE    | VALID   | 15-NOV-25
EMP_SEQ        | SEQUENCE    | VALID   | 15-NOV-25
DEPARTMENTS    | TABLE       | VALID   | 01-JAN-25
EMPLOYEES      | TABLE       | VALID   | 01-JAN-25
EMP_DEPT_FK    | INDEX       | VALID   | 01-JAN-25
EMP_DETAILS_VW | VIEW        | VALID   | 10-NOV-25

-- Count objects by type
SELECT object_type, COUNT(*) AS object_count
FROM user_objects
GROUP BY object_type
ORDER BY object_type;

-- Result:
OBJECT_TYPE  | OBJECT_COUNT
INDEX        | 8
SEQUENCE     | 2
TABLE        | 7
VIEW         | 3

-- Find invalid objects
SELECT object_name, object_type, status
FROM user_objects
WHERE status = 'INVALID';
```

### Practical Data Dictionary Queries
```sql
-- Find tables with no primary key
SELECT table_name
FROM user_tables
WHERE table_name NOT IN (SELECT table_name
                         FROM user_constraints
                         WHERE constraint_type = 'P');

-- Find foreign key relationships
SELECT 
    a.constraint_name AS fk_name,
    a.table_name AS child_table,
    a.column_name AS child_column,
    c.table_name AS parent_table,
    c.column_name AS parent_column
FROM user_cons_columns a
JOIN user_constraints b ON a.constraint_name = b.constraint_name
JOIN user_cons_columns c ON b.r_constraint_name = c.constraint_name
WHERE b.constraint_type = 'R'
ORDER BY a.table_name;

-- Result:
FK_NAME      | CHILD_TABLE | CHILD_COLUMN  | PARENT_TABLE | PARENT_COLUMN
EMP_DEPT_FK  | EMPLOYEES   | DEPARTMENT_ID | DEPARTMENTS  | DEPARTMENT_ID
EMP_JOB_FK   | EMPLOYEES   | JOB_ID        | JOBS         | JOB_ID

-- Find unused indexes (Oracle-specific)
SELECT index_name, table_name
FROM user_indexes
WHERE index_name NOT IN (SELECT index_name
                         FROM v$object_usage
                         WHERE used = 'YES');

-- Find table sizes
SELECT 
    segment_name AS table_name,
    ROUND(bytes/1024/1024, 2) AS size_mb
FROM user_segments
WHERE segment_type = 'TABLE'
ORDER BY bytes DESC;

-- Result:
TABLE_NAME  | SIZE_MB
EMPLOYEES   | 2.50
DEPARTMENTS | 0.06
JOBS        | 0.06
```

***

## Module 14: Creating Sequences, Synonyms, and Indexes

### CREATE SEQUENCE
```sql
-- Basic sequence
CREATE SEQUENCE dept_seq
    START WITH 300
    INCREMENT BY 10;

-- Sequence created

-- Use sequence to generate values
INSERT INTO departments (department_id, department_name)
VALUES (dept_seq.NEXTVAL, 'New Department');

-- Get next value: dept_seq.NEXTVAL (300, 310, 320...)
-- Get current value: dept_seq.CURRVAL (must call NEXTVAL first)

-- Sequence with all options
CREATE SEQUENCE emp_seq
    START WITH 210
    INCREMENT BY 1
    MAXVALUE 9999
    MINVALUE 1
    NOCYCLE
    CACHE 20
    NOORDER;

-- Options explained:
-- START WITH: Starting number
-- INCREMENT BY: Amount to increment (can be negative)
-- MAXVALUE: Maximum value (or NOMAXVALUE)
-- MINVALUE: Minimum value (or NOMINVALUE)
-- CYCLE: Restart when reaching limit (or NOCYCLE)
-- CACHE: Pre-allocate numbers in memory for performance
-- ORDER: Guarantee sequential order (or NOORDER)

-- Cycling sequence
CREATE SEQUENCE invoice_seq
    START WITH 1
    INCREMENT BY 1
    MAXVALUE 999
    MINVALUE 1
    CYCLE
    CACHE 10;

-- When it reaches 999, cycles back to 1
```

### Using Sequences
```sql
-- Insert using sequence
INSERT INTO employees (employee_id, first_name, last_name, email, 
                       hire_date, job_id)
VALUES (emp_seq.NEXTVAL, 'Michael', 'Brown', 'MBROWN',
        SYSDATE, 'IT_PROG');

-- Current value (in same session after NEXTVAL)
SELECT emp_seq.CURRVAL FROM dual;
-- Result: 210

-- Use in SELECT
SELECT emp_seq.NEXTVAL FROM dual;
-- Result: 211 (increments each time)

-- Multiple inserts using sequence
INSERT INTO employees (employee_id, first_name, last_name, email,
                       hire_date, job_id)
SELECT emp_seq.NEXTVAL, first_name, last_name, email,
       hire_date, job_id
FROM temp_employees;
-- Each row gets unique sequential number

-- Cannot use CURRVAL before NEXTVAL in session
SELECT emp_seq.CURRVAL FROM dual;
-- Error if NEXTVAL not called yet in this session
```

### ALTER SEQUENCE
```sql
-- Modify sequence (cannot change START WITH)
ALTER SEQUENCE dept_seq
    INCREMENT BY 20
    MAXVALUE 9999
    CACHE 50;

-- Change to CYCLE
ALTER SEQUENCE invoice_seq
    CYCLE;

-- Change cache size
ALTER SEQUENCE emp_seq
    CACHE 100;

-- Cannot change START WITH - must drop and recreate
-- This is invalid:
-- ALTER SEQUENCE emp_seq START WITH 500;
```

### DROP SEQUENCE
```sql
-- Drop sequence
DROP SEQUENCE dept_seq;

-- Sequence dropped
```

### CREATE INDEX
```sql
-- Simple index on single column
CREATE INDEX emp_last_name_idx
ON employees(last_name);

-- Index created
-- Speeds up queries filtering or sorting by last_name

-- Composite index (multiple columns)
CREATE INDEX emp_name_idx
ON employees(last_name, first_name);

-- Order matters: good for queries on last_name alone or both columns
-- Not useful for queries on first_name alone

-- Unique index
CREATE UNIQUE INDEX emp_email_idx
ON employees(email);

-- Enforces uniqueness like UNIQUE constraint

-- Index with computed column
CREATE INDEX emp_annual_sal_idx
ON employees(salary * 12);

-- Speeds up queries filtering on annual salary
```

### When to Create Indexes
```sql
-- CREATE indexes when:
-- 1. Column frequently used in WHERE clause
CREATE INDEX emp_dept_idx ON employees(department_id);
-- Good for: SELECT * FROM employees WHERE department_id = 60;

-- 2. Column frequently used in JOIN
CREATE INDEX emp_mgr_idx ON employees(manager_id);
-- Good for: SELECT ... FROM employees e JOIN employees m 
--           ON e.manager_id = m.employee_id;

-- 3. Column frequently used in ORDER BY
CREATE INDEX emp_hire_date_idx ON employees(hire_date);
-- Good for: SELECT * FROM employees ORDER BY hire_date;

-- 4. Large tables with selective queries
CREATE INDEX emp_job_idx ON employees(job_id);
-- Good if many different job_ids, bad if only 2-3 values

-- DON'T create indexes when:
-- - Small tables (full table scan faster)
-- - Columns frequently updated (index maintenance overhead)
-- - Low selectivity (column has few distinct values)
-- - Columns already part of primary key (automatically indexed)
```

### Function-Based Indexes
```sql
-- Index on function result
CREATE INDEX emp_upper_last_name_idx
ON employees(UPPER(last_name));

-- Now this query can use the index:
SELECT * FROM employees
WHERE UPPER(last_name) = 'KING';

-- Without function-based index, this wouldn't use regular index

-- Index on expression
CREATE INDEX emp_annual_comp_idx
ON employees(salary * 12 + NVL(commission_pct * salary * 12, 0));

-- Speeds up complex calculations
```

### Viewing Index Information
```sql
-- View your indexes
SELECT index_name, table_name, uniqueness, status
FROM user_indexes
WHERE table_name = 'EMPLOYEES';

-- Result:
INDEX_NAME          | TABLE_NAME | UNIQUENESS | STATUS
EMP_EMP_ID_PK      | EMPLOYEES  | UNIQUE     | VALID
EMP_LAST_NAME_IDX  | EMPLOYEES  | NONUNIQUE  | VALID
EMP_EMAIL_IDX      | EMPLOYEES  | UNIQUE     | VALID

-- View indexed columns
SELECT index_name, column_name, column_position
FROM user_ind_columns
WHERE table_name = 'EMPLOYEES'
ORDER BY index_name, column_position;

-- Result:
INDEX_NAME         | COLUMN_NAME | COLUMN_POSITION
EMP_EMP_ID_PK     | EMPLOYEE_ID | 1
EMP_NAME_IDX      | LAST_NAME   | 1
EMP_NAME_IDX      | FIRST_NAME  | 2
EMP_EMAIL_IDX     | EMAIL       | 1
```

### DROP INDEX
```sql
-- Drop index
DROP INDEX emp_last_name_idx;

-- Index dropped

-- Cannot drop primary key or unique constraint indexes directly
-- Must drop the constraint instead
ALTER TABLE employees DROP CONSTRAINT emp_email_uk;
-- This also drops the associated unique index
```

### CREATE SYNONYM
```sql
-- Create private synonym (for current user)
CREATE SYNONYM emp
FOR employees;

-- Now can use shorter name:
SELECT * FROM emp;
-- Same as: SELECT * FROM employees;

-- Synonym for table in another schema
CREATE SYNONYM hr_emp
FOR hr.employees;

-- Use it:
SELECT * FROM hr_emp;

-- Synonym for view
CREATE SYNONYM emp_view
FOR employee_details_view;

-- Synonym for sequence
CREATE SYNONYM emp_id
FOR emp_seq;

-- Use it:
INSERT INTO employees VALUES (emp_id.NEXTVAL, ...);
```

### PUBLIC SYNONYM
```sql
-- Create public synonym (accessible to all users)
CREATE PUBLIC SYNONYM departments
FOR hr.departments;

-- Any user can now use:
SELECT * FROM departments;

-- Requires CREATE PUBLIC SYNONYM privilege

-- Typical use: System-wide shortcuts
CREATE PUBLIC SYNONYM emp FOR hr.employees;
CREATE PUBLIC SYNONYM dept FOR hr.departments;
```

### DROP SYNONYM
```sql
-- Drop private synonym
DROP SYNONYM emp;

-- Drop public synonym
DROP PUBLIC SYNONYM departments;

-- Requires DROP PUBLIC SYNONYM privilege
```

### Practical Synonym Example
```sql
-- Before synonyms:
SELECT e.employee_id, e.first_name, d.department_name
FROM hr.employees e
JOIN hr.departments d ON e.department_id = d.department_id;

-- Create synonyms:
CREATE SYNONYM emp FOR hr.employees;
CREATE SYNONYM dept FOR hr.departments;

-- Now simpler:
SELECT e.employee_id, e.first_name, d.department_name
FROM emp e
JOIN dept d ON e.department_id = d.department_id;
```

***

## Module 15: Creating Views

### CREATE VIEW - Simple View
```sql
-- Basic view
CREATE VIEW emp_view AS
SELECT employee_id, first_name, last_name, salary, department_id
FROM employees;

-- Use view like a table:
SELECT * FROM emp_view;

-- View with renamed columns
CREATE VIEW emp_details AS
SELECT employee_id AS emp_id,
       first_name || ' ' || last_name AS full_name,
       salary,
       department_id AS dept_id
FROM employees;

-- Query the view:
SELECT * FROM emp_details WHERE dept_id = 60;

-- View with WHERE clause
CREATE VIEW it_employees AS
SELECT employee_id, first_name, last_name, salary
FROM employees
WHERE department_id = 60;

-- Result: Only IT employees visible
```

### CREATE VIEW - Complex View
```sql
-- View with JOIN
CREATE VIEW emp_dept_view AS
SELECT e.employee_id,
       e.first_name,
       e.last_name,
       e.salary,
       d.department_name,
       l.city
FROM employees e   

## Module 15: Creating Views (Continued)

### CREATE VIEW - Complex View (Continued)
```sql
-- View with JOIN
CREATE VIEW emp_dept_view AS
SELECT e.employee_id,
       e.first_name,
       e.last_name,
       e.salary,
       d.department_name,
       l.city
FROM employees e
JOIN departments d ON e.department_id = d.department_id
JOIN locations l ON d.location_id = l.location_id;

-- Use complex view:
SELECT * FROM emp_dept_view WHERE city = 'Seattle';

-- View with GROUP BY
CREATE VIEW dept_salary_summary AS
SELECT d.department_name,
       COUNT(e.employee_id) AS emp_count,
       SUM(e.salary) AS total_salary,
       ROUND(AVG(e.salary), 2) AS avg_salary,
       MIN(e.salary) AS min_salary,
       MAX(e.salary) AS max_salary
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_name;

-- Query aggregated view:
SELECT * FROM dept_salary_summary
WHERE emp_count > 5
ORDER BY avg_salary DESC;

-- Result:
DEPARTMENT_NAME | EMP_COUNT | TOTAL_SALARY | AVG_SALARY | MIN_SALARY | MAX_SALARY
Sales          | 6         | 60200        | 10033.33   | 6100       | 14000
Shipping       | 5         | 17500        | 3500       | 2500       | 4400
```

### CREATE OR REPLACE VIEW
```sql
-- Modify existing view without dropping
CREATE OR REPLACE VIEW emp_view AS
SELECT employee_id, first_name, last_name, salary, 
       department_id, hire_date
FROM employees
WHERE salary > 5000;

-- Adds hire_date and WHERE clause to existing view
-- No need to drop and recreate
-- Preserves grants on the view

-- Replace with different structure
CREATE OR REPLACE VIEW emp_view AS
SELECT employee_id, first_name, last_name,
       salary * 12 AS annual_salary,
       department_id
FROM employees;
```

### View Column Aliases
```sql
-- Method 1: Aliases in SELECT
CREATE VIEW emp_summary AS
SELECT employee_id AS emp_id,
       first_name AS fname,
       salary * 12 AS annual_sal
FROM employees;

-- Method 2: Column list in CREATE VIEW
CREATE VIEW emp_summary (emp_id, fname, annual_sal) AS
SELECT employee_id, first_name, salary * 12
FROM employees;

-- Both produce same result:
SELECT * FROM emp_summary;

-- Result:
EMP_ID | FNAME     | ANNUAL_SAL
100    | Steven    | 288000
101    | Neena     | 204000
```

### WITH CHECK OPTION
```sql
-- Prevent updates that would remove rows from view
CREATE OR REPLACE VIEW it_emp_view AS
SELECT employee_id, first_name, last_name, salary, department_id
FROM employees
WHERE department_id = 60
WITH CHECK OPTION CONSTRAINT it_emp_ck;

-- Can update IT employees:
UPDATE it_emp_view
SET salary = salary * 1.10
WHERE employee_id = 103;
-- Success

-- Cannot change department (would remove from view):
UPDATE it_emp_view
SET department_id = 90
WHERE employee_id = 103;
-- Error: view WITH CHECK OPTION where-clause violation

-- Cannot insert non-IT employees:
INSERT INTO it_emp_view (employee_id, first_name, last_name, 
                         email, hire_date, job_id, department_id)
VALUES (999, 'Test', 'User', 'TUSER', SYSDATE, 'SA_REP', 80);
-- Error: view WITH CHECK OPTION where-clause violation

-- Valid insert (IT department):
INSERT INTO it_emp_view 
VALUES (999, 'Test', 'User', 7000, 60);
-- Success
```

### WITH READ ONLY
```sql
-- Create read-only view (no DML allowed)
CREATE OR REPLACE VIEW emp_readonly AS
SELECT employee_id, first_name, last_name, salary
FROM employees
WHERE department_id = 60
WITH READ ONLY;

-- Can query:
SELECT * FROM emp_readonly;
-- Success

-- Cannot update:
UPDATE emp_readonly
SET salary = salary * 1.10;
-- Error: cannot perform a DML operation on a read-only view

-- Cannot insert:
INSERT INTO emp_readonly VALUES (999, 'Test', 'User', 5000);
-- Error: cannot perform a DML operation on a read-only view

-- Cannot delete:
DELETE FROM emp_readonly WHERE employee_id = 103;
-- Error: cannot perform a DML operation on a read-only view
```

### Updatable Views
```sql
-- Simple view (usually updatable)
CREATE VIEW emp_simple AS
SELECT employee_id, first_name, last_name, salary, department_id
FROM employees;

-- Can perform DML:
UPDATE emp_simple
SET salary = 10000
WHERE employee_id = 107;
-- Success - updates base table

INSERT INTO emp_simple (employee_id, first_name, last_name, 
                        email, hire_date, job_id, department_id)
VALUES (999, 'New', 'Employee', 'NEWEMP', SYSDATE, 'IT_PROG', 60);
-- Success if all NOT NULL columns included

DELETE FROM emp_simple WHERE employee_id = 999;
-- Success - deletes from base table

-- Non-updatable view (contains GROUP BY)
CREATE VIEW dept_summary AS
SELECT department_id, COUNT(*) AS emp_count, AVG(salary) AS avg_sal
FROM employees
GROUP BY department_id;

-- Cannot update aggregated data:
UPDATE dept_summary SET avg_sal = 10000 WHERE department_id = 60;
-- Error: cannot modify a column which maps to a non key-preserved table

-- Rules for updatable views:
-- ✓ Single base table
-- ✓ No GROUP BY, DISTINCT, or aggregate functions
-- ✓ No set operators (UNION, INTERSECT, MINUS)
-- ✓ No subqueries in SELECT list
-- ✗ Cannot update computed columns
```

### Inline Views (Views in FROM Clause)
```sql
-- Temporary view in query
SELECT e.first_name, e.salary, dept_avg.avg_salary
FROM employees e,
     (SELECT department_id, AVG(salary) AS avg_salary
      FROM employees
      GROUP BY department_id) dept_avg
WHERE e.department_id = dept_avg.department_id
  AND e.salary > dept_avg.avg_salary;

-- Result: Employees earning above department average

-- Complex inline view
SELECT emp_data.full_name, emp_data.dept_name, emp_data.city
FROM (SELECT e.first_name || ' ' || e.last_name AS full_name,
             d.department_name AS dept_name,
             l.city
      FROM employees e
      JOIN departments d ON e.department_id = d.department_id
      JOIN locations l ON d.location_id = l.location_id
      WHERE e.salary > 10000) emp_data
WHERE emp_data.city = 'Seattle';
```

### TOP-N Analysis with Views
```sql
-- Find top 5 highest paid employees
CREATE VIEW top5_earners AS
SELECT employee_id, first_name, last_name, salary
FROM (SELECT employee_id, first_name, last_name, salary
      FROM employees
      ORDER BY salary DESC)
WHERE ROWNUM <= 5;

-- Query the view:
SELECT * FROM top5_earners;

-- Result:
EMPLOYEE_ID | FIRST_NAME | LAST_NAME | SALARY
100        | Steven     | King      | 24000
101        | Neena      | Kochhar   | 17000
102        | Lex        | De Haan   | 17000
108        | Nancy      | Greenberg | 12008
205        | Shelley    | Higgins   | 12008

-- Alternative using FETCH (Oracle 12c+):
CREATE VIEW top5_earners_v2 AS
SELECT employee_id, first_name, last_name, salary
FROM employees
ORDER BY salary DESC
FETCH FIRST 5 ROWS ONLY;
```

### DROP VIEW
```sql
-- Drop view
DROP VIEW emp_view;

-- View dropped
-- Base table unaffected

-- Attempting to use dropped view:
SELECT * FROM emp_view;
-- Error: table or view does not exist
```

### Practical View Examples
```sql
-- Security view - hide sensitive data
CREATE VIEW emp_public AS
SELECT employee_id, first_name, last_name, job_id, department_id
FROM employees;
-- Excludes salary, commission, SSN, etc.

-- Grant access to view instead of base table
GRANT SELECT ON emp_public TO public_users;

-- Simplification view - complex joins made easy
CREATE VIEW emp_complete_info AS
SELECT e.employee_id,
       e.first_name || ' ' || e.last_name AS employee_name,
       j.job_title,
       e.salary,
       d.department_name,
       l.city || ', ' || l.state_province AS location,
       c.country_name,
       r.region_name,
       m.first_name || ' ' || m.last_name AS manager_name
FROM employees e
LEFT JOIN jobs j ON e.job_id = j.job_id
LEFT JOIN departments d ON e.department_id = d.department_id
LEFT JOIN locations l ON d.location_id = l.location_id
LEFT JOIN countries c ON l.country_id = c.country_id
LEFT JOIN regions r ON c.region_id = r.region_id
LEFT JOIN employees m ON e.manager_id = m.employee_id;

-- Now simple queries:
SELECT * FROM emp_complete_info WHERE region_name = 'Americas';
```

***

## Module 16: Managing Schema Objects

### Add Column with DEFAULT
```sql
-- Add column to existing table
ALTER TABLE employees
ADD (bonus NUMBER(8,2) DEFAULT 0);

-- All existing rows get default value 0

-- Add multiple columns
ALTER TABLE employees
ADD (performance_rating NUMBER(1),
     notes VARCHAR2(500),
     last_review_date DATE DEFAULT SYSDATE);
```

### Modify Column
```sql
-- Increase column width
ALTER TABLE employees
MODIFY (first_name VARCHAR2(50));

-- Decrease column width (only if data fits)
ALTER TABLE employees
MODIFY (first_name VARCHAR2(15));
-- Error if any first_name > 15 chars

-- Change data type (table must be empty or data compatible)
ALTER TABLE employees
MODIFY (employee_id VARCHAR2(10));

-- Add NOT NULL constraint to existing column
ALTER TABLE employees
MODIFY (email NOT NULL);
-- Error if any NULL values exist

-- Remove NOT NULL constraint
ALTER TABLE employees
MODIFY (phone_number NULL);

-- Change default value
ALTER TABLE employees
MODIFY (bonus DEFAULT 1000);
-- Only affects future inserts, not existing rows
```

### Drop Column
```sql
-- Drop single column
ALTER TABLE employees
DROP COLUMN bonus;

-- Drop multiple columns
ALTER TABLE employees
DROP (performance_rating, notes, last_review_date);

-- Drop column with dependencies
ALTER TABLE employees
DROP COLUMN department_id CASCADE CONSTRAINTS;
-- Also drops foreign keys referencing this column

-- Set column unused (faster, physical removal delayed)
ALTER TABLE employees
SET UNUSED (notes);

-- Query still works, column hidden
SELECT * FROM employees;
-- notes column not displayed

-- Later remove unused columns
ALTER TABLE employees
DROP UNUSED COLUMNS;
-- Physical removal happens now
```

### Rename Column
```sql
-- Rename column
ALTER TABLE employees
RENAME COLUMN first_name TO fname;

-- Now use new name:
SELECT fname FROM employees;

-- Rename back:
ALTER TABLE employees
RENAME COLUMN fname TO first_name;
```

### Add Constraints to Existing Table
```sql
-- Add PRIMARY KEY
ALTER TABLE temp_employees
ADD CONSTRAINT temp_emp_pk PRIMARY KEY (employee_id);

-- Add FOREIGN KEY
ALTER TABLE temp_employees
ADD CONSTRAINT temp_emp_dept_fk 
    FOREIGN KEY (department_id)
    REFERENCES departments(department_id)
    ON DELETE CASCADE;

-- Add UNIQUE constraint
ALTER TABLE temp_employees
ADD CONSTRAINT temp_emp_email_uk UNIQUE (email);

-- Add CHECK constraint
ALTER TABLE temp_employees
ADD CONSTRAINT temp_emp_sal_ck CHECK (salary BETWEEN 2000 AND 50000);

-- Add NOT NULL (use MODIFY)
ALTER TABLE temp_employees
MODIFY (email CONSTRAINT temp_emp_email_nn NOT NULL);
```

### Drop Constraints
```sql
-- Drop PRIMARY KEY
ALTER TABLE temp_employees
DROP PRIMARY KEY;

-- Drop PRIMARY KEY with CASCADE
ALTER TABLE departments
DROP PRIMARY KEY CASCADE;
-- Also drops dependent foreign keys in other tables

-- Drop FOREIGN KEY
ALTER TABLE employees
DROP CONSTRAINT emp_dept_fk;

-- Drop UNIQUE constraint
ALTER TABLE employees
DROP CONSTRAINT emp_email_uk;

-- Drop CHECK constraint
ALTER TABLE employees
DROP CONSTRAINT emp_salary_ck;

-- Drop NOT NULL (use MODIFY)
ALTER TABLE employees
MODIFY (commission_pct NULL);
```

### Enable and Disable Constraints
```sql
-- Disable constraint temporarily
ALTER TABLE employees
DISABLE CONSTRAINT emp_dept_fk;

-- Now can insert invalid data:
INSERT INTO employees (employee_id, first_name, last_name,
                       email, hire_date, job_id, department_id)
VALUES (999, 'Test', 'User', 'TUSER', SYSDATE, 'IT_PROG', 9999);
-- Success even though dept 9999 doesn't exist

-- Re-enable constraint
ALTER TABLE employees
ENABLE CONSTRAINT emp_dept_fk;
-- Error if invalid data exists

-- Enable NOVALIDATE (apply to future only, ignore existing violations)
ALTER TABLE employees
ENABLE NOVALIDATE CONSTRAINT emp_dept_fk;
-- Constraint active for new data, existing violations ignored

-- Disable all constraints on table
ALTER TABLE employees
DISABLE ALL TRIGGERS;

-- Enable all constraints
ALTER TABLE employees
ENABLE ALL TRIGGERS;
```

### Rename Table
```sql
-- Rename table
RENAME employees TO employees_backup;

-- Table renamed
-- All indexes, constraints, grants preserved

-- Use new name:
SELECT * FROM employees_backup;

-- Rename back:
RENAME employees_backup TO employees;
```

### TRUNCATE vs DELETE
```sql
-- DELETE - removes rows, can rollback, slower
DELETE FROM temp_employees;
-- Can ROLLBACK
-- Generates undo information
-- Triggers fire
-- Keeps storage allocated

ROLLBACK;  -- Data restored

-- TRUNCATE - removes all rows, cannot rollback, faster
TRUNCATE TABLE temp_employees;
-- Cannot ROLLBACK
-- Minimal undo generated
-- Triggers don't fire
-- Releases storage (resets high-water mark)
-- Faster for large tables

-- Attempting rollback after truncate:
ROLLBACK;
-- No effect, data is gone
```

### External Tables
```sql
-- Create external table (data stored in file)
CREATE TABLE external_employees (
    employee_id   NUMBER(6),
    first_name    VARCHAR2(20),
    last_name     VARCHAR2(25),
    email         VARCHAR2(25),
    hire_date     DATE,
    salary        NUMBER(8,2)
)
ORGANIZATION EXTERNAL (
    TYPE ORACLE_LOADER
    DEFAULT DIRECTORY data_dir
    ACCESS PARAMETERS (
        RECORDS DELIMITED BY NEWLINE
        FIELDS TERMINATED BY ','
        MISSING FIELD VALUES ARE NULL
    )
    LOCATION ('employees.csv')
)
REJECT LIMIT UNLIMITED;

-- Query like normal table:
SELECT * FROM external_employees;

-- Data comes from employees.csv file
-- Cannot INSERT, UPDATE, or DELETE
-- Useful for loading data from files
```

***

## Module 17: Retrieving Data by Using Subqueries (Advanced)

### Multiple-Column Subqueries - Pairwise
```sql

## Module 17: Retrieving Data by Using Subqueries (Advanced) (Continued)

### Multiple-Column Subqueries - Pairwise
```sql
-- Compare multiple columns together as pairs
SELECT employee_id, first_name, department_id, salary
FROM employees
WHERE (department_id, salary) IN 
      (SELECT department_id, MAX(salary)
       FROM employees
       GROUP BY department_id);

-- Result: Highest paid employee in each department
EMPLOYEE_ID | FIRST_NAME | DEPARTMENT_ID | SALARY
100        | Steven     | 90           | 24000
103        | Alexander  | 60           | 9000
120        | Matthew    | 50           | 8000
145        | John       | 80           | 14000

-- Pairwise comparison ensures BOTH conditions match together
-- Employee must be in the department AND have the max salary for THAT dept
```

### Multiple-Column Subqueries - Non-Pairwise
```sql
-- Compare columns separately
SELECT employee_id, first_name, department_id, salary
FROM employees
WHERE department_id IN (SELECT department_id
                        FROM employees
                        WHERE first_name = 'John')
  AND salary IN (SELECT MAX(salary)
                 FROM employees
                 GROUP BY department_id);

-- Result: Different from pairwise!
-- Employee can be in John's dept OR have max salary from ANY dept

-- Example showing difference:
-- Pairwise: Must have max salary in THEIR OWN department
-- Non-pairwise: Can have max salary from ANY department
```

### Scalar Subqueries
```sql
-- Subquery that returns single value (one row, one column)
SELECT employee_id,
       first_name,
       salary,
       (SELECT AVG(salary) FROM employees) AS company_avg,
       salary - (SELECT AVG(salary) FROM employees) AS diff_from_avg
FROM employees;

-- Result:
EMPLOYEE_ID | FIRST_NAME | SALARY | COMPANY_AVG | DIFF_FROM_AVG
100        | Steven     | 24000  | 6461.83     | 17538.17
103        | Alexander  | 9000   | 6461.83     | 2538.17
107        | Diana      | 4200   | 6461.83     | -2261.83

-- Scalar subquery in SELECT list
SELECT e.first_name,
       e.salary,
       e.department_id,
       (SELECT department_name 
        FROM departments d 
        WHERE d.department_id = e.department_id) AS dept_name
FROM employees e;

-- Scalar subquery in ORDER BY
SELECT first_name, salary, department_id
FROM employees
ORDER BY (SELECT AVG(salary) 
          FROM employees e2 
          WHERE e2.department_id = employees.department_id);
-- Orders by department average salary
```

### Correlated Subqueries (Advanced)
```sql
-- Inner query references outer query
-- Find employees earning more than their department average
SELECT e.employee_id,
       e.first_name,
       e.salary,
       e.department_id
FROM employees e
WHERE e.salary > (SELECT AVG(salary)
                  FROM employees
                  WHERE department_id = e.department_id);

-- Execution process:
-- 1. Outer query selects first employee
-- 2. Inner query calculates average for that employee's department
-- 3. Compare employee's salary to department average
-- 4. Repeat for each employee

-- Find employees who have subordinates
SELECT e.employee_id, e.first_name, e.last_name
FROM employees e
WHERE EXISTS (SELECT 1
              FROM employees
              WHERE manager_id = e.employee_id);

-- Result:
EMPLOYEE_ID | FIRST_NAME | LAST_NAME
100        | Steven     | King
101        | Neena      | Kochhar
102        | Lex        | De Haan
103        | Alexander  | Hunold

-- Find employees without subordinates
SELECT e.employee_id, e.first_name, e.last_name
FROM employees e
WHERE NOT EXISTS (SELECT 1
                  FROM employees
                  WHERE manager_id = e.employee_id);
```

### EXISTS vs IN
```sql
-- Using EXISTS (more efficient for large datasets)
SELECT d.department_name
FROM departments d
WHERE EXISTS (SELECT 1
              FROM employees e
              WHERE e.department_id = d.department_id);

-- Using IN (simpler syntax)
SELECT d.department_name
FROM departments d
WHERE d.department_id IN (SELECT department_id
                          FROM employees);

-- Both return same result: departments with employees

-- EXISTS advantages:
-- - Stops at first match (faster)
-- - Better with NULLs
-- - More efficient for large inner queries

-- NOT EXISTS vs NOT IN
SELECT d.department_name
FROM departments d
WHERE NOT EXISTS (SELECT 1
                  FROM employees e
                  WHERE e.department_id = d.department_id);

-- Result: Departments without employees

-- NOT IN has NULL problem:
SELECT department_name
FROM departments
WHERE department_id NOT IN (SELECT department_id FROM employees);
-- Returns NO rows if any employee has NULL department_id!

-- Safe version:
SELECT department_name
FROM departments
WHERE department_id NOT IN (SELECT department_id 
                            FROM employees 
                            WHERE department_id IS NOT NULL);
```

### Subqueries with UPDATE
```sql
-- Update based on subquery
UPDATE employees
SET salary = (SELECT AVG(salary)
              FROM employees
              WHERE job_id = 'IT_PROG')
WHERE employee_id = 999;

-- Update multiple columns with subquery
UPDATE employees e
SET (salary, commission_pct) = 
    (SELECT AVG(salary), AVG(commission_pct)
     FROM employees
     WHERE department_id = e.department_id)
WHERE employee_id = 999;

-- Correlated update
UPDATE employees e
SET salary = salary * 1.1
WHERE salary < (SELECT AVG(salary)
                FROM employees
                WHERE department_id = e.department_id);
-- Raises salary for employees below dept average
```

### Subqueries with DELETE
```sql
-- Delete based on subquery
DELETE FROM employees
WHERE salary < (SELECT AVG(salary) FROM employees);
-- Deletes employees earning below average

-- Delete using correlated subquery
DELETE FROM employees e
WHERE salary = (SELECT MIN(salary)
                FROM employees
                WHERE department_id = e.department_id);
-- Deletes lowest paid employee in each department

-- Delete with NOT EXISTS
DELETE FROM departments d
WHERE NOT EXISTS (SELECT 1
                  FROM employees e
                  WHERE e.department_id = d.department_id);
-- Deletes departments with no employees
```

### WITH Clause (Subquery Factoring)
```sql
-- Define subquery once, use multiple times
WITH 
dept_costs AS (
    SELECT department_id, SUM(salary) AS total_salary
    FROM employees
    GROUP BY department_id
),
avg_cost AS (
    SELECT AVG(total_salary) AS avg_dept_salary
    FROM dept_costs
)
SELECT d.department_name, 
       dc.total_salary,
       ac.avg_dept_salary,
       dc.total_salary - ac.avg_dept_salary AS difference
FROM departments d
JOIN dept_costs dc ON d.department_id = dc.department_id
CROSS JOIN avg_cost ac
WHERE dc.total_salary > ac.avg_dept_salary;

-- Result: Departments with above-average total salary

-- Multiple CTEs (Common Table Expressions)
WITH
high_earners AS (
    SELECT employee_id, first_name, salary, department_id
    FROM employees
    WHERE salary > 10000
),
dept_summary AS (
    SELECT department_id, COUNT(*) AS high_earner_count
    FROM high_earners
    GROUP BY department_id
)
SELECT d.department_name, 
       NVL(ds.high_earner_count, 0) AS high_earners
FROM departments d
LEFT JOIN dept_summary ds ON d.department_id = ds.department_id
ORDER BY high_earners DESC;
```

### Recursive Subqueries (Hierarchical Queries)
```sql
-- Find organizational hierarchy using CONNECT BY
SELECT LEVEL,
       LPAD(' ', 2 * (LEVEL - 1)) || first_name AS employee_hierarchy,
       employee_id,
       manager_id
FROM employees
START WITH manager_id IS NULL  -- Start with CEO
CONNECT BY PRIOR employee_id = manager_id
ORDER SIBLINGS BY first_name;

-- Result:
LEVEL | EMPLOYEE_HIERARCHY | EMPLOYEE_ID | MANAGER_ID
1     | Steven            | 100         | NULL
2     |   Neena           | 101         | 100
3     |     Nancy         | 108         | 101
4     |       Daniel      | 109         | 108
2     |   Lex             | 102         | 100
3     |     Alexander     | 103         | 102

-- Recursive CTE (Oracle 11g R2+)
WITH emp_hierarchy (employee_id, first_name, manager_id, level_num) AS (
    -- Anchor: Start with CEO
    SELECT employee_id, first_name, manager_id, 1
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: Get subordinates
    SELECT e.employee_id, e.first_name, e.manager_id, eh.level_num + 1
    FROM employees e
    JOIN emp_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT LPAD(' ', 2 * (level_num - 1)) || first_name AS hierarchy,
       level_num
FROM emp_hierarchy
ORDER BY level_num, first_name;
```

***

## Module 18: Manipulating Data by Using Subqueries

### INSERT with Subquery
```sql
-- Copy data from one table to another
INSERT INTO sales_reps (employee_id, first_name, last_name, hire_date, salary)
SELECT employee_id, first_name, last_name, hire_date, salary
FROM employees
WHERE job_id = 'SA_REP';

-- Insert with calculations
INSERT INTO bonus_table (employee_id, bonus_amount, bonus_date)
SELECT employee_id, 
       salary * 0.10,
       SYSDATE
FROM employees
WHERE commission_pct IS NOT NULL;

-- Insert with JOIN in subquery
INSERT INTO dept_summary (dept_id, dept_name, emp_count, total_sal)
SELECT d.department_id,
       d.department_name,
       COUNT(e.employee_id),
       SUM(e.salary)
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_id, d.department_name;
```

### Multi-Table INSERT
```sql
-- Unconditional INSERT ALL (insert into all tables)
INSERT ALL
    INTO sales_info VALUES (employee_id, first_name, hire_date, salary)
    INTO managers_info VALUES (employee_id, first_name, department_id)
SELECT employee_id, first_name, hire_date, salary, department_id
FROM employees
WHERE job_id = 'SA_MAN';

-- Each qualifying row inserted into BOTH tables

-- Conditional INSERT ALL
INSERT ALL
    WHEN salary > 15000 THEN
        INTO high_earners VALUES (employee_id, first_name, salary)
    WHEN salary BETWEEN 10000 AND 15000 THEN
        INTO medium_earners VALUES (employee_id, first_name, salary)
    WHEN salary < 10000 THEN
        INTO low_earners VALUES (employee_id, first_name, salary)
SELECT employee_id, first_name, salary
FROM employees;

-- Row can be inserted into multiple tables based on conditions

-- INSERT FIRST (stop after first match)
INSERT FIRST
    WHEN salary > 15000 THEN
        INTO high_earners VALUES (employee_id, first_name, salary)
    WHEN salary > 10000 THEN
        INTO medium_earners VALUES (employee_id, first_name, salary)
    ELSE
        INTO low_earners VALUES (employee_id, first_name, salary)
SELECT employee_id, first_name, salary
FROM employees;

-- Each row inserted into only FIRST matching table
```

### Pivoting INSERT
```sql
-- Transform columns into rows
INSERT ALL
    INTO sales_data VALUES (employee_id, 'Q1', q1_sales)
    INTO sales_data VALUES (employee_id, 'Q2', q2_sales)
    INTO sales_data VALUES (employee_id, 'Q3', q3_sales)
    INTO sales_data VALUES (employee_id, 'Q4', q4_sales)
SELECT employee_id, q1_sales, q2_sales, q3_sales, q4_sales
FROM quarterly_sales;

-- Before: One row per employee with 4 columns
-- After: Four rows per employee with 1 column each

-- Example:
-- Source: EMP_ID | Q1   | Q2   | Q3   | Q4
--         100    | 5000 | 6000 | 5500 | 7000
-- Result: EMP_ID | QUARTER | AMOUNT
--         100    | Q1      | 5000
--         100    | Q2      | 6000
--         100    | Q3      | 5500
--         100    | Q4      | 7000
```

### UPDATE with Correlated Subquery
```sql
-- Update based on related data
UPDATE employees e
SET salary = (SELECT AVG(salary) * 1.1
              FROM employees
              WHERE department_id = e.department_id)
WHERE department_id IN (SELECT department_id
                        FROM departments
                        WHERE location_id = 1700);

-- Updates employees in specific location to 110% of dept average

-- Update with multiple column subquery
UPDATE employees e
SET (salary, commission_pct) = 
    (SELECT MAX(salary), MAX(commission_pct)
     FROM employees
     WHERE job_id = e.job_id)
WHERE employee_id IN (SELECT employee_id
                      FROM job_history
                      WHERE end_date > ADD_MONTHS(SYSDATE, -12));

-- Update recently promoted employees to max for their job
```

### DELETE with Subquery
```sql
-- Delete based on subquery condition
DELETE FROM employees
WHERE department_id IN (SELECT department_id
                        FROM departments
                        WHERE location_id = 2500);

-- Deletes employees in departments at specific location

-- Delete with correlated subquery
DELETE FROM employees e
WHERE salary < (SELECT AVG(salary) * 0.8
                FROM employees
                WHERE department_id = e.department_id);

-- Deletes employees earning less than 80% of dept average

-- Delete with NOT EXISTS
DELETE FROM job_history jh
WHERE NOT EXISTS (SELECT 1
                  FROM employees e
                  WHERE e.employee_id = jh.employee_id);

-- Deletes job history for employees no longer in company
```

### WITH Clause for DML
```sql
-- Use WITH clause in UPDATE
WITH dept_avg AS (
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
)
UPDATE employees e
SET salary = (SELECT avg_salary * 1.05
              FROM dept_avg da
              WHERE da.department_id = e.department_id)
WHERE EXISTS (SELECT 1 FROM dept_avg WHERE department_id = e.department_id);

-- Raises salary to 105% of department average

-- WITH clause in DELETE
WITH low_performers AS (
    SELECT employee_id
    FROM employees
    WHERE salary < (SELECT AVG(salary) * 0.7 FROM employees)
      AND MONTHS_BETWEEN(SYSDATE, hire_date) > 60
)
DELETE FROM employees
WHERE employee_id IN (SELECT employee_id FROM low_performers);

-- Deletes long-term employees with very low salaries
```

***

## Module 19: Controlling User Access

### System Privileges
```sql
-- Grant system-level privileges (requires DBA)
GRANT CREATE SESSION TO user1;

## Module 19: Controlling User Access (Continued)

### System Privileges (Continued)
```sql
-- Grant system-level privileges (requires DBA)
GRANT CREATE SESSION TO user1;
-- Allows user1 to connect to database

GRANT CREATE TABLE TO user1;
-- Allows user1 to create tables in their schema

GRANT CREATE VIEW TO user1;
-- Allows user1 to create views

-- Grant multiple privileges
GRANT CREATE TABLE, CREATE VIEW, CREATE SEQUENCE TO user1;

-- Grant with ADMIN OPTION
GRANT CREATE TABLE TO user1 WITH ADMIN OPTION;
-- user1 can now grant CREATE TABLE to other users

-- Common system privileges:
-- CREATE SESSION - connect to database
-- CREATE TABLE - create tables
-- CREATE VIEW - create views
-- CREATE PROCEDURE - create procedures/functions
-- CREATE SEQUENCE - create sequences
-- CREATE SYNONYM - create synonyms
-- CREATE USER - create new users
-- DROP USER - drop users
-- ALTER USER - modify user properties
```

### Object Privileges
```sql
-- Grant privileges on specific objects
-- SELECT privilege
GRANT SELECT ON employees TO user1;
-- user1 can now query employees table

-- INSERT privilege
GRANT INSERT ON employees TO user1;
-- user1 can insert rows into employees

-- UPDATE privilege (all columns)
GRANT UPDATE ON employees TO user1;

-- UPDATE privilege (specific columns)
GRANT UPDATE (salary, commission_pct) ON employees TO user1;
-- user1 can only update salary and commission_pct columns

-- DELETE privilege
GRANT DELETE ON employees TO user1;

-- REFERENCES privilege (create foreign keys)
GRANT REFERENCES ON departments TO user1;
-- user1 can create foreign keys referencing departments

-- Multiple privileges at once
GRANT SELECT, INSERT, UPDATE, DELETE ON employees TO user1;

-- Grant with GRANT OPTION
GRANT SELECT ON employees TO user1 WITH GRANT OPTION;
-- user1 can grant SELECT on employees to other users

-- Grant on views
GRANT SELECT ON emp_details_view TO user1;

-- Grant on sequences
GRANT SELECT ON emp_seq TO user1;
-- Allows using NEXTVAL and CURRVAL
```

### ALL PRIVILEGES
```sql
-- Grant all available privileges on object
GRANT ALL ON employees TO user1;
-- Includes SELECT, INSERT, UPDATE, DELETE, etc.

-- Grant all privileges with GRANT OPTION
GRANT ALL ON departments TO user1 WITH GRANT OPTION;
```

### PUBLIC Grants
```sql
-- Grant to all users
GRANT SELECT ON employees TO PUBLIC;
-- All database users can now query employees

-- Grant multiple privileges to PUBLIC
GRANT SELECT, INSERT ON departments TO PUBLIC;

-- Commonly used for:
-- - Read-only reference tables
-- - Public views
-- - Shared utility objects
```

### REVOKE Privileges
```sql
-- Revoke object privileges
REVOKE SELECT ON employees FROM user1;
-- user1 can no longer query employees

-- Revoke multiple privileges
REVOKE INSERT, UPDATE, DELETE ON employees FROM user1;

-- Revoke all privileges
REVOKE ALL ON employees FROM user1;

-- Revoke from PUBLIC
REVOKE SELECT ON employees FROM PUBLIC;

-- Revoke system privileges
REVOKE CREATE TABLE FROM user1;

-- CASCADE effects:
-- If user1 granted SELECT to user2 WITH GRANT OPTION
-- and user2 granted to user3...
REVOKE SELECT ON employees FROM user1;
-- Also revokes from user2 and user3 (CASCADE)
```

### Creating Roles
```sql
-- Create role to group privileges
CREATE ROLE developer_role;
-- Role created

-- Grant privileges to role
GRANT CREATE SESSION TO developer_role;
GRANT CREATE TABLE TO developer_role;
GRANT CREATE VIEW TO developer_role;
GRANT CREATE PROCEDURE TO developer_role;

-- Grant object privileges to role
GRANT SELECT, INSERT, UPDATE, DELETE ON employees TO developer_role;
GRANT SELECT ON departments TO developer_role;
GRANT SELECT ON jobs TO developer_role;

-- Grant role to users
GRANT developer_role TO user1, user2, user3;
-- All three users get all privileges in the role

-- Advantages of roles:
-- 1. Easier to manage (grant once to role, assign role to users)
-- 2. Consistent permissions across users
-- 3. Easy to modify (change role, affects all users)
```

### Pre-defined Roles
```sql
-- Oracle provides built-in roles

-- CONNECT role (basic connection privileges)
GRANT CONNECT TO user1;
-- Includes CREATE SESSION and other basic privileges

-- RESOURCE role (create schema objects)
GRANT RESOURCE TO user1;
-- Includes CREATE TABLE, CREATE SEQUENCE, etc.

-- DBA role (all system privileges)
GRANT DBA TO user1;
-- Full database administrative privileges (use carefully!)

-- Example: Grant standard developer access
GRANT CONNECT, RESOURCE TO new_developer;
```

### Enable/Disable Roles
```sql
-- User can enable/disable their roles
-- Set default role for user
ALTER USER user1 DEFAULT ROLE developer_role;

-- User enables role in session
SET ROLE developer_role;

-- User disables all roles
SET ROLE NONE;

-- User enables all granted roles
SET ROLE ALL;

-- User enables specific roles
SET ROLE developer_role, reporting_role;
```

### Revoke Roles
```sql
-- Revoke role from user
REVOKE developer_role FROM user1;

-- Drop role entirely
DROP ROLE developer_role;
-- Removes role from all users who had it
```

### View Privileges and Roles
```sql
-- View system privileges granted to you
SELECT * FROM user_sys_privs;

-- Result:
USERNAME | PRIVILEGE      | ADMIN_OPTION
HR       | CREATE SESSION | NO
HR       | CREATE TABLE   | NO
HR       | CREATE VIEW    | YES

-- View object privileges granted to you
SELECT * FROM user_tab_privs;

-- Result:
GRANTOR | TABLE_NAME | PRIVILEGE | GRANTABLE
SYSTEM  | EMPLOYEES  | SELECT    | NO
SYSTEM  | EMPLOYEES  | INSERT    | YES

-- View roles granted to you
SELECT * FROM user_role_privs;

-- Result:
USERNAME | GRANTED_ROLE    | DEFAULT_ROLE
HR       | CONNECT         | YES
HR       | RESOURCE        | YES
HR       | DEVELOPER_ROLE  | YES

-- View privileges on specific table
SELECT grantee, privilege, grantable
FROM dba_tab_privs
WHERE table_name = 'EMPLOYEES'
  AND owner = 'HR';

-- View role privileges (what privileges are in a role)
SELECT * FROM role_sys_privs WHERE role = 'DEVELOPER_ROLE';

SELECT * FROM role_tab_privs WHERE role = 'DEVELOPER_ROLE';
```

### Practical Security Example
```sql
-- Create application roles
CREATE ROLE app_read_only;
CREATE ROLE app_data_entry;
CREATE ROLE app_manager;

-- Read-only role: SELECT only
GRANT SELECT ON employees TO app_read_only;
GRANT SELECT ON departments TO app_read_only;
GRANT SELECT ON jobs TO app_read_only;

-- Data entry role: SELECT, INSERT, UPDATE
GRANT SELECT, INSERT, UPDATE ON employees TO app_data_entry;
GRANT SELECT ON departments TO app_data_entry;
GRANT SELECT ON jobs TO app_data_entry;

-- Manager role: Full access
GRANT ALL ON employees TO app_manager;
GRANT ALL ON departments TO app_manager;
GRANT ALL ON jobs TO app_manager;

-- Assign roles to users
GRANT app_read_only TO reporting_user;
GRANT app_data_entry TO clerk_user;
GRANT app_manager TO manager_user;

-- Later, easily modify access
-- Add new table access to all data entry users:
GRANT SELECT, INSERT, UPDATE ON new_table TO app_data_entry;
-- All users with app_data_entry role automatically get access
```

***

## Module 20: Manipulating Data Using Advanced Queries

### Multi-Table INSERT Examples
```sql
-- Example table structures
CREATE TABLE sales_history (
    employee_id NUMBER,
    sale_date DATE,
    amount NUMBER
);

CREATE TABLE top_sales (
    employee_id NUMBER,
    amount NUMBER,
    recorded_date DATE
);

CREATE TABLE regular_sales (
    employee_id NUMBER,
    amount NUMBER
);

-- Conditional multi-table insert
INSERT ALL
    WHEN amount > 10000 THEN
        INTO top_sales VALUES (employee_id, amount, SYSDATE)
    WHEN amount BETWEEN 5000 AND 10000 THEN
        INTO regular_sales VALUES (employee_id, amount)
SELECT employee_id, amount
FROM daily_sales
WHERE sale_date = TRUNC(SYSDATE);

-- Pivoting data example
INSERT ALL
    INTO monthly_sales VALUES (emp_id, 'JAN', jan_amt)
    INTO monthly_sales VALUES (emp_id, 'FEB', feb_amt)
    INTO monthly_sales VALUES (emp_id, 'MAR', mar_amt)
    INTO monthly_sales VALUES (emp_id, 'APR', apr_amt)
SELECT employee_id AS emp_id,
       jan_sales AS jan_amt,
       feb_sales AS feb_amt,
       mar_sales AS mar_amt,
       apr_sales AS apr_amt
FROM quarterly_sales_2025;

-- Transform: 1 row × 4 columns → 4 rows × 1 column per employee
```

### MERGE Statement Advanced
```sql
-- Complete MERGE example
MERGE INTO employees_target t
USING (SELECT employee_id, first_name, last_name, salary, department_id
       FROM employees_source) s
ON (t.employee_id = s.employee_id)
WHEN MATCHED THEN
    UPDATE SET 
        t.first_name = s.first_name,
        t.last_name = s.last_name,
        t.salary = s.salary,
        t.department_id = s.department_id,
        t.last_updated = SYSDATE
    DELETE WHERE t.salary < 3000  -- Delete after update if condition met
WHEN NOT MATCHED THEN
    INSERT (employee_id, first_name, last_name, salary, 
            department_id, created_date)
    VALUES (s.employee_id, s.first_name, s.last_name, s.salary,
            s.department_id, SYSDATE);

-- MERGE with complex conditions
MERGE INTO inventory i
USING (SELECT product_id, quantity_sold
       FROM daily_sales
       WHERE sale_date = TRUNC(SYSDATE)) s
ON (i.product_id = s.product_id)
WHEN MATCHED THEN
    UPDATE SET 
        i.quantity_available = i.quantity_available - s.quantity_sold,
        i.last_sale_date = SYSDATE
    WHERE i.quantity_available >= s.quantity_sold
WHEN NOT MATCHED THEN
    INSERT (product_id, quantity_available, last_sale_date)
    VALUES (s.product_id, 0, SYSDATE);
```

### Advanced UPDATE Techniques
```sql
-- Update with complex subquery
UPDATE employees e
SET salary = 
    CASE 
        WHEN (SELECT COUNT(*) FROM employees 
              WHERE manager_id = e.employee_id) > 5 
        THEN salary * 1.15  -- 15% raise for managers with >5 reports
        WHEN MONTHS_BETWEEN(SYSDATE, hire_date) > 120
        THEN salary * 1.10  -- 10% raise for 10+ years
        ELSE salary * 1.05  -- 5% standard raise
    END
WHERE department_id IN (SELECT department_id 
                        FROM departments 
                        WHERE location_id = 1700);

-- Update using inline view (updateable join view)
UPDATE (SELECT e.salary AS emp_salary,
               e.commission_pct AS emp_comm,
               j.max_salary AS job_max
        FROM employees e
        JOIN jobs j ON e.job_id = j.job_id
        WHERE e.salary > j.max_salary)
SET emp_salary = job_max,
    emp_comm = 0;

-- Corrects salaries that exceed job maximum
```

### Advanced DELETE Techniques
```sql
-- Delete with complex correlated subquery
DELETE FROM employees e
WHERE NOT EXISTS (
    SELECT 1 FROM job_history jh
    WHERE jh.employee_id = e.employee_id
)
AND NOT EXISTS (
    SELECT 1 FROM employees
    WHERE manager_id = e.employee_id
)
AND MONTHS_BETWEEN(SYSDATE, e.hire_date) < 6;

-- Deletes recent hires with no history and no subordinates

-- Delete using ranking
DELETE FROM employees
WHERE employee_id IN (
    SELECT employee_id
    FROM (SELECT employee_id,
                 ROW_NUMBER() OVER (PARTITION BY department_id 
                                    ORDER BY salary DESC) AS rn
          FROM employees)
    WHERE rn > 10
);

-- Keeps only top 10 earners per department
```

### Hierarchical Queries Advanced
```sql
-- Complete organizational structure
SELECT 
    LEVEL AS org_level,
    LPAD(' ', 2 * (LEVEL - 1)) || first_name || ' ' || last_name AS employee,
    job_id,
    salary,
    SYS_CONNECT_BY_PATH(first_name, ' -> ') AS path
FROM employees
START WITH manager_id IS NULL
CONNECT BY PRIOR employee_id = manager_id
ORDER SIBLINGS BY last_name;

-- Result shows hierarchical tree with path

-- Find all subordinates of specific manager
SELECT first_name, last_name, LEVEL
FROM employees
START WITH employee_id = 100  -- Steven King
CONNECT BY PRIOR employee_id = manager_id;

-- Find path from employee to CEO
SELECT LPAD(' ', 2 * (LEVEL - 1)) || first_name AS hierarchy
FROM employees
START WITH employee_id = 107  -- Diana Lorentz
CONNECT BY employee_id = PRIOR manager_id;

-- Result shows Diana -> her manager -> their manager -> CEO
```

### Analytic Functions (Window Functions)
```sql
-- ROW_NUMBER - assigns unique sequential number
SELECT employee_id,
       first_name,
       department_id,
       salary,
       ROW_NUMBER() OVER (PARTITION BY department_id 
                          ORDER BY salary DESC) AS salary_rank
FROM employees;

-- Result:
EMPLOYEE_ID | FIRST_NAME | DEPARTMENT_ID | SALARY | SALARY_RANK
100        | Steven     | 90           | 24000  | 1
101        | Neena      | 90           | 17000  | 2
103        | Alexander  | 60           | 9000   | 1
104        | Bruce      | 60           | 6000   | 2

-- RANK - same values get same rank, gaps in sequence
SELECT first_name,
       salary,
       RANK() OVER (ORDER BY salary DESC) AS rank,
       DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;

-- Result:
FIRST_NAME | SALARY | RANK | DENSE_RANK
Steven     | 24000  | 1    | 1
Neena      | 17000  | 2    | 2
Lex        | 17000  | 2    | 2  (same rank)
Nancy      | 12008  | 4    | 3  (RANK skips 3, DENSE_RANK doesn't)

-- LAG and LEAD - access previous/next rows
SELECT employee_id,
       first_name,
       hire_date,
       salary,
       LAG(salary) OVER (ORDER BY hire_date) AS prev_hire_salary,
       LEAD(salary) OVER (ORDER BY hire_date) AS next_hire_salary
FROM employees;

-- FIRST_VALUE and LAST_VALUE
SELECT employee_id,
       first_name,
       department_id,
       salary,
       FIRST_VALUE(salary) OVER (PARTITION BY department_id 
                                 ORDER BY salary DESC) AS highest_in_dept,
       LAST_VALUE(salary) OVER (PARTITION BY

 ## Module 20: Manipulating Data Using Advanced Queries (Continued)

### Analytic Functions (Continued)
```sql
-- FIRST_VALUE and LAST_VALUE (continued)
SELECT employee_id,
       first_name,
       department_id,
       salary,
       FIRST_VALUE(salary) OVER (PARTITION BY department_id 
                                 ORDER BY salary DESC) AS highest_in_dept,
       LAST_VALUE(salary) OVER (PARTITION BY department_id 
                                ORDER BY salary DESC
                                ROWS BETWEEN UNBOUNDED PRECEDING 
                                AND UNBOUNDED FOLLOWING) AS lowest_in_dept
FROM employees;

-- Result:
EMPLOYEE_ID | FIRST_NAME | DEPARTMENT_ID | SALARY | HIGHEST_IN_DEPT | LOWEST_IN_DEPT
100        | Steven     | 90           | 24000  | 24000          | 17000
101        | Neena      | 90           | 17000  | 24000          | 17000
103        | Alexander  | 60           | 9000   | 9000           | 4200

-- Running totals with SUM analytic
SELECT employee_id,
       first_name,
       hire_date,
       salary,
       SUM(salary) OVER (ORDER BY hire_date 
                         ROWS BETWEEN UNBOUNDED PRECEDING 
                         AND CURRENT ROW) AS running_total
FROM employees
ORDER BY hire_date;

-- Result shows cumulative salary total as employees are hired

-- Moving average
SELECT employee_id,
       first_name,
       hire_date,
       salary,
       AVG(salary) OVER (ORDER BY hire_date
                         ROWS BETWEEN 2 PRECEDING 
                         AND CURRENT ROW) AS moving_avg_3
FROM employees
ORDER BY hire_date;

-- Calculates average of current row + 2 previous rows
```

### NTILE - Divide into Buckets
```sql
-- Divide employees into salary quartiles
SELECT employee_id,
       first_name,
       salary,
       NTILE(4) OVER (ORDER BY salary) AS salary_quartile
FROM employees;

-- Result:
EMPLOYEE_ID | FIRST_NAME | SALARY | SALARY_QUARTILE
107        | Diana      | 4200   | 1  (bottom 25%)
110        | John       | 4400   | 1
103        | Alexander  | 9000   | 2
108        | Nancy      | 12008  | 3
101        | Neena      | 17000  | 4
100        | Steven     | 24000  | 4  (top 25%)

-- Divide by department
SELECT first_name,
       department_id,
       salary,
       NTILE(3) OVER (PARTITION BY department_id 
                      ORDER BY salary) AS dept_tercile
FROM employees;

-- Each department divided into 3 salary groups
```

### PIVOT - Rows to Columns
```sql
-- Transform row data into columns
SELECT *
FROM (SELECT department_id, job_id, salary
      FROM employees)
PIVOT (SUM(salary)
       FOR job_id IN ('IT_PROG' AS IT,
                      'SA_REP' AS SALES,
                      'ST_CLERK' AS CLERK))
ORDER BY department_id;

-- Before:
DEPT_ID | JOB_ID   | SALARY
60      | IT_PROG  | 9000
60      | IT_PROG  | 6000
80      | SA_REP   | 11000

-- After:
DEPT_ID | IT    | SALES | CLERK
60      | 15000 | NULL  | NULL
80      | NULL  | 29000 | NULL

-- Multiple aggregates
SELECT *
FROM (SELECT department_id, job_id, salary
      FROM employees)
PIVOT (COUNT(*) AS emp_count,
       SUM(salary) AS total_sal,
       AVG(salary) AS avg_sal
       FOR job_id IN ('IT_PROG', 'SA_REP'))
ORDER BY department_id;
```

### UNPIVOT - Columns to Rows
```sql
-- Transform columns into rows
-- Assume table: quarterly_sales (emp_id, Q1, Q2, Q3, Q4)
SELECT *
FROM quarterly_sales
UNPIVOT (sales FOR quarter IN (Q1, Q2, Q3, Q4));

-- Before:
EMP_ID | Q1   | Q2   | Q3   | Q4
100    | 5000 | 6000 | 5500 | 7000

-- After:
EMP_ID | QUARTER | SALES
100    | Q1      | 5000
100    | Q2      | 6000
100    | Q3      | 5500
100    | Q4      | 7000

-- UNPIVOT multiple columns
SELECT *
FROM sales_data
UNPIVOT ((sales, commission) 
         FOR quarter IN ((q1_sales, q1_comm) AS 'Q1',
                        (q2_sales, q2_comm) AS 'Q2'));
```

***

## Module 21: Managing Data in Different Time Zones

### DATE vs TIMESTAMP
```sql
-- DATE: stores date and time (second precision)
CREATE TABLE events_date (
    event_id NUMBER,
    event_time DATE
);

INSERT INTO events_date VALUES (1, SYSDATE);

SELECT event_time FROM events_date;
-- Result: 21-NOV-25 (shows date, time hidden but stored)

-- TIMESTAMP: stores date and time with fractional seconds
CREATE TABLE events_timestamp (
    event_id NUMBER,
    event_time TIMESTAMP
);

INSERT INTO events_timestamp VALUES (1, SYSTIMESTAMP);

SELECT event_time FROM events_timestamp;
-- Result: 21-NOV-25 11.30.45.123456 AM
-- Shows microseconds
```

### TIMESTAMP WITH TIME ZONE
```sql
-- Store timestamp with time zone information
CREATE TABLE global_events (
    event_id NUMBER,
    event_time TIMESTAMP WITH TIME ZONE
);

INSERT INTO global_events VALUES 
(1, TIMESTAMP '2025-11-21 14:30:00 -05:00');  -- EST

INSERT INTO global_events VALUES 
(2, TIMESTAMP '2025-11-21 11:30:00 -08:00');  -- PST

SELECT event_time FROM global_events;

-- Result:
EVENT_TIME
21-NOV-25 02.30.00.000000 PM -05:00
21-NOV-25 11.30.00.000000 AM -08:00

-- Same moment in time, different time zones
-- Converts for display based on session time zone

-- Check session time zone
SELECT SESSIONTIMEZONE FROM dual;
-- Result: -05:00 (EST)

-- Change session time zone
ALTER SESSION SET TIME_ZONE = '-08:00';  -- PST

-- Now same query shows in PST
SELECT event_time FROM global_events;
-- Result shows times converted to PST
```

### TIMESTAMP WITH LOCAL TIME ZONE
```sql
-- Store in database time zone, display in session time zone
CREATE TABLE meetings (
    meeting_id NUMBER,
    meeting_time TIMESTAMP WITH LOCAL TIME ZONE
);

-- User in EST inserts:
INSERT INTO meetings VALUES 
(1, TIMESTAMP '2025-11-21 14:00:00');

-- Stored in database time zone (UTC)
-- Displayed in user's session time zone

-- User in EST views:
ALTER SESSION SET TIME_ZONE = '-05:00';
SELECT meeting_time FROM meetings;
-- Result: 21-NOV-25 02.00.00.000000 PM (EST)

-- User in PST views same data:
ALTER SESSION SET TIME_ZONE = '-08:00';
SELECT meeting_time FROM meetings;
-- Result: 21-NOV-25 11.00.00.000000 AM (PST)

-- Same stored time, different display based on session
```

### Time Zone Conversions
```sql
-- Convert between time zones using AT TIME ZONE
SELECT 
    SYSTIMESTAMP AS server_time,
    SYSTIMESTAMP AT TIME ZONE 'US/Pacific' AS pacific_time,
    SYSTIMESTAMP AT TIME ZONE 'US/Eastern' AS eastern_time,
    SYSTIMESTAMP AT TIME ZONE 'Europe/London' AS london_time,
    SYSTIMESTAMP AT TIME ZONE 'Asia/Tokyo' AS tokyo_time
FROM dual;

-- Result shows same moment in different time zones

-- Convert stored timestamp
SELECT event_time,
       event_time AT TIME ZONE 'US/Pacific' AS pacific,
       event_time AT TIME ZONE 'UTC' AS utc
FROM global_events;

-- FROM_TZ - create TIMESTAMP WITH TIME ZONE
SELECT FROM_TZ(TIMESTAMP '2025-11-21 14:30:00', '-05:00') AS est_time
FROM dual;

-- Result: 21-NOV-25 02.30.00.000000 PM -05:00
```

### Time Zone Functions
```sql
-- CURRENT_DATE - session date
SELECT CURRENT_DATE FROM dual;
-- Result: 21-NOV-25 (in session time zone)

-- CURRENT_TIMESTAMP - session timestamp with time zone
SELECT CURRENT_TIMESTAMP FROM dual;
-- Result: 21-NOV-25 11.30.00.000000 AM -05:00

-- SYSDATE - database server date
SELECT SYSDATE FROM dual;
-- Result: 21-NOV-25

-- SYSTIMESTAMP - database server timestamp
SELECT SYSTIMESTAMP FROM dual;
-- Result: 21-NOV-25 11.30.00.000000 AM -05:00

-- LOCALTIMESTAMP - session timestamp without time zone
SELECT LOCALTIMESTAMP FROM dual;
-- Result: 21-NOV-25 11.30.00.000000 AM

-- Difference between CURRENT_TIMESTAMP and SYSTIMESTAMP
ALTER SESSION SET TIME_ZONE = '-08:00';
SELECT CURRENT_TIMESTAMP AS session_time,
       SYSTIMESTAMP AS server_time
FROM dual;
-- session_time shows PST, server_time shows server time zone
```

### EXTRACT Function for Date/Time Components
```sql
-- Extract components from date/timestamp
SELECT hire_date,
       EXTRACT(YEAR FROM hire_date) AS year,
       EXTRACT(MONTH FROM hire_date) AS month,
       EXTRACT(DAY FROM hire_date) AS day
FROM employees;

-- Result:
HIRE_DATE  | YEAR | MONTH | DAY
17-JUN-03  | 2003 | 6     | 17
21-SEP-05  | 2005 | 9     | 21

-- Extract from TIMESTAMP WITH TIME ZONE
SELECT event_time,
       EXTRACT(HOUR FROM event_time) AS hour,
       EXTRACT(MINUTE FROM event_time) AS minute,
       EXTRACT(SECOND FROM event_time) AS second,
       EXTRACT(TIMEZONE_HOUR FROM event_time) AS tz_hour,
       EXTRACT(TIMEZONE_MINUTE FROM event_time) AS tz_min
FROM global_events;

-- Result:
EVENT_TIME                          | HOUR | MINUTE | SECOND | TZ_HOUR | TZ_MIN
21-NOV-25 02.30.00.000000 PM -05:00 | 14   | 30     | 0      | -5      | 0
```

### TZ_OFFSET Function
```sql
-- Get time zone offset for region
SELECT TZ_OFFSET('US/Eastern') AS est_offset,
       TZ_OFFSET('US/Pacific') AS pst_offset,
       TZ_OFFSET('Europe/London') AS london_offset,
       TZ_OFFSET('Asia/Tokyo') AS tokyo_offset
FROM dual;

-- Result:
EST_OFFSET | PST_OFFSET | LONDON_OFFSET | TOKYO_OFFSET
-05:00     | -08:00     | +00:00        | +09:00

-- Accounts for daylight saving time automatically
```

### DBTIMEZONE and SESSIONTIMEZONE
```sql
-- Database time zone
SELECT DBTIMEZONE FROM dual;
-- Result: +00:00 (typically UTC)

-- Session time zone
SELECT SESSIONTIMEZONE FROM dual;
-- Result: -05:00 (current session setting)

-- Set session time zone
ALTER SESSION SET TIME_ZONE = 'US/Pacific';
SELECT SESSIONTIMEZONE FROM dual;
-- Result: -08:00

-- Set using offset
ALTER SESSION SET TIME_ZONE = '+05:30';  -- India

-- Set to database time zone
ALTER SESSION SET TIME_ZONE = DBTIMEZONE;

-- Set to local operating system time zone
ALTER SESSION SET TIME_ZONE = LOCAL;
```

### Practical Time Zone Example
```sql
-- Global meeting scheduler
CREATE TABLE global_meetings (
    meeting_id NUMBER PRIMARY KEY,
    meeting_name VARCHAR2(100),
    start_time TIMESTAMP WITH TIME ZONE,
    end_time TIMESTAMP WITH TIME ZONE,
    organizer_tz VARCHAR2(50)
);

-- Schedule meeting (organizer in EST)
INSERT INTO global_meetings VALUES (
    1,
    'Quarterly Review',
    TIMESTAMP '2025-12-01 10:00:00 -05:00',  -- 10 AM EST
    TIMESTAMP '2025-12-01 11:30:00 -05:00',  -- 11:30 AM EST
    'US/Eastern'
);

-- View in different time zones
-- EST user:
ALTER SESSION SET TIME_ZONE = 'US/Eastern';
SELECT meeting_name, 
       TO_CHAR(start_time, 'DD-MON-YYYY HH:MI AM TZR') AS my_time
FROM global_meetings;
-- Result: 01-DEC-2025 10:00 AM EST

-- PST user:
ALTER SESSION SET TIME_ZONE = 'US/Pacific';
SELECT meeting_name,
       TO_CHAR(start_time, 'DD-MON-YYYY HH:MI AM TZR') AS my_time
FROM global_meetings;
-- Result: 01-DEC-2025 07:00 AM PST

-- London user:
ALTER SESSION SET TIME_ZONE = 'Europe/London';
SELECT meeting_name,
       TO_CHAR(start_time, 'DD-MON-YYYY HH:MI AM TZR') AS my_time
FROM global_meetings;
-- Result: 01-DEC-2025 03:00 PM GMT
```

***

## Module 22: Conclusion

### Course Summary

**Key SQL Skills Acquired:**

1. **Data Retrieval**
   - SELECT statements with filtering and sorting
   - Joining multiple tables
   - Using subqueries and set operators
   - Advanced querying techniques

2. **Data Manipulation**
   - INSERT, UPDATE, DELETE, MERGE statements
   - Transaction control (COMMIT, ROLLBACK, SAVEPOINT)
   - Multi-table inserts
   - Subqueries in DML

3. **Schema Objects**
   - Creating and managing tables
   - Implementing constraints
   - Working with views, sequences, indexes, synonyms
   - Data dictionary usage

4. **Functions**
   - Single-row functions (character, number, date)
   - Conversion functions
   - Aggregate functions
   - Analytic functions

5. **Advanced Topics**
   - Hierarchical queries
   - Time zone management
   - User access control
   - Complex data transformations

### Best Practices Learned

```sql
-- 1. Always use explicit joins (not implicit)
-- Good:
SELECT e.first_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id;

-- Avoid:
SELECT e.first_name, d.department_name
FROM employees e, departments d
WHERE e.department_id = d.department_id;

-- 2. Use table aliases for readability
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id;

-- 3. Always specify column list

## Module 22: Conclusion (Continued)

### Best Practices Learned (Continued)

```sql
-- 3. Always specify column list in INSERT
-- Good:
INSERT INTO employees (employee_id, first_name, last_name, email, hire_date, job_id)
VALUES (207, 'John', 'Smith', 'JSMITH', SYSDATE, 'IT_PROG');

-- Avoid:
INSERT INTO employees VALUES (207, 'John', 'Smith', ...);  -- Fragile if table structure changes

-- 4. Use meaningful constraint names
-- Good:
ALTER TABLE employees
ADD CONSTRAINT emp_salary_positive_ck CHECK (salary > 0);

-- Avoid:
ALTER TABLE employees
ADD CHECK (salary > 0);  -- Oracle generates cryptic name like SYS_C007428

-- 5. Use bind variables for performance (in applications)
-- Good (in application code):
SELECT * FROM employees WHERE employee_id = :emp_id;

-- Avoid:
SELECT * FROM employees WHERE employee_id = 100;  -- Hard-coded value

-- 6. Use EXISTS instead of COUNT(*) for existence checks
-- Good:
SELECT department_name
FROM departments d
WHERE EXISTS (SELECT 1 FROM employees e WHERE e.department_id = d.department_id);

-- Less efficient:
SELECT department_name
FROM departments d
WHERE (SELECT COUNT(*) FROM employees e WHERE e.department_id = d.department_id) > 0;

-- 7. Always handle NULLs explicitly
-- Good:
SELECT first_name, NVL(commission_pct, 0) AS commission
FROM employees;

-- Be aware:
SELECT first_name, commission_pct
FROM employees
WHERE commission_pct = NULL;  -- Returns NO rows (should use IS NULL)

-- 8. Use transactions appropriately
BEGIN
    INSERT INTO orders (order_id, customer_id) VALUES (1001, 50);
    INSERT INTO order_items (order_id, product_id) VALUES (1001, 200);
    COMMIT;  -- Save both or neither
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;  -- Undo if any error
        RAISE;
END;

-- 9. Create indexes on foreign key columns
CREATE INDEX emp_dept_fk_idx ON employees(department_id);
-- Improves JOIN performance and DELETE from parent table

-- 10. Use views for security and simplification
CREATE VIEW emp_public AS
SELECT employee_id, first_name, last_name, job_id, department_id
FROM employees;
-- Hide sensitive columns like salary

GRANT SELECT ON emp_public TO application_users;
-- Don't grant access to base table
```

### Common Mistakes to Avoid

```sql
-- MISTAKE 1: Forgetting WHERE clause in UPDATE/DELETE
UPDATE employees SET salary = 10000;  -- Updates ALL rows!
-- Should be:
UPDATE employees SET salary = 10000 WHERE employee_id = 107;

-- MISTAKE 2: Using OR with NULL
SELECT * FROM employees
WHERE commission_pct = 0 OR commission_pct = NULL;  -- Wrong! Returns nothing for NULL
-- Should be:
SELECT * FROM employees
WHERE commission_pct = 0 OR commission_pct IS NULL;

-- MISTAKE 3: Not understanding operator precedence
SELECT * FROM employees
WHERE department_id = 60 OR department_id = 90 AND salary > 10000;
-- AND evaluated first! Gets dept 60 (all) OR dept 90 with salary > 10000
-- Should use parentheses:
SELECT * FROM employees
WHERE (department_id = 60 OR department_id = 90) AND salary > 10000;

-- MISTAKE 4: Cartesian product (missing join condition)
SELECT e.first_name, d.department_name
FROM employees e, departments d;
-- Returns 20 × 27 = 540 rows! Missing WHERE e.department_id = d.department_id

-- MISTAKE 5: Using SELECT * in production code
SELECT * FROM employees;  -- Avoid: gets all columns, inefficient, breaks if structure changes
-- Should specify needed columns:
SELECT employee_id, first_name, last_name, salary FROM employees;

-- MISTAKE 6: Not using COMMIT after DML
INSERT INTO employees VALUES (...);
-- Session disconnects before COMMIT
-- Data is lost!
-- Always:
INSERT INTO employees VALUES (...);
COMMIT;

-- MISTAKE 7: Incorrect date comparisons
SELECT * FROM employees
WHERE hire_date = '17-JUN-03';  -- Implicit conversion, may fail with different NLS settings
-- Should use:
SELECT * FROM employees
WHERE hire_date = TO_DATE('17-JUN-2003', 'DD-MON-YYYY');

-- MISTAKE 8: Dropping table without backup
DROP TABLE employees;  -- Permanent! (unless Flashback enabled)
-- Should:
CREATE TABLE employees_backup AS SELECT * FROM employees;
DROP TABLE employees;

-- MISTAKE 9: Using NOT IN with nullable columns
SELECT * FROM departments
WHERE department_id NOT IN (SELECT department_id FROM employees);
-- Returns NO rows if any employee has NULL department_id!
-- Should use:
SELECT * FROM departments
WHERE department_id NOT IN (SELECT department_id FROM employees 
                            WHERE department_id IS NOT NULL);
-- Or use NOT EXISTS:
SELECT * FROM departments d
WHERE NOT EXISTS (SELECT 1 FROM employees e WHERE e.department_id = d.department_id);

-- MISTAKE 10: Inefficient subqueries
SELECT * FROM employees
WHERE employee_id IN (SELECT employee_id FROM employees WHERE salary > 10000);
-- Unnecessary subquery!
-- Should be:
SELECT * FROM employees WHERE salary > 10000;
```

### SQL Performance Tips

```sql
-- TIP 1: Use indexes on frequently queried columns
CREATE INDEX emp_hire_date_idx ON employees(hire_date);
-- Speeds up queries like:
SELECT * FROM employees WHERE hire_date > TO_DATE('01-JAN-2020', 'DD-MON-YYYY');

-- TIP 2: Use EXPLAIN PLAN to analyze queries
EXPLAIN PLAN FOR
SELECT e.first_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE e.salary > 10000;

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
-- Shows execution plan, identifies full table scans, etc.

-- TIP 3: Limit result set size
SELECT first_name, last_name
FROM employees
WHERE ROWNUM <= 100;  -- Limit to 100 rows

-- Or use FETCH (Oracle 12c+):
SELECT first_name, last_name
FROM employees
FETCH FIRST 100 ROWS ONLY;

-- TIP 4: Use bulk operations for large data sets
-- Instead of row-by-row processing:
DECLARE
    CURSOR emp_cur IS SELECT employee_id, salary FROM employees;
BEGIN
    FOR emp_rec IN emp_cur LOOP
        UPDATE employees SET salary = salary * 1.1 
        WHERE employee_id = emp_rec.employee_id;
    END LOOP;
    COMMIT;
END;

-- Use set-based operation:
UPDATE employees SET salary = salary * 1.1;
COMMIT;
-- Much faster!

-- TIP 5: Use appropriate data types
-- Avoid VARCHAR2(4000) when VARCHAR2(50) is sufficient
-- Smaller data types = better performance and less storage

-- TIP 6: Analyze statistics regularly (DBA task)
EXEC DBMS_STATS.GATHER_TABLE_STATS('HR', 'EMPLOYEES');
-- Helps optimizer choose best execution plan
```

### Certification Path

```sql
-- After completing this course, you're prepared for:

-- Oracle Database SQL Certified Associate (1Z0-071)
-- Covers:
-- - Relational database concepts
-- - Retrieving data using SELECT
-- - Restricting and sorting data
-- - Using single-row functions
-- - Using conversion functions
-- - Reporting aggregated data
-- - Displaying data from multiple tables
-- - Using subqueries
-- - Using set operators
-- - Managing tables using DML
-- - Introduction to DDL
-- - Data dictionary views
-- - Creating sequences, synonyms, indexes, and views
-- - Managing schema objects
-- - Controlling user access
-- - Managing data in different time zones

-- Next steps:
-- 1. Oracle Database 19c: PL/SQL Workshop
-- 2. Oracle Database PL/SQL Certified Professional
-- 3. Advanced SQL Tuning
```

### Real-World Application Examples

```sql
-- EXAMPLE 1: Employee Management System
-- Find employees due for review (hired > 1 year ago, not reviewed in 6 months)
SELECT e.employee_id,
       e.first_name || ' ' || e.last_name AS employee_name,
       e.hire_date,
       MONTHS_BETWEEN(SYSDATE, e.hire_date) AS months_employed,
       NVL(TO_CHAR(e.last_review_date, 'DD-MON-YYYY'), 'Never') AS last_review
FROM employees e
WHERE MONTHS_BETWEEN(SYSDATE, e.hire_date) > 12
  AND (e.last_review_date IS NULL 
       OR MONTHS_BETWEEN(SYSDATE, e.last_review_date) > 6)
ORDER BY e.last_review_date NULLS FIRST;

-- EXAMPLE 2: Sales Analysis
-- Top 10 sales representatives by revenue this quarter
SELECT sales_rep_name, total_sales, rank
FROM (SELECT e.first_name || ' ' || e.last_name AS sales_rep_name,
             SUM(o.order_total) AS total_sales,
             RANK() OVER (ORDER BY SUM(o.order_total) DESC) AS rank
      FROM employees e
      JOIN orders o ON e.employee_id = o.sales_rep_id
      WHERE o.order_date >= TRUNC(SYSDATE, 'Q')  -- Current quarter
        AND e.job_id = 'SA_REP'
      GROUP BY e.employee_id, e.first_name, e.last_name)
WHERE rank <= 10;

-- EXAMPLE 3: Inventory Management
-- Products below reorder point
SELECT p.product_id,
       p.product_name,
       p.quantity_on_hand,
       p.reorder_point,
       p.reorder_point - p.quantity_on_hand AS qty_to_order,
       s.supplier_name,
       s.supplier_phone
FROM products p
JOIN suppliers s ON p.supplier_id = s.supplier_id
WHERE p.quantity_on_hand < p.reorder_point
  AND p.discontinued = 'N'
ORDER BY (p.reorder_point - p.quantity_on_hand) DESC;

-- EXAMPLE 4: Audit Trail
-- Track all salary changes
CREATE TABLE salary_audit (
    audit_id NUMBER PRIMARY KEY,
    employee_id NUMBER,
    old_salary NUMBER,
    new_salary NUMBER,
    change_date DATE,
    changed_by VARCHAR2(50)
);

-- Trigger to automatically log changes (PL/SQL - next course)
CREATE OR REPLACE TRIGGER salary_audit_trigger
AFTER UPDATE OF salary ON employees
FOR EACH ROW
BEGIN
    INSERT INTO salary_audit VALUES (
        salary_audit_seq.NEXTVAL,
        :NEW.employee_id,
        :OLD.salary,
        :NEW.salary,
        SYSDATE,
        USER
    );
END;

-- EXAMPLE 5: Department Budget Report
-- Compare actual vs budgeted salaries
SELECT d.department_name,
       d.budget,
       NVL(SUM(e.salary), 0) AS actual_salary,
       d.budget - NVL(SUM(e.salary), 0) AS remaining,
       ROUND((NVL(SUM(e.salary), 0) / d.budget) * 100, 2) AS pct_used
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_id, d.department_name, d.budget
HAVING NVL(SUM(e.salary), 0) > d.budget * 0.9  -- Over 90% of budget
ORDER BY pct_used DESC;
```

### Final Summary

**What You've Learned:**

✓ **Foundation**: Relational database concepts, SQL syntax, data types  
✓ **Querying**: SELECT, WHERE, ORDER BY, joins, subqueries, set operators  
✓ **Functions**: Single-row, aggregate, conversion, analytic functions  
✓ **DML**: INSERT, UPDATE, DELETE, MERGE, transaction control  
✓ **DDL**: CREATE, ALTER, DROP tables and other objects  
✓ **Objects**: Constraints, indexes, sequences, views, synonyms  
✓ **Security**: User access, privileges, roles  
✓ **Advanced**: Hierarchical queries, time zones, complex transformations  

**You can now:**
- Write efficient SQL queries to retrieve and analyze data
- Create and manage database schema objects
- Implement data integrity through constraints
- Control user access and security
- Manipulate data using DML statements
- Work with dates and time zones across global applications
- Optimize queries using indexes and proper techniques
- Use advanced features like analytic functions and pivoting

**Continue Your Learning:**
1. Practice with real datasets
2. Explore Oracle Database 19c: PL/SQL Workshop
3. Study for SQL certification
4. Learn database design and normalization
5. Explore performance tuning and optimization

**Remember:** SQL is a powerful tool. The more you practice, the more proficient you'll become!

***

## Additional Resources Summary

### Documentation
- Oracle Database SQL Language Reference
- Oracle Database Concepts Guide  
- Oracle Database Administrator's Guide
- Oracle SQL Developer User's Guide

### Practice Environments
- Oracle Live SQL (free online): live.oracle.com/sql
- Oracle Express Edition (free download)
- Oracle Cloud Free Tier

### Community
- Oracle Learning Library
- Oracle Technology Network forums
- Stack Overflow (tag: oracle)
- Oracle ACE Program

***

**End of Oracle Database 19c: SQL Workshop Complete Study Notes**

*Total Course Duration: 14 hours 41 minutes*  
*Modules Covered: 22*  
*Examples Provided: 500+*

These comprehensive notes with examples cover all topics from the course and provide practical, working SQL code that you can use as reference material for learning and daily work with Oracle databases.
Here are **expanded study notes with more examples** for the video "Managing Tables Using DML Statements in Oracle" from the Oracle Database 19c: SQL Workshop course:

***

# Managing Tables Using DML Statements in Oracle

## DML (Data Manipulation Language) Overview

- DML is used to manage data within Oracle tables.
- Main statements:
  - `INSERT`
  - `UPDATE`
  - `DELETE`
  - `MERGE` (covered in advanced modules)

***

## 1. INSERT Statement

- **Purpose:** Add new rows to a table.
- **Examples:**
  ```sql
  -- Insert single row
  INSERT INTO employees (employee_id, first_name, last_name)
  VALUES (1001, 'Alice', 'Smith');

  -- Insert multiple rows
  INSERT ALL
    INTO departments (department_id, department_name) VALUES (10, 'IT')
    INTO departments (department_id, department_name) VALUES (20, 'Finance')
  SELECT * FROM dual;
  ```

***

## 2. UPDATE Statement

- **Purpose:** Modify existing data in table rows.
- **Examples:**
  ```sql
  -- Update a salary for a specific employee
  UPDATE employees
  SET salary = salary * 1.10
  WHERE employee_id = 1001;

  -- Update all employees in a department
  UPDATE employees
  SET department_id = 20
  WHERE department_id = 10;
  ```

***

## 3. DELETE Statement

- **Purpose:** Remove rows from a table.
- **Examples:**
  ```sql
  -- Delete specific employee
  DELETE FROM employees
  WHERE employee_id = 1001;

  -- Delete all employees from a department
  DELETE FROM employees
  WHERE department_id = 10;
  ```

***

## 4. COMMIT and ROLLBACK

- **Purpose:** Manage transactions (save or undo changes).
- **Examples:**
  ```sql
  -- Save changes
  COMMIT;

  -- Undo changes
  ROLLBACK;
  ```

***

## 5. Transaction Best Practices

- Always use WHERE clause with `UPDATE` and `DELETE` to avoid unintentional changes.
- Validate data before inserting or updating.

***

## 6. SELECT Statement (for Review)

- **Purpose:** Retrieve and view data.
- **Examples:**
  ```sql
  SELECT employee_id, first_name, last_name FROM employees;
  ```

***

## 7. Advanced: MERGE Statement

- **Purpose:** Synchronize two tables (insert or update as needed).
- **Example:**
  ```sql
  MERGE INTO employees e
  USING new_employees n
  ON (e.employee_id = n.employee_id)
  WHEN MATCHED THEN
    UPDATE SET e.salary = n.salary
  WHEN NOT MATCHED THEN
    INSERT (employee_id, first_name, last_name, salary)
    VALUES (n.employee_id, n.first_name, n.last_name, n.salary);
  ```

***

## 8. Practice Case Study

- Example: You have new employees to add and some need salary adjustments.
  1. Use `INSERT` to add new employees.
  2. Use `UPDATE` to increase salaries by 5% for sales department.
  3. Use `DELETE` to remove employees who have left.
  4. Use `COMMIT` to save your work.

***

## Frequently Used Clauses

- `WHERE` — restricts rows affected
- `SET` — specifies columns and values for UPDATE
- `VALUES` — states inserted data

***

## Summary Checklist

- Use DML for data changes: `INSERT`, `UPDATE`, `DELETE`.
- Always pair with `COMMIT`/`ROLLBACK` for database integrity.
- Use WHERE clause thoughtfully.
- Test changes with SELECT before and after.

***

**End of notes**[1]

[1](https://mylearn.oracle.com/ou/course/oracle-database-19c-sql-workshop/105208/159030)