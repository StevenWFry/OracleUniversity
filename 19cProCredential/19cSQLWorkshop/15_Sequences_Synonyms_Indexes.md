# Sequences, Synonyms, and Indexes

## Table of Contents
1. [Overview](#overview)
2. [SEQUENCES](#sequences)
3. [SYNONYMS](#synonyms)
4. [INDEXES](#indexes)
5. [Best Practices](#best-practices)
6. [Performance Considerations](#performance-considerations)

---

### Overview

- Purpose: Create and manage auxiliary schema objects to enhance functionality, simplify access, and optimize performance.
- Common usage: Use sequences for auto-generated keys, synonyms for simplified object references, indexes for query optimization.
- Important: These are DDL operations; they may cause implicit commits and require appropriate privileges.

---

### SEQUENCES

**Purpose:** Generate unique numeric values automatically, typically for surrogate primary keys.

**Common usage:** Auto-populate primary keys in INSERT statements; ensure uniqueness across concurrent sessions; reset values as needed.

**CREATE SEQUENCE (Oracle):**

```sql
-- Basic sequence
CREATE SEQUENCE employees_seq
  START WITH 1000
  INCREMENT BY 1
  MINVALUE 1
  MAXVALUE 999999
  NOCACHE
  NOCYCLE;

-- Sequence with caching (performance optimization)
CREATE SEQUENCE emp_id_seq
  START WITH 100
  INCREMENT BY 1
  CACHE 20
  CYCLE;
```

Parameters:
- START WITH: starting value for the sequence.
- INCREMENT BY: increment value per NEXTVAL call.
- MINVALUE / MAXVALUE: range boundaries.
- CACHE: number of pre-generated values in memory (improves performance; default 20).
- NOCACHE: disable caching (ensures no gaps, slower).
- CYCLE: restart from MINVALUE when MAXVALUE reached.
- NOCYCLE: raise error if MAXVALUE reached (default).

**Using SEQUENCE in INSERT:**

```sql
INSERT INTO employees (emp_id, first_name, last_name, salary)
VALUES (employees_seq.NEXTVAL, 'John', 'Doe', 75000);

-- Get current value (does NOT increment)
SELECT employees_seq.CURRVAL FROM dual;

-- Get next value (increments sequence)
SELECT employees_seq.NEXTVAL FROM dual;
```

**ALTER SEQUENCE:**

```sql
-- Change increment
ALTER SEQUENCE employees_seq INCREMENT BY 5;

-- Change maximum value
ALTER SEQUENCE employees_seq MAXVALUE 9999999;

-- Reset cache
ALTER SEQUENCE employees_seq CACHE 50;
```

**DROP SEQUENCE:**

```sql
DROP SEQUENCE employees_seq;

-- Drop if exists (no error if not found)
DROP SEQUENCE IF EXISTS employees_seq;
```

**View Sequence Information (Oracle):**

```sql
-- List all sequences
SELECT sequence_name, min_value, max_value, increment_by, last_number, cache_size, cycle_flag
FROM user_sequences;

-- Get details for specific sequence
SELECT * FROM user_sequences WHERE sequence_name = 'EMPLOYEES_SEQ';
```

**Sequence in PL/SQL:**

```sql
DECLARE
  v_emp_id employees.emp_id%TYPE;
BEGIN
  v_emp_id := employees_seq.NEXTVAL;
  INSERT INTO employees (emp_id, first_name, last_name)
  VALUES (v_emp_id, 'Jane', 'Smith');
  DBMS_OUTPUT.PUT_LINE('Inserted employee id: ' || v_emp_id);
  COMMIT;
END;
/
```

**MySQL Alternative (AUTO_INCREMENT):**

MySQL does not have sequences; use AUTO_INCREMENT instead:

```sql
CREATE TABLE employees (
  emp_id INT AUTO_INCREMENT PRIMARY KEY,
  first_name VARCHAR(50),
  last_name VARCHAR(50)
);

INSERT INTO employees (first_name, last_name) VALUES ('John', 'Doe');

-- Get last inserted id
SELECT LAST_INSERT_ID();

-- Set next auto_increment value
ALTER TABLE employees AUTO_INCREMENT = 1000;
```

**PostgreSQL Alternative (SERIAL):**

PostgreSQL provides SERIAL pseudo-type (creates sequence + trigger automatically):

```sql
CREATE TABLE employees (
  emp_id SERIAL PRIMARY KEY,
  first_name VARCHAR(50),
  last_name VARCHAR(50)
);

-- Explicitly query sequence
SELECT last_value FROM employees_emp_id_seq;

-- Insert (sequence auto-increments)
INSERT INTO employees (first_name, last_name) VALUES ('John', 'Doe');
```

---

### SYNONYMS

**Purpose:** Create an alias for a table, view, sequence, or other object (local or public).

**Common usage:** Simplify object references across schemas, hide schema qualifiers, provide abstraction layer for schema refactoring.

**CREATE SYNONYM (Oracle):**

```sql
-- Local synonym (private to user)
CREATE SYNONYM emp FOR hr.employees;

-- Now you can query using synonym
SELECT * FROM emp;

-- Public synonym (accessible to all users, requires DBA privilege)
CREATE PUBLIC SYNONYM emp FOR hr.employees;
```

**DROP SYNONYM:**

```sql
DROP SYNONYM emp;

-- Drop public synonym
DROP PUBLIC SYNONYM emp;

-- Drop if exists
DROP SYNONYM IF EXISTS emp;
```

**View Synonyms (Oracle):**

```sql
-- List user's synonyms
SELECT synonym_name, table_owner, table_name, db_link
FROM user_synonyms;

-- List public synonyms (DBA privilege required)
SELECT synonym_name, table_owner, table_name
FROM dba_synonyms
WHERE owner = 'PUBLIC';
```

**Practical Example:**

```sql
-- DBA creates public synonym for shared table
CREATE PUBLIC SYNONYM employees FOR hr_schema.employees;

-- Application users can now query without schema prefix
SELECT * FROM employees;  -- Instead of: SELECT * FROM hr_schema.employees;

-- Useful for schema migration (change synonym target, no app code changes)
-- Old setup
CREATE PUBLIC SYNONYM employees FOR old_schema.employees;

-- New setup (no app changes needed)
DROP PUBLIC SYNONYM employees;
CREATE PUBLIC SYNONYM employees FOR new_schema.employees;
```

**Synonym for Remote Database Objects (using DB Link):**

```sql
-- Create database link to remote Oracle database
CREATE DATABASE LINK remote_db 
  CONNECT TO username IDENTIFIED BY password 
  USING 'remote_tnsname';

-- Create synonym for remote table
CREATE SYNONYM remote_emp FOR employees@remote_db;

-- Query remote table via synonym
SELECT * FROM remote_emp;
```

**Cross-Schema Access:**

```sql
-- Schema A: create synonym to Schema B's table
-- (assuming Schema B has granted SELECT privilege)
CREATE SYNONYM dept FOR schema_b.departments;

SELECT * FROM dept;
```

**Notes:**
- Synonyms are schema objects; they do not copy data (just reference).
- Dropping underlying object leaves synonym intact but unusable.
- Useful for application code stability during schema reorganization.

---

### INDEXES

**Purpose:** Create access paths to improve query performance by reducing full table scans.

**Common usage:** Index frequently filtered/joined columns, unique keys, and large tables; balance read performance against DML overhead.

**CREATE INDEX (Oracle):**

```sql
-- Simple index on single column
CREATE INDEX idx_emp_lastname ON employees(last_name);

-- Composite index (multiple columns)
CREATE INDEX idx_emp_dept_lastname ON employees(department_id, last_name);

-- Unique index
CREATE UNIQUE INDEX idx_emp_email ON employees(email);

-- Index with NOCOMPRESS (default)
CREATE INDEX idx_emp_fname ON employees(first_name) NOCOMPRESS;

-- Function-based index (Oracle-specific)
CREATE INDEX idx_emp_lastname_upper ON employees(UPPER(last_name));
```

**Index Options (Oracle):**

```sql
-- Index with tablespace specification
CREATE INDEX idx_emp_dept ON employees(department_id) TABLESPACE idx_tbsp;

-- Bitmap index (for low-cardinality columns in data warehouse)
CREATE BITMAP INDEX idx_emp_status ON employees(status);

-- Reverse key index (useful for sequences to avoid hot blocks)
CREATE INDEX idx_emp_id_rev ON employees(emp_id) REVERSE;

-- Partitioned index
CREATE INDEX idx_emp_part ON employees(hire_date) LOCAL;
```

**DROP INDEX:**

```sql
DROP INDEX idx_emp_lastname;

-- Drop if exists
DROP INDEX IF EXISTS idx_emp_lastname;

-- Rebuild index (Oracle)
ALTER INDEX idx_emp_lastname REBUILD;
```

**View Index Information (Oracle):**

```sql
-- List all indexes on a table
SELECT index_name, uniqueness, status
FROM user_indexes
WHERE table_name = 'EMPLOYEES';

-- View columns in an index
SELECT index_name, column_name, column_position
FROM user_ind_columns
WHERE table_name = 'EMPLOYEES'
ORDER BY index_name, column_position;

-- Check index fragmentation
SELECT index_name, blevel, leaf_blocks, distinct_keys, clustering_factor
FROM user_indexes
WHERE table_name = 'EMPLOYEES';

-- Monitor index usage (v$ view)
SELECT index_name, leaf_node_splits, leaf_delete_leaf_counts
FROM v$segment_statistics
WHERE object_type = 'INDEX' AND object_name = 'IDX_EMP_LASTNAME';
```

**MySQL CREATE INDEX:**

```sql
-- Simple index
CREATE INDEX idx_emp_lastname ON employees(last_name);

-- Unique index
CREATE UNIQUE INDEX idx_emp_email ON employees(email);

-- Composite index
CREATE INDEX idx_emp_dept_hire ON employees(department_id, hire_date);

-- Full-text index (MySQL-specific)
CREATE FULLTEXT INDEX idx_emp_bio ON employees(bio);

-- Drop index
DROP INDEX idx_emp_lastname ON employees;
```

**PostgreSQL CREATE INDEX:**

```sql
-- Simple index
CREATE INDEX idx_emp_lastname ON employees(last_name);

-- Unique index
CREATE UNIQUE INDEX idx_emp_email ON employees(email);

-- Composite index
CREATE INDEX idx_emp_dept_hire ON employees(department_id, hire_date);

-- Index on expression
CREATE INDEX idx_emp_lastname_upper ON employees(UPPER(last_name));

-- Drop index
DROP INDEX idx_emp_lastname;

-- Concurrently rebuild (non-blocking)
REINDEX INDEX CONCURRENTLY idx_emp_lastname;
```

**Index Design Best Practices:**

```sql
-- Good: Index frequently used WHERE columns
CREATE INDEX idx_emp_dept ON employees(department_id);
CREATE INDEX idx_emp_hire_date ON employees(hire_date);

-- Good: Composite index for multi-column filters
CREATE INDEX idx_emp_dept_hire ON employees(department_id, hire_date);

-- Avoid: Index on low-selectivity columns (few distinct values)
-- Bad: CREATE INDEX idx_emp_status ON employees(status);  -- only 3 values

-- Good: Use covering indexes (include columns needed for query)
-- Oracle 12c+: invisible indexes for testing
CREATE INDEX idx_emp_test ON employees(last_name) INVISIBLE;
ALTER INDEX idx_emp_test VISIBLE;

-- Check if index is used
SELECT index_name, used FROM v$object_usage WHERE index_name = 'IDX_EMP_LASTNAME';
```

**Index Maintenance:**

```sql
-- Oracle: monitor and rebuild fragmented indexes
ALTER INDEX idx_emp_lastname REBUILD ONLINE;

-- MySQL: analyze table to update index statistics
ANALYZE TABLE employees;

-- PostgreSQL: vacuum and analyze
VACUUM ANALYZE employees;

-- Check index size (Oracle)
SELECT index_name, round(bytes / 1024 / 1024, 2) AS size_mb
FROM user_segments
WHERE segment_type = 'INDEX';
```

**Index vs Performance Trade-offs:**

```sql
-- Too many indexes:
-- - Slow INSERT/UPDATE/DELETE (must maintain all indexes)
-- - Use extra storage
-- - Optimizer may choose sub-optimal plans

-- Too few indexes:
-- - Slow SELECT queries (full table scans)
-- - Less useful for ad-hoc queries

-- Balance: Index only columns frequently used in WHERE, JOIN, ORDER BY clauses
-- Example efficient set for employees table:
CREATE INDEX idx_emp_dept ON employees(department_id);        -- FK join
CREATE INDEX idx_emp_lastname ON employees(last_name);        -- Common search
CREATE UNIQUE INDEX idx_emp_email ON employees(email);        -- Unique constraint support
```

---

### Best Practices

**Sequences:**
- Use sequences for surrogate keys in OLTP systems.
- Set appropriate CACHE for balance between performance and gap tolerance.
- Document sequence naming convention.
- Test MAXVALUE and CYCLE behavior in edge cases.

**Synonyms:**
- Use public synonyms sparingly; prefer schema-qualified names for clarity.
- Update synonym targets during schema migrations (cleaner than code changes).
- Document synonym purposes and target objects.
- Drop unused synonyms to reduce clutter.

**Indexes:**
- Index columns used in WHERE, JOIN, and ORDER BY clauses.
- Avoid indexing low-cardinality columns (status, boolean flags).
- Monitor index usage and remove unused indexes.
- Balance read performance against DML overhead.
- Use composite indexes when queries filter/sort on multiple columns.
- Rebuild fragmented indexes periodically.
- Test query plans (EXPLAIN PLAN) before and after index creation.

---

### Performance Considerations

**Query Optimizer Interaction:**

```sql
-- Oracle: show execution plan
EXPLAIN PLAN FOR
  SELECT emp_id, first_name FROM employees WHERE last_name = 'Smith';

-- Display plan
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);

-- MySQL: show execution plan
EXPLAIN SELECT emp_id, first_name FROM employees WHERE last_name = 'Smith';

-- PostgreSQL: analyze query
EXPLAIN ANALYZE SELECT emp_id, first_name FROM employees WHERE last_name = 'Smith';
```

**Index Statistics:**

```sql
-- Oracle: gather statistics for better optimizer decisions
BEGIN
  DBMS_STATS.gather_table_stats(ownname => 'HR', tabname => 'EMPLOYEES');
END;
/

-- MySQL: analyze table
ANALYZE TABLE employees;

-- PostgreSQL: vacuum and analyze
VACUUM ANALYZE employees;
```

**Identify Unused Indexes (Oracle):**

```sql
-- Reset monitoring
ALTER INDEX idx_emp_lastname MONITORING USAGE;

-- Later, query usage
SELECT index_name, used, start_monitoring
FROM v$object_usage
WHERE index_name = 'IDX_EMP_LASTNAME';

-- Drop unused indexes
DROP INDEX idx_unused_col;
```

---

**Course**: Oracle 19c SQL Workshop  
**Last Updated**: November 21, 2025