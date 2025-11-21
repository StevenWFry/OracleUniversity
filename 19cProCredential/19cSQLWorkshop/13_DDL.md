# Data Definition Language (DDL)

## Table of Contents
1. [Overview](#overview)
2. [CREATE TABLE](#create-table)
3. [ALTER TABLE](#alter-table)
4. [DROP TABLE](#drop-table)
5. [TRUNCATE TABLE](#truncate-table)
6. [CONSTRAINTS](#constraints)
7. [INDEXES (CREATE / DROP)](#indexes-create--drop)
8. [VIEWS (CREATE / DROP)](#views-create--drop)
9. [SEQUENCES](#sequences)
10. [SYNONYMS](#synonyms)
11. [USERS / SCHEMAS / TABLESPACES](#users--schemas--tablespaces)
12. [GRANT / REVOKE (DCL overlap)](#grant--revoke-dcl-overlap)
13. [Best Practices & Notes](#best-practices--notes)
14. [Quick Examples](#quick-examples)
15. [Data Types](#data-types)
16. [DateTime Data Types](#datetime-data-types)

---

### Overview
- Purpose: Define and modify database schema objects (tables, indexes, views, sequences, users).
- Common usage: Create schema objects before inserting data, evolve schema with ALTER, remove objects with DROP, and manage access with GRANT/REVOKE.
- Important: DDL statements often issue implicit commits (Oracle, MySQL) and cannot be rolled back after execution.

---

### CREATE TABLE
- Purpose: Define a new table and its columns, datatypes, constraints.
- Common usage: Create persistent storage for application data or staging tables.

Example (Oracle):
```sql
CREATE TABLE employees (
  emp_id      NUMBER PRIMARY KEY,
  first_name  VARCHAR2(50),
  last_name   VARCHAR2(50) NOT NULL,
  hire_date   DATE DEFAULT SYSDATE,
  salary      NUMBER(10,2),
  department_id NUMBER
);
```

Example (MySQL):
```sql
CREATE TABLE employees (
  emp_id INT AUTO_INCREMENT PRIMARY KEY,
  first_name VARCHAR(50),
  last_name VARCHAR(50) NOT NULL,
  email VARCHAR(100) UNIQUE,
  salary DECIMAL(10,2),
  department_id INT,
  hire_date DATE DEFAULT CURDATE(),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (department_id) REFERENCES departments(department_id),
  CHECK (salary >= 0)
);
```

Notes:
- Specify column datatypes and constraints at creation.
- Consider storage clauses and tablespace in production DBs.

---

### ALTER TABLE
- Purpose: Modify table structure (add/drop/modify columns, enable/disable constraints).
- Common usage: Add new column, change datatype, rename column, add/drop constraints.

Examples:
```sql
-- Add column
ALTER TABLE employees ADD (email VARCHAR2(100));

-- Modify datatype (Oracle syntax may require MOVE / column rebuild in some cases)
ALTER TABLE employees MODIFY (salary NUMBER(12,2));

-- Drop column (Oracle 12c+)
ALTER TABLE employees DROP COLUMN email;

-- Add constraint
ALTER TABLE employees ADD CONSTRAINT fk_dept FOREIGN KEY (department_id) REFERENCES departments(department_id);
```

MySQL-specific ALTER TABLE examples:
```sql
-- Add column
ALTER TABLE employees ADD COLUMN phone VARCHAR(15);

-- Modify column
ALTER TABLE employees MODIFY COLUMN salary DECIMAL(12,2);

-- Rename column (MySQL 8.0+)
ALTER TABLE employees RENAME COLUMN email TO employee_email;

-- Drop column
ALTER TABLE employees DROP COLUMN phone;

-- Add constraint
ALTER TABLE employees ADD CONSTRAINT emp_email_uk UNIQUE (employee_email);
ALTER TABLE employees ADD CONSTRAINT emp_salary_chk CHECK (salary >= 0);
```

Notes:
- Some ALTER operations are instantaneous; others require table rebuilds or exclusive locks.
- Test ALTER on copies for large tables.

---

### DROP TABLE
- Purpose: Remove table definition and data permanently.
- Common usage: Remove temporary or obsolete tables in development / cleanup scripts.

Example:
```sql
DROP TABLE employee_temp;
```

MySQL:
```sql
DROP TABLE employees;
-- Or with IF EXISTS to avoid error if table doesn't exist
DROP TABLE IF EXISTS employees;
```

Notes:
- DROP will implicitly commit in many DBs; data is lost unless backed up.
- Oracle supports PURGE to skip recycle bin: DROP TABLE t PURGE;

---

### TRUNCATE TABLE
- Purpose: Remove all rows quickly and reset storage/space metadata.
- Common usage: Fast clear of staging or temp tables between loads.

Example:
```sql
TRUNCATE TABLE employee_temp;
```

MySQL:
```sql
TRUNCATE TABLE employees;
-- Resets AUTO_INCREMENT counter to 1
```

Notes:
- TRUNCATE is DDL-like (implicit commit) and cannot be rolled back in many databases.
- TRUNCATE typically faster than DELETE without WHERE.

---

### CONSTRAINTS

- Purpose: Enforce data integrity by restricting what values can be inserted or updated in columns.
- Common usage: Define at CREATE TABLE, add/drop with ALTER TABLE, disable for bulk loads, re-enable afterward.

**Constraint Types:**

| Constraint | Description | Example |
|-----------|-------------|---------|
| NOT NULL | Column must always have a value | last_name VARCHAR2(50) NOT NULL |
| UNIQUE | All values in column must be unique (allows one NULL) | email VARCHAR2(100) UNIQUE |
| PRIMARY KEY | Unique identifier for each row (NOT NULL + UNIQUE) | emp_id NUMBER PRIMARY KEY |
| FOREIGN KEY | Reference a column in another table (referential integrity) | dept_id NUMBER REFERENCES departments(department_id) |
| CHECK | Restrict values to a specific condition | salary NUMBER CHECK (salary >= 0) |
| DEFAULT | Provide default value if none specified | hire_date DATE DEFAULT SYSDATE |

**NOT NULL Constraint:**

- Purpose: Ensure a column always contains a value.
- Common usage: Applied to required business fields (name, ID, email).

```sql
CREATE TABLE employees (
  emp_id NUMBER PRIMARY KEY,
  first_name VARCHAR2(50) NOT NULL,
  last_name VARCHAR2(50) NOT NULL,
  email VARCHAR2(100),
  salary NUMBER(10,2) NOT NULL
);

-- This will fail (NOT NULL violation):
INSERT INTO employees (emp_id, first_name, last_name, salary) VALUES (1, NULL, 'Smith', 50000);
```

**UNIQUE Constraint:**

- Purpose: Ensure all non-NULL values in a column are unique.
- Common usage: Email, username, employee badge number.

```sql
CREATE TABLE employees (
  emp_id NUMBER PRIMARY KEY,
  first_name VARCHAR2(50),
  last_name VARCHAR2(50),
  email VARCHAR2(100) UNIQUE,
  badge_number NUMBER UNIQUE
);

-- This will fail (UNIQUE violation):
INSERT INTO employees (emp_id, first_name, last_name, email) VALUES (1, 'John', 'Doe', 'john@example.com');
INSERT INTO employees (emp_id, first_name, last_name, email) VALUES (2, 'Jane', 'Smith', 'john@example.com');
```

Composite UNIQUE (multiple columns):
```sql
CREATE TABLE employee_projects (
  emp_id NUMBER,
  project_id NUMBER,
  UNIQUE (emp_id, project_id)  -- combination must be unique
);
```

**PRIMARY KEY Constraint:**

- Purpose: Uniquely identify each row in a table (NOT NULL + UNIQUE).
- Common usage: Employee ID, product code, order ID.
- Note: Each table should have exactly one PRIMARY KEY.

```sql
CREATE TABLE employees (
  emp_id NUMBER PRIMARY KEY,
  first_name VARCHAR2(50),
  last_name VARCHAR2(50)
);

-- Composite PRIMARY KEY
CREATE TABLE employee_skills (
  emp_id NUMBER,
  skill_id NUMBER,
  PRIMARY KEY (emp_id, skill_id)
);

-- This will fail (PRIMARY KEY violation):
INSERT INTO employees (emp_id, first_name, last_name) VALUES (101, 'John', 'Doe');
INSERT INTO employees (emp_id, first_name, last_name) VALUES (101, 'Jane', 'Smith');
```

**FOREIGN KEY Constraint:**

- Purpose: Enforce referential integrity by requiring values in one table to exist in another.
- Common usage: Link employees to departments, orders to customers.

```sql
CREATE TABLE departments (
  department_id NUMBER PRIMARY KEY,
  department_name VARCHAR2(100)
);

CREATE TABLE employees (
  emp_id NUMBER PRIMARY KEY,
  first_name VARCHAR2(50),
  last_name VARCHAR2(50),
  department_id NUMBER,
  FOREIGN KEY (department_id) REFERENCES departments(department_id)
);

-- This will fail (FOREIGN KEY violation - no department_id=999):
INSERT INTO employees (emp_id, first_name, last_name, department_id) VALUES (1, 'John', 'Doe', 999);

-- This will succeed (department_id=10 exists):
INSERT INTO departments (department_id, department_name) VALUES (10, 'IT');
INSERT INTO employees (emp_id, first_name, last_name, department_id) VALUES (1, 'John', 'Doe', 10);
```

FOREIGN KEY with ON DELETE CASCADE (automatically delete child rows):
```sql
CREATE TABLE employees (
  emp_id NUMBER PRIMARY KEY,
  first_name VARCHAR2(50),
  department_id NUMBER,
  FOREIGN KEY (department_id) REFERENCES departments(department_id) ON DELETE CASCADE
);

-- When department deleted, related employees are also deleted
DELETE FROM departments WHERE department_id = 10;
```

FOREIGN KEY with ON DELETE SET NULL (set child FK to NULL):
```sql
CREATE TABLE employees (
  emp_id NUMBER PRIMARY KEY,
  first_name VARCHAR2(50),
  department_id NUMBER,
  FOREIGN KEY (department_id) REFERENCES departments(department_id) ON DELETE SET NULL
);

-- When department deleted, employee.department_id becomes NULL
```

**CHECK Constraint:**

- Purpose: Restrict values to a specific condition or range.
- Common usage: Validate salary ranges, age limits, status values.

```sql
CREATE TABLE employees (
  emp_id NUMBER PRIMARY KEY,
  first_name VARCHAR2(50),
  salary NUMBER(10,2) CHECK (salary >= 0),
  age NUMBER CHECK (age >= 18 AND age <= 70),
  status VARCHAR2(20) CHECK (status IN ('Active', 'Inactive', 'Terminated'))
);

-- This will fail (CHECK constraint violation):
INSERT INTO employees (emp_id, first_name, salary, age, status) VALUES (1, 'John', -5000, 25, 'Active');

-- This will succeed:
INSERT INTO employees (emp_id, first_name, salary, age, status) VALUES (1, 'John', 50000, 25, 'Active');
```

Multiple CHECK constraints:
```sql
CREATE TABLE projects (
  project_id NUMBER PRIMARY KEY,
  project_name VARCHAR2(100),
  start_date DATE,
  end_date DATE,
  CHECK (end_date >= start_date),
  CHECK (project_name IS NOT NULL)
);
```

**DEFAULT Constraint:**

- Purpose: Provide a default value when no value is specified during INSERT.
- Common usage: Auto-populate timestamps, set default status.

```sql
CREATE TABLE employees (
  emp_id NUMBER PRIMARY KEY,
  first_name VARCHAR2(50),
  hire_date DATE DEFAULT SYSDATE,
  status VARCHAR2(20) DEFAULT 'Active',
  created_at TIMESTAMP DEFAULT SYSTIMESTAMP
);

-- Insert without specifying hire_date (uses DEFAULT SYSDATE)
INSERT INTO employees (emp_id, first_name) VALUES (1, 'John');

-- Query shows defaults applied:
SELECT * FROM employees;
-- emp_id=1, first_name='John', hire_date=TODAY, status='Active', created_at=NOW
```

**Managing Constraints with ALTER TABLE:**

Add constraint:
```sql
ALTER TABLE employees ADD CONSTRAINT emp_email_uk UNIQUE (email);

ALTER TABLE employees ADD CONSTRAINT emp_salary_chk CHECK (salary >= 0);

ALTER TABLE employees ADD CONSTRAINT emp_dept_fk FOREIGN KEY (department_id) REFERENCES departments(department_id);
```

Disable constraint (for bulk loads):
```sql
ALTER TABLE employees DISABLE CONSTRAINT emp_dept_fk;
-- Perform bulk INSERT/UPDATE without FK validation
ALTER TABLE employees ENABLE CONSTRAINT emp_dept_fk;
```

Drop constraint:
```sql
ALTER TABLE employees DROP CONSTRAINT emp_email_uk;
```

**Naming Constraints:**

Best practice — use descriptive names:
```sql
CREATE TABLE employees (
  emp_id NUMBER CONSTRAINT pk_emp PRIMARY KEY,
  email VARCHAR2(100) CONSTRAINT uk_emp_email UNIQUE,
  salary NUMBER CONSTRAINT chk_emp_salary CHECK (salary >= 0),
  department_id NUMBER CONSTRAINT fk_emp_dept REFERENCES departments(department_id),
  status VARCHAR2(20) CONSTRAINT chk_emp_status CHECK (status IN ('Active','Inactive','Terminated'))
);
```

**Viewing Constraints (Oracle):**

```sql
-- View all constraints on a table
SELECT constraint_name, constraint_type, column_name
FROM user_constraints
WHERE table_name = 'EMPLOYEES';

-- View constraint details
SELECT constraint_name, search_condition
FROM user_constraints
WHERE table_name = 'EMPLOYEES' AND constraint_type = 'C';  -- CHECK constraints
```

**Tips:**
- Always name constraints explicitly for easier management and debugging.
- Use NOT NULL for required columns; use UNIQUE for business identifiers.
- Define FOREIGN KEYs to maintain referential integrity across tables.
- Use CHECK constraints to validate business rules at the database level.
- Disable constraints during bulk loads, then re-enable for ongoing data entry.
- Consider CASCADE vs SET NULL vs RESTRICT for FOREIGN KEY delete rules based on business logic.

---

### INDEXES (CREATE / DROP)
- Purpose: Improve query performance by creating access paths.
- Common usage: Create indexes on frequently filtered or joined columns; avoid over-indexing.

Examples:
```sql
-- Create index
CREATE INDEX idx_emp_dept ON employees(department_id);

-- Create unique index
CREATE UNIQUE INDEX idx_emp_email ON employees(email);

-- Drop index
DROP INDEX idx_emp_dept;
```

MySQL:
```sql
CREATE INDEX idx_emp_lastname ON employees(last_name);

-- Create unique index
CREATE UNIQUE INDEX idx_emp_email ON employees(email);

-- Drop index
DROP INDEX idx_emp_lastname ON employees;
```

Notes:
- Indexes cost on DML (INSERT/UPDATE/DELETE) — tradeoff between read performance and write overhead.
- Consider bitmap indexes only for low-cardinality columns in data-warehousing contexts (Oracle-specific).

---

### VIEWS (CREATE / DROP)
- Purpose: Provide a named query (virtual table) to encapsulate logic and simplify access.
- Common usage: Present simplified APIs, mask columns, or aggregate frequently used joins.

Examples:
```sql
CREATE OR REPLACE VIEW v_emp_dept AS
SELECT e.emp_id, e.first_name, e.last_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id;

DROP VIEW v_emp_dept;
```

MySQL:
```sql
CREATE VIEW v_emp_dept AS
SELECT e.emp_id, e.first_name, e.last_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id;

-- Replace view
CREATE OR REPLACE VIEW v_emp_dept AS
SELECT e.emp_id, e.first_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id;

-- Drop view
DROP VIEW v_emp_dept;
```

Notes:
- Views do not store data (unless materialized view).
- Materialized views store results and must be refreshed.

---

### SEQUENCES
- Purpose: Generate unique numeric values (commonly used for surrogate keys in Oracle).
- Common usage: Use NEXTVAL in INSERT statements to populate primary keys.

Examples (Oracle):
```sql
CREATE SEQUENCE employees_seq START WITH 1000 INCREMENT BY 1 NOCACHE NOCYCLE;

INSERT INTO employees (emp_id, first_name) VALUES (employees_seq.NEXTVAL, 'Ana');
```

Notes:
- MySQL uses AUTO_INCREMENT instead of sequences.
- Configure CACHE for performance vs durability tradeoffs.

---

### SYNONYMS
- Purpose: Provide alternative names for objects (local or public synonyms).
- Common usage: Simplify cross-schema references or provide abstraction layer.

Example (Oracle):
```sql
CREATE SYNONYM emp FOR hr.employees;
SELECT * FROM emp;
```

Notes:
- Synonyms help hide schema qualifiers in application SQL.

---

### USERS / SCHEMAS / TABLESPACES
- Purpose: Create database users/schemas and manage storage allocation.
- Common usage: Provision new application schemas, assign tablespaces for physical storage.

Examples (Oracle):
```sql
CREATE USER app_user IDENTIFIED BY strong_pass DEFAULT TABLESPACE users;
GRANT CONNECT, RESOURCE TO app_user;
```

Notes:
- User/schema management and tablespace administration are DBA responsibilities.

---

### GRANT / REVOKE (DCL overlap)
- Purpose: Grant or revoke privileges on objects (SELECT, INSERT, UPDATE, DELETE, EXECUTE).
- Common usage: Assign least privilege required to application users or roles.

Examples:
```sql
GRANT SELECT, INSERT ON employees TO app_user;
REVOKE INSERT ON employees FROM app_user;
```

Notes:
- GRANT/REVOKE are often considered DCL but closely tied to DDL lifecycle and object security.

---

### Best Practices & Notes
- Backup or export schema before destructive DDL (DROP/TRUNCATE).
- Run DDL in maintenance windows for large objects to avoid prolonged locks.
- Use version-controlled DDL scripts and migrations (Flyway, Liquibase).
- Test schema changes on staging with realistic data volumes before production.
- Document implicit commit behavior: many DDLs cause commit and release savepoints.
- Use descriptive object names and consistent naming conventions.

---

### Quick Examples
```sql
-- Create table with constraints
CREATE TABLE departments (
  department_id NUMBER PRIMARY KEY,
  department_name VARCHAR2(100) UNIQUE
);

-- Add foreign key
ALTER TABLE employees ADD CONSTRAINT fk_emp_dept FOREIGN KEY (department_id) REFERENCES departments(department_id);

-- Create index for performance
CREATE INDEX idx_emp_lastname ON employees(last_name);

-- Create materialized view (Oracle)
CREATE MATERIALIZED VIEW mv_emp_sal AS
SELECT department_id, AVG(salary) AS avg_sal FROM employees GROUP BY department_id
REFRESH FAST ON COMMIT;
```

---

### Data Types

- Purpose: Define the kind of data a column can hold (numeric, text, date, binary, etc.).
- Common usage: Choose appropriate types to ensure data integrity, optimize storage, and support application logic.

**Oracle Data Types:**

| Data Type | Description | Example |
|-----------|-------------|---------|
| NUMBER(p,s) | Numeric with precision and scale | NUMBER(10,2) for currency |
| INTEGER | Whole numbers | INTEGER or NUMBER(38,0) |
| FLOAT | Floating-point numbers | FLOAT(126) |
| VARCHAR2(n) | Variable-length string | VARCHAR2(100) for names |
| CHAR(n) | Fixed-length string (padded) | CHAR(2) for state codes |
| DATE | Date and time | hire_date DATE |
| TIMESTAMP | Date/time with fractional seconds | TIMESTAMP(6) WITH TIME ZONE |
| CLOB | Large text objects (up to 4GB) | CLOB for documents |
| BLOB | Binary large objects | BLOB for images |
| RAW(n) | Fixed-length binary (up to 2000 bytes) | RAW(16) for UUIDs |
| LONG RAW | Variable-length binary (deprecated, up to 2GB) | LONG RAW for legacy systems |
| BFILE | Pointer to binary file in OS filesystem | BFILE for external media files |
| ROWID | Unique row identifier (pseudo-column) | ROWID for internal row reference |
| BOOLEAN | TRUE/FALSE (Oracle 23c+) | BOOLEAN |

**MySQL Data Types:**

| Data Type | Description | Example |
|-----------|-------------|---------|
| INT / INTEGER | Whole numbers | INT(11) |
| DECIMAL(p,s) | Fixed-point numeric | DECIMAL(10,2) for currency |
| FLOAT / DOUBLE | Floating-point numbers | FLOAT(10,2) |
| VARCHAR(n) | Variable-length string | VARCHAR(100) for names |
| CHAR(n) | Fixed-length string | CHAR(2) for state codes |
| TEXT | Large text (65KB) | TEXT for documents |
| LONGTEXT | Very large text (4GB) | LONGTEXT |
| DATE | Date only | DATE |
| DATETIME | Date and time | DATETIME |
| TIMESTAMP | Date/time with auto-update | TIMESTAMP DEFAULT CURRENT_TIMESTAMP |
| BLOB | Binary large object | BLOB for images |
| LONGBLOB | Very large binary (4GB) | LONGBLOB |
| BOOLEAN / TINYINT(1) | Boolean true/false | BOOLEAN |
| JSON | JSON document | JSON |

**PostgreSQL Data Types:**

| Data Type | Description | Example |
|-----------|-------------|---------|
| INTEGER / INT | Whole numbers | INTEGER |
| DECIMAL(p,s) / NUMERIC(p,s) | Fixed-point numeric | DECIMAL(10,2) |
| REAL / DOUBLE PRECISION | Floating-point | DOUBLE PRECISION |
| VARCHAR(n) / CHARACTER VARYING | Variable-length string | VARCHAR(100) |
| CHAR(n) / CHARACTER | Fixed-length string | CHAR(2) |
| TEXT | Variable-length text (unlimited) | TEXT |
| DATE | Date only | DATE |
| TIMESTAMP | Date and time without timezone | TIMESTAMP |
| TIMESTAMP WITH TIME ZONE | Date/time with timezone (TIMESTAMPTZ) | TIMESTAMP WITH TIME ZONE |
| BYTEA | Binary data | BYTEA |
| BOOLEAN | TRUE/FALSE | BOOLEAN |
| JSON / JSONB | JSON document | JSONB for better performance |
| UUID | Universally unique identifier | UUID |

**Oracle-Specific Data Types Detail:**

RAW(n):
- Fixed-length binary data type (max 2000 bytes).
- Used for storing raw binary data like UUIDs, cryptographic hashes.
```sql
CREATE TABLE audit_log (
  log_id NUMBER PRIMARY KEY,
  data_hash RAW(16),  -- 16-byte hash
  created_date DATE
);
```

LONG RAW (Deprecated):
- Variable-length binary data (up to 2GB).
- Legacy type; deprecated in favor of BLOB. Rarely used in new designs.
```sql
CREATE TABLE legacy_documents (
  doc_id NUMBER PRIMARY KEY,
  binary_content LONG RAW  -- Not recommended for new code
);
```

BFILE (Binary File):
- Pointer to a binary file stored in the operating system filesystem outside the database.
- Useful for managing large media files (images, videos) without storing them in the database.
- Read-only within SQL; requires DBMS_LOB package for manipulation.

```sql
CREATE TABLE media_library (
  media_id NUMBER PRIMARY KEY,
  media_title VARCHAR2(100),
  media_file BFILE  -- Pointer to OS file
);

-- Example: insert a BFILE reference
INSERT INTO media_library (media_id, media_title, media_file)
VALUES (1001, 'Company Video', BFILENAME('MEDIA_DIR', 'company_promo.mp4'));
```

ROWID (Pseudo-column):
- Unique identifier for each row in the table (physical location in the database).
- Not a stored column; automatically available for any table.
- Useful for unique row identification and performance tuning.
- Oracle internal representation; avoid relying on it in application logic.

```sql
SELECT ROWID, emp_id, first_name FROM employees;
-- Example output: ROWID might look like: AAAB1nAAEAAAAA3AAA

-- Use ROWID in WHERE (performance benefit)
DELETE FROM employees WHERE ROWID = (SELECT MAX(ROWID) FROM employees);
```

**Examples:**

Oracle:
```sql
CREATE TABLE employees (
  emp_id      NUMBER PRIMARY KEY,
  first_name  VARCHAR2(50),
  last_name   VARCHAR2(50) NOT NULL,
  hire_date   DATE,
  salary      NUMBER(10,2),
  commission  FLOAT,
  photo       BLOB,
  bio         CLOB,
  data_hash   RAW(32),       -- SHA-256 hash (32 bytes)
  media_file  BFILE          -- Reference to OS file
);
```

MySQL:
```sql
CREATE TABLE employees (
  emp_id      INT AUTO_INCREMENT PRIMARY KEY,
  first_name  VARCHAR(50),
  last_name   VARCHAR(50) NOT NULL,
  hire_date   DATE,
  salary      DECIMAL(10,2),
  commission  FLOAT(10,2),
  photo       BLOB,
  bio         LONGTEXT
);
```

PostgreSQL:
```sql
CREATE TABLE employees (
  emp_id      SERIAL PRIMARY KEY,
  first_name  VARCHAR(50),
  last_name   VARCHAR(50) NOT NULL,
  hire_date   DATE,
  salary      DECIMAL(10,2),
  commission  REAL,
  photo       BYTEA,
  bio         TEXT
);
```

**Tips:**
- Use VARCHAR for varying-length strings; avoid CHAR for most use cases (wasteful padding).
- Use DECIMAL/NUMERIC for financial data; avoid FLOAT/DOUBLE to prevent rounding errors.
- Use DATE for date-only columns; TIMESTAMP when time precision is needed.
- Use BLOB/BYTEA/LONGBLOB for binary data; TEXT/CLOB/LONGTEXT for large text.
- RAW for fixed-size binary like hashes or UUIDs (not LONG RAW, which is deprecated).
- BFILE for managing large external media files (Oracle); configure directory objects for OS access.
- ROWID for internal row identification and optimization; not recommended for application keys (use surrogate keys instead).
- Consider NULL vs NOT NULL constraint to enforce data quality.

---

### DateTime Data Types

- Purpose: Store date, time, and interval values with varying precision and timezone support.
- Common usage: Record timestamps for auditing, track durations between events, manage time-sensitive business logic.

**Oracle DateTime Data Types:**

| Data Type | Description | Example |
|-----------|-------------|---------|
| DATE | Date and time (stored as century, year, month, day, hour, minute, second) | DATE |
| TIMESTAMP | Date with fractional seconds precision (default 6 digits) | TIMESTAMP(6) |
| TIMESTAMP WITH TIME ZONE | Timestamp including timezone offset | TIMESTAMP WITH TIME ZONE |
| TIMESTAMP WITH LOCAL TIME ZONE | Timestamp converted to DB timezone | TIMESTAMP WITH LOCAL TIME ZONE |
| INTERVAL YEAR TO MONTH | Interval stored as years and months | INTERVAL YEAR(2) TO MONTH |
| INTERVAL DAY TO SECOND | Interval stored as days, hours, minutes, seconds | INTERVAL DAY(2) TO SECOND(6) |

**MySQL DateTime Data Types:**

| Data Type | Description | Example |
|-----------|-------------|---------|
| DATE | Date only (YYYY-MM-DD) | DATE |
| TIME | Time only (HH:MM:SS) | TIME |
| DATETIME | Date and time (YYYY-MM-DD HH:MM:SS) | DATETIME |
| TIMESTAMP | Date/time with auto-update on row modification | TIMESTAMP DEFAULT CURRENT_TIMESTAMP |
| YEAR | Year only (4-digit or 2-digit) | YEAR |

**PostgreSQL DateTime Data Types:**

| Data Type | Description | Example |
|-----------|-------------|---------|
| DATE | Date only (YYYY-MM-DD) | DATE |
| TIME | Time only (HH:MM:SS) | TIME |
| TIMESTAMP | Date and time without timezone | TIMESTAMP |
| TIMESTAMP WITH TIME ZONE | Date/time with timezone (TIMESTAMPTZ) | TIMESTAMP WITH TIME ZONE |
| INTERVAL | Time interval (duration) | INTERVAL |

**Oracle DateTime Examples:**

TIMESTAMP:
```sql
CREATE TABLE audit_log (
  log_id NUMBER PRIMARY KEY,
  action VARCHAR2(100),
  action_timestamp TIMESTAMP(6),  -- Fractional seconds with 6-digit precision
  created_date TIMESTAMP DEFAULT SYSTIMESTAMP
);

INSERT INTO audit_log (log_id, action, action_timestamp)
VALUES (1, 'INSERT', TIMESTAMP '2025-11-21 14:30:45.123456');
```

TIMESTAMP WITH TIME ZONE:
```sql
CREATE TABLE global_events (
  event_id NUMBER PRIMARY KEY,
  event_name VARCHAR2(100),
  event_time TIMESTAMP WITH TIME ZONE
);

INSERT INTO global_events (event_id, event_name, event_time)
VALUES (1, 'Server Launch', TIMESTAMP '2025-11-21 14:30:45.123456 -05:00');
```

INTERVAL YEAR TO MONTH:
```sql
CREATE TABLE employee_tenure (
  emp_id NUMBER PRIMARY KEY,
  hire_date DATE,
  tenure INTERVAL YEAR(2) TO MONTH
);

INSERT INTO employee_tenure (emp_id, hire_date, tenure)
VALUES (101, TO_DATE('2020-06-15','YYYY-MM-DD'), INTERVAL '5-3' YEAR TO MONTH);
-- Employee hired 5 years and 3 months ago
```

INTERVAL DAY TO SECOND:
```sql
CREATE TABLE project_timeline (
  project_id NUMBER PRIMARY KEY,
  project_name VARCHAR2(100),
  duration INTERVAL DAY(2) TO SECOND(6)
);

INSERT INTO project_timeline (project_id, project_name, duration)
VALUES (201, 'Website Redesign', INTERVAL '30 08:45:30.123456' DAY TO SECOND);
-- Project duration: 30 days, 8 hours, 45 minutes, 30.123456 seconds
```

**MySQL DateTime Examples:**

```sql
CREATE TABLE events (
  event_id INT AUTO_INCREMENT PRIMARY KEY,
  event_name VARCHAR(100),
  event_date DATE,
  event_time TIME,
  event_datetime DATETIME,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

INSERT INTO events (event_name, event_date, event_time, event_datetime)
VALUES ('Conference', '2025-11-21', '14:30:00', '2025-11-21 14:30:00');

-- Query events in the last 24 hours
SELECT * FROM events WHERE created_at >= DATE_SUB(NOW(), INTERVAL 1 DAY);
```

**PostgreSQL DateTime Examples:**

```sql
CREATE TABLE audit_log (
  log_id SERIAL PRIMARY KEY,
  action VARCHAR(100),
  action_timestamp TIMESTAMP,
  action_with_tz TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO audit_log (action, action_timestamp, action_with_tz)
VALUES ('INSERT', '2025-11-21 14:30:45', '2025-11-21 14:30:45+00:00');

-- Query events using interval
SELECT * FROM audit_log WHERE created_at > NOW() - INTERVAL '1 day';
```

**DateTime Arithmetic Examples:**

Oracle:
```sql
-- Add/subtract days
SELECT hire_date + 30 FROM employees;  -- Add 30 days
SELECT hire_date - INTERVAL '1' YEAR FROM employees;  -- Subtract 1 year

-- Calculate duration between timestamps
SELECT EXTRACT(DAY FROM (SYSDATE - hire_date)) AS days_employed FROM employees;

-- Interval arithmetic
SELECT INTERVAL '1' YEAR + INTERVAL '3' MONTH FROM dual;
```

MySQL:
```sql
-- Add/subtract intervals
SELECT DATE_ADD(hire_date, INTERVAL 30 DAY) FROM employees;
SELECT DATE_SUB(hire_date, INTERVAL 1 YEAR) FROM employees;

-- Calculate days between dates
SELECT DATEDIFF(NOW(), hire_date) AS days_employed FROM employees;

-- Calculate specific intervals
SELECT TIMESTAMPDIFF(YEAR, hire_date, NOW()) AS years_employed FROM employees;
```

PostgreSQL:
```sql
-- Add/subtract intervals
SELECT hire_date + INTERVAL '30 days' FROM employees;
SELECT hire_date - INTERVAL '1 year' FROM employees;

-- Calculate age
SELECT AGE(NOW(), hire_date) FROM employees;

-- Extract interval parts
SELECT EXTRACT(DAY FROM (NOW() - hire_date)) AS days_employed FROM employees;
```

**Tips:**
- Use DATE for date-only values; DATETIME/TIMESTAMP for date and time.
- Use TIMESTAMP WITH TIME ZONE when working with global applications; handle timezone conversions carefully.
- INTERVAL types are useful for calculating durations and time differences.
- MySQL TIMESTAMP auto-updates on row modification; use DATETIME for static values.
- PostgreSQL INTERVAL is flexible and supports arithmetic on date/time values.
- Oracle INTERVAL YEAR TO MONTH and DAY TO SECOND provide explicit interval storage.
- Always consider precision requirements (milliseconds vs microseconds) when choosing TIMESTAMP precision.

---

### MySQL DDL Examples

MySQL has similar DDL concepts to Oracle but with different syntax and available features. Below are key MySQL DDL operations.

**CREATE TABLE (MySQL):**

```sql
CREATE TABLE employees (
  emp_id INT AUTO_INCREMENT PRIMARY KEY,
  first_name VARCHAR(50),
  last_name VARCHAR(50) NOT NULL,
  email VARCHAR(100) UNIQUE,
  salary DECIMAL(10,2),
  department_id INT,
  hire_date DATE DEFAULT CURDATE(),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (department_id) REFERENCES departments(department_id),
  CHECK (salary >= 0)
);
```

**ALTER TABLE (MySQL):**

Add column:
```sql
ALTER TABLE employees ADD COLUMN phone VARCHAR(15);
```

Modify column:
```sql
ALTER TABLE employees MODIFY COLUMN salary DECIMAL(12,2);
```

Rename column (MySQL 8.0+):
```sql
ALTER TABLE employees RENAME COLUMN email TO employee_email;
```

Drop column:
```sql
ALTER TABLE employees DROP COLUMN phone;
```

Add constraint:
```sql
ALTER TABLE employees ADD CONSTRAINT emp_email_uk UNIQUE (employee_email);
ALTER TABLE employees ADD CONSTRAINT emp_salary_chk CHECK (salary >= 0);
```

Drop constraint:
```sql
ALTER TABLE employees DROP CONSTRAINT emp_email_uk;
```

Disable/Enable foreign key checks (MySQL-specific):
```sql
-- Disable FK checks for bulk loads
SET FOREIGN_KEY_CHECKS = 0;
-- Perform bulk INSERT/UPDATE
INSERT INTO employees (emp_id, first_name, last_name, department_id) VALUES (1, 'John', 'Doe', 999);
-- Re-enable FK checks
SET FOREIGN_KEY_CHECKS = 1;
```

**DROP TABLE (MySQL):**

```sql
DROP TABLE employees;
-- Or with IF EXISTS to avoid error if table doesn't exist
DROP TABLE IF EXISTS employees;
```

**TRUNCATE TABLE (MySQL):**

```sql
TRUNCATE TABLE employees;
-- Resets AUTO_INCREMENT counter to 1
```

**RENAME TABLE (MySQL):**

```sql
RENAME TABLE employees TO employees_archive;
-- Or rename multiple tables
RENAME TABLE employees TO employees_old, employees_backup TO employees;
```

**CREATE INDEX (MySQL):**

```sql
CREATE INDEX idx_emp_lastname ON employees(last_name);

-- Create unique index
CREATE UNIQUE INDEX idx_emp_email ON employees(email);

-- Drop index
DROP INDEX idx_emp_lastname ON employees;
```

**CREATE VIEW (MySQL):**

```sql
CREATE VIEW v_emp_dept AS
SELECT e.emp_id, e.first_name, e.last_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id;

-- Replace view
CREATE OR REPLACE VIEW v_emp_dept AS
SELECT e.emp_id, e.first_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id;

-- Drop view
DROP VIEW v_emp_dept;
```

**CREATE SEQUENCE (MySQL equivalent - AUTO_INCREMENT):**

MySQL uses AUTO_INCREMENT instead of sequences:
```sql
CREATE TABLE employees (
  emp_id INT AUTO_INCREMENT PRIMARY KEY,
  first_name VARCHAR(50)
);

-- Get last inserted auto_increment id
SELECT LAST_INSERT_ID();

-- Set starting value
ALTER TABLE employees AUTO_INCREMENT = 1000;
```

**CONSTRAINTS (MySQL):**

```sql
CREATE TABLE employees (
  emp_id INT AUTO_INCREMENT PRIMARY KEY,
  first_name VARCHAR(50) NOT NULL,
  last_name VARCHAR(50) NOT NULL,
  email VARCHAR(100) UNIQUE,
  salary DECIMAL(10,2) CHECK (salary >= 0),
  department_id INT,
  hire_date DATE DEFAULT CURDATE(),
  CONSTRAINT fk_emp_dept FOREIGN KEY (department_id) REFERENCES departments(department_id),
  CONSTRAINT chk_salary CHECK (salary >= 0)
);
```

**FOREIGN KEY with Actions (MySQL):**

```sql
CREATE TABLE employees (
  emp_id INT AUTO_INCREMENT PRIMARY KEY,
  first_name VARCHAR(50),
  department_id INT,
  FOREIGN KEY (department_id) REFERENCES departments(department_id) ON DELETE CASCADE ON UPDATE CASCADE
);

-- ON DELETE SET NULL (set FK to NULL when parent deleted)
CREATE TABLE employees (
  emp_id INT AUTO_INCREMENT PRIMARY KEY,
  first_name VARCHAR(50),
  department_id INT,
  FOREIGN KEY (department_id) REFERENCES departments(department_id) ON DELETE SET NULL ON UPDATE CASCADE
);
```

**MySQL-specific features:**

CHARSET and COLLATION:
```sql
CREATE TABLE employees (
  emp_id INT AUTO_INCREMENT PRIMARY KEY,
  first_name VARCHAR(50) COLLATE utf8mb4_unicode_ci,
  last_name VARCHAR(50) COLLATE utf8mb4_unicode_ci
) CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

Engine specification (InnoDB for transactions, MyISAM for read-only):
```sql
CREATE TABLE employees (
  emp_id INT AUTO_INCREMENT PRIMARY KEY,
  first_name VARCHAR(50)
) ENGINE=InnoDB;
```

Temporary table (auto-dropped when session ends):
```sql
CREATE TEMPORARY TABLE temp_emp (
  emp_id INT,
  first_name VARCHAR(50)
);
```

**View all tables (MySQL):**

```sql
SHOW TABLES;
SHOW TABLES LIKE 'emp%';  -- Tables starting with 'emp'
```

**View table structure (MySQL):**

```sql
DESCRIBE employees;
-- or
SHOW COLUMNS FROM employees;
-- or
SHOW CREATE TABLE employees;
```

**View constraints (MySQL):**

```sql
SELECT CONSTRAINT_NAME, CONSTRAINT_TYPE, TABLE_NAME
FROM INFORMATION_SCHEMA.TABLE_CONSTRAINTS
WHERE TABLE_NAME = 'employees';

-- View foreign keys
SELECT CONSTRAINT_NAME, TABLE_NAME, COLUMN_NAME, REFERENCED_TABLE_NAME, REFERENCED_COLUMN_NAME
FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE TABLE_NAME = 'employees' AND REFERENCED_TABLE_NAME IS NOT NULL;
```

**MySQL practical example (complete schema):**

```sql
CREATE DATABASE hr_system;
USE hr_system;

CREATE TABLE departments (
  department_id INT AUTO_INCREMENT PRIMARY KEY,
  department_name VARCHAR(100) NOT NULL UNIQUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE employees (
  emp_id INT AUTO_INCREMENT PRIMARY KEY,
  first_name VARCHAR(50) NOT NULL,
  last_name VARCHAR(50) NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  phone VARCHAR(15),
  salary DECIMAL(10,2) NOT NULL CHECK (salary > 0),
  department_id INT NOT NULL,
  hire_date DATE DEFAULT CURDATE(),
  status ENUM('Active', 'Inactive', 'Terminated') DEFAULT 'Active',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  CONSTRAINT fk_emp_dept FOREIGN KEY (department_id) REFERENCES departments(department_id) ON DELETE CASCADE,
  INDEX idx_emp_lastname (last_name),
  INDEX idx_emp_department (department_id)
);

CREATE TABLE projects (
  project_id INT AUTO_INCREMENT PRIMARY KEY,
  project_name VARCHAR(100) NOT NULL,
  department_id INT,
  start_date DATE,
  end_date DATE,
  CHECK (end_date >= start_date),
  FOREIGN KEY (department_id) REFERENCES departments(department_id)
);

CREATE VIEW v_emp_details AS
SELECT e.emp_id, e.first_name, e.last_name, e.email, d.department_name, e.salary
FROM employees e
JOIN departments d ON e.department_id = d.department_id;

-- Insert sample data
INSERT INTO departments (department_name) VALUES ('IT'), ('HR'), ('Sales');
INSERT INTO employees (first_name, last_name, email, salary, department_id) 
VALUES ('John', 'Doe', 'john.doe@company.com', 75000, 1);

-- Query the view
SELECT * FROM v_emp_details;
```

**Key MySQL DDL Differences from Oracle:**

| Feature | Oracle | MySQL |
|---------|--------|-------|
| Auto-increment | SEQUENCE + NEXTVAL | AUTO_INCREMENT |
| Read-only table | ALTER TABLE ... READ ONLY | Not directly supported; use triggers |
| Unused columns | SET UNUSED / DROP UNUSED | N/A; must use DROP COLUMN |
| Online DDL | ALTER TABLE ... ONLINE | Supported in MySQL 5.6+ |
| FK checks | Always enforced (if enabled) | Can disable with SET FOREIGN_KEY_CHECKS = 0 |
| Tablespaces | Specified in CREATE/ALTER | InnoDB uses .ibd files |
| Constraints | Named constraints | Constraints have names but less explicit control |
| Temp tables | CREATE GLOBAL/PRIVATE TEMP | CREATE TEMPORARY TABLE |
| Sequence alternatives | SEQUENCE objects | Use AUTO_INCREMENT or table generator |

---

**Course**: Oracle 19c SQL Workshop  
**Last Updated**: November 21, 2025