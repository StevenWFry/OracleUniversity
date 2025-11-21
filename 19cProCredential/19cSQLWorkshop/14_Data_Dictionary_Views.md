# Data Dictionary Views

## Table of Contents
1. [Overview](#overview)
2. [Oracle Data Dictionary Views](#oracle-data-dictionary-views)
3. [MySQL Information Schema](#mysql-information-schema)
4. [PostgreSQL System Catalog](#postgresql-system-catalog)
5. [Common Queries](#common-queries)
6. [Performance and Metadata Management](#performance-and-metadata-management)
7. [Best Practices](#best-practices)

---

### Overview

- Purpose: Query metadata about database objects, users, privileges, storage, and performance.
- Common usage: View table/index definitions, check constraints, find object dependencies, audit permissions, troubleshoot performance.
- Important: Data dictionary views are read-only system views; they reflect current database schema state.

**Key Concepts:**
- USER_* views show objects owned by current user.
- ALL_* views show objects accessible to current user.
- DBA_* views show all objects in database (DBA privilege required).
- v$ views show dynamic performance data (real-time metrics).

---

### Oracle Data Dictionary Views

**Table Information:**

```sql
-- List all tables owned by current user
SELECT table_name, tablespace_name, num_rows, last_analyzed
FROM user_tables;

-- List all accessible tables (including those granted to you)
SELECT owner, table_name, tablespace_name
FROM all_tables
ORDER BY owner, table_name;

-- Get table size (blocks used)
SELECT table_name, blocks, num_rows, avg_row_len
FROM user_tables
WHERE table_name = 'EMPLOYEES';
```

**Column Information:**

```sql
-- View all columns in a table
SELECT column_name, data_type, data_length, nullable, data_default
FROM user_tab_columns
WHERE table_name = 'EMPLOYEES'
ORDER BY column_id;

-- Find all BLOB/CLOB columns
SELECT table_name, column_name, data_type
FROM user_tab_columns
WHERE data_type IN ('BLOB', 'CLOB', 'LONG RAW');

-- Find nullable columns
SELECT table_name, column_name, data_type
FROM user_tab_columns
WHERE table_name = 'EMPLOYEES' AND nullable = 'Y';
```

**Index Information:**

```sql
-- List all indexes
SELECT index_name, table_name, uniqueness, status
FROM user_indexes;

-- View columns in an index
SELECT index_name, column_name, column_position
FROM user_ind_columns
WHERE table_name = 'EMPLOYEES'
ORDER BY index_name, column_position;

-- Check index fragmentation
SELECT index_name, blevel, leaf_blocks, distinct_keys
FROM user_indexes
WHERE table_name = 'EMPLOYEES';
```

**Constraint Information:**

```sql
-- View all constraints
SELECT constraint_name, constraint_type, table_name, status
FROM user_constraints
WHERE table_name = 'EMPLOYEES';

-- View constraint details (columns involved)
SELECT constraint_name, column_name
FROM user_cons_columns
WHERE table_name = 'EMPLOYEES'
ORDER BY constraint_name, position;

-- Check for disabled constraints
SELECT constraint_name, constraint_type, status
FROM user_constraints
WHERE table_name = 'EMPLOYEES' AND status = 'DISABLED';

-- View foreign key relationships
SELECT uc.constraint_name, uc.table_name, ucc.column_name, 
       uc.r_constraint_name, ru.table_name AS referenced_table
FROM user_constraints uc
JOIN user_cons_columns ucc ON uc.constraint_name = ucc.constraint_name
JOIN user_constraints ru ON uc.r_constraint_name = ru.constraint_name
WHERE uc.constraint_type = 'R' AND uc.table_name = 'EMPLOYEES';
```

**View Information:**

```sql
-- List all views
SELECT view_name, text
FROM user_views;

-- Get view definition
SELECT view_name, text_length, text
FROM user_views
WHERE view_name = 'V_EMP_DEPT';

-- Check if view is valid
SELECT object_name, object_type, status
FROM user_objects
WHERE object_type = 'VIEW';
```

**Sequence Information:**

```sql
-- List all sequences
SELECT sequence_name, min_value, max_value, increment_by, last_number
FROM user_sequences;

-- Check sequence current value
SELECT sequence_name, last_number
FROM user_sequences
WHERE sequence_name = 'EMPLOYEES_SEQ';
```

**Synonym Information:**

```sql
-- List all synonyms
SELECT synonym_name, table_owner, table_name
FROM user_synonyms;

-- Check public synonyms
SELECT synonym_name, table_owner, table_name
FROM dba_synonyms
WHERE owner = 'PUBLIC';
```

**User and Privilege Information:**

```sql
-- View current user
SELECT user FROM dual;

-- View all users (DBA privilege required)
SELECT username, account_status, created
FROM dba_users;

-- View system privileges granted to user
SELECT privilege, admin_option
FROM user_sys_privs;

-- View object privileges granted to user
SELECT grantor, privilege, table_name
FROM user_tab_privs;

-- View roles granted to user
SELECT granted_role, admin_option
FROM user_role_privs;
```

**Tablespace Information:**

```sql
-- List all tablespaces
SELECT tablespace_name, extent_management, segment_space_management, status
FROM dba_tablespaces;

-- Check tablespace usage
SELECT tablespace_name, 
       SUM(bytes) / 1024 / 1024 AS total_mb,
       SUM(free_space) / 1024 / 1024 AS free_mb
FROM dba_free_space
GROUP BY tablespace_name;
```

**Dynamic Performance Views (v$ views):**

```sql
-- View active sessions
SELECT sid, serial#, username, osuser, machine, status
FROM v$session
WHERE username IS NOT NULL;

-- View locked objects
SELECT sid, serial#, username, type, lmode, request
FROM v$lock
WHERE request > 0;

-- View current SQL being executed
SELECT sid, sql_text
FROM v$sqlarea
WHERE executions > 0
ORDER BY executions DESC;

-- Check instance parameters
SELECT name, value, type, isdefault
FROM v$parameter
WHERE name LIKE '%cursor%';
```

**Object Dependencies:**

```sql
-- Find objects that depend on a table
SELECT name, type, referenced_owner, referenced_name, referenced_type
FROM user_dependencies
WHERE referenced_name = 'EMPLOYEES' AND referenced_type = 'TABLE';

-- Find which views use a specific table
SELECT view_name
FROM user_views
WHERE UPPER(text) LIKE '%EMPLOYEES%';
```

---

### MySQL Information Schema

MySQL uses INFORMATION_SCHEMA database for metadata queries.

**Table Information:**

```sql
-- List all tables in a database
SELECT TABLE_NAME, TABLE_TYPE, ENGINE, ROW_FORMAT, TABLE_ROWS, DATA_LENGTH
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = 'hr_system'
ORDER BY TABLE_NAME;

-- Get table size
SELECT TABLE_NAME, 
       ROUND(((DATA_LENGTH + INDEX_LENGTH) / 1024 / 1024), 2) AS size_mb
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = 'hr_system';
```

**Column Information:**

```sql
-- List all columns in a table
SELECT COLUMN_NAME, DATA_TYPE, COLUMN_TYPE, IS_NULLABLE, COLUMN_DEFAULT, EXTRA
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'hr_system' AND TABLE_NAME = 'employees'
ORDER BY ORDINAL_POSITION;

-- Find columns with specific datatype
SELECT TABLE_NAME, COLUMN_NAME, COLUMN_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_SCHEMA = 'hr_system' AND DATA_TYPE = 'VARCHAR';
```

**Index Information:**

```sql
-- List all indexes
SELECT TABLE_NAME, INDEX_NAME, COLUMN_NAME, SEQ_IN_INDEX, NON_UNIQUE
FROM INFORMATION_SCHEMA.STATISTICS
WHERE TABLE_SCHEMA = 'hr_system'
ORDER BY TABLE_NAME, INDEX_NAME, SEQ_IN_INDEX;

-- Find unique indexes
SELECT INDEX_NAME, TABLE_NAME, COLUMN_NAME
FROM INFORMATION_SCHEMA.STATISTICS
WHERE TABLE_SCHEMA = 'hr_system' AND NON_UNIQUE = 0;
```

**Constraint Information:**

```sql
-- List all constraints
SELECT CONSTRAINT_NAME, CONSTRAINT_TYPE, TABLE_NAME, COLUMN_NAME
FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE TABLE_SCHEMA = 'hr_system'
ORDER BY TABLE_NAME, CONSTRAINT_NAME;

-- View foreign key relationships
SELECT CONSTRAINT_NAME, TABLE_NAME, COLUMN_NAME, 
       REFERENCED_TABLE_NAME, REFERENCED_COLUMN_NAME
FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE TABLE_SCHEMA = 'hr_system' AND REFERENCED_TABLE_NAME IS NOT NULL;

-- Check primary keys
SELECT TABLE_NAME, COLUMN_NAME, ORDINAL_POSITION
FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE TABLE_SCHEMA = 'hr_system' AND CONSTRAINT_NAME = 'PRIMARY'
ORDER BY TABLE_NAME, ORDINAL_POSITION;
```

**View Information:**

```sql
-- List all views
SELECT TABLE_NAME, TABLE_TYPE, VIEW_DEFINITION
FROM INFORMATION_SCHEMA.VIEWS
WHERE TABLE_SCHEMA = 'hr_system';

-- Get view definition
SELECT VIEW_DEFINITION
FROM INFORMATION_SCHEMA.VIEWS
WHERE TABLE_SCHEMA = 'hr_system' AND TABLE_NAME = 'v_emp_details';
```

**Stored Procedures and Functions:**

```sql
-- List stored procedures
SELECT ROUTINE_NAME, ROUTINE_TYPE, CREATED, LAST_ALTERED
FROM INFORMATION_SCHEMA.ROUTINES
WHERE ROUTINE_SCHEMA = 'hr_system'
ORDER BY ROUTINE_TYPE, ROUTINE_NAME;
```

**Character Sets and Collations:**

```sql
-- Check table character set
SELECT TABLE_NAME, TABLE_COLLATION
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = 'hr_system';

-- Available character sets
SELECT CHARACTER_SET_NAME, DEFAULT_COLLATE_NAME, MAXLEN
FROM INFORMATION_SCHEMA.CHARACTER_SETS;
```

---

### PostgreSQL System Catalog

PostgreSQL stores metadata in system catalogs (pg_* tables in pg_catalog schema).

**Table Information:**

```sql
-- List all tables
SELECT schemaname, tablename, tableowner
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY schemaname, tablename;

-- Get table size
SELECT schemaname, tablename, pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

**Column Information:**

```sql
-- List columns in a table
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_schema = 'public' AND table_name = 'employees'
ORDER BY ordinal_position;
```

**Index Information:**

```sql
-- List all indexes
SELECT schemaname, tablename, indexname, indexdef
FROM pg_indexes
WHERE schemaname = 'public'
ORDER BY tablename, indexname;
```

**Constraint Information:**

```sql
-- List constraints
SELECT constraint_name, constraint_type, table_name
FROM information_schema.table_constraints
WHERE table_schema = 'public'
ORDER BY table_name, constraint_name;

-- View foreign key relationships
SELECT constraint_name, table_name, column_name, 
       referenced_table_name, referenced_column_name
FROM information_schema.referential_constraints rc
JOIN information_schema.key_column_usage kcu 
  ON rc.constraint_name = kcu.constraint_name
WHERE constraint_schema = 'public';
```

**View Information:**

```sql
-- List views
SELECT table_schema, table_name, view_definition
FROM information_schema.views
WHERE table_schema = 'public';
```

**Sequence Information:**

```sql
-- List sequences
SELECT schemaname, sequencename
FROM pg_sequences
WHERE schemaname = 'public';

-- Get current sequence value
SELECT last_value FROM employees_seq;
```

---

### Common Queries

**Across All Databases:**

Find all foreign keys:
```sql
-- Oracle
SELECT uc.constraint_name, uc.table_name, ucc.column_name,
       uc.r_constraint_name, ru.table_name AS ref_table
FROM user_constraints uc
JOIN user_cons_columns ucc ON uc.constraint_name = ucc.constraint_name
JOIN user_constraints ru ON uc.r_constraint_name = ru.constraint_name
WHERE uc.constraint_type = 'R';

-- MySQL
SELECT CONSTRAINT_NAME, TABLE_NAME, COLUMN_NAME,
       REFERENCED_TABLE_NAME, REFERENCED_COLUMN_NAME
FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE
WHERE REFERENCED_TABLE_NAME IS NOT NULL AND TABLE_SCHEMA = DATABASE();

-- PostgreSQL
SELECT constraint_name, table_name, column_name,
       referenced_table_name, referenced_column_name
FROM information_schema.referential_constraints rc
JOIN information_schema.key_column_usage kcu 
  ON rc.constraint_name = kcu.constraint_name;
```

Find disabled constraints:
```sql
-- Oracle
SELECT constraint_name, table_name, status
FROM user_constraints
WHERE status = 'DISABLED';

-- MySQL (check INFORMATION_SCHEMA for constraint status)
-- MySQL doesn't track disabled status directly; disable via ALTER TABLE

-- PostgreSQL
SELECT constraint_name, table_name, constraint_type
FROM information_schema.table_constraints
WHERE is_deferrable = 'YES';
```

List all objects and their sizes:
```sql
-- Oracle
SELECT object_type, COUNT(*) AS count
FROM user_objects
GROUP BY object_type;

-- MySQL
SELECT TABLE_TYPE, COUNT(*) AS count
FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = DATABASE()
GROUP BY TABLE_TYPE;

-- PostgreSQL
SELECT object_type, COUNT(*) AS count
FROM (
  SELECT 'TABLE' AS object_type FROM pg_tables WHERE schemaname = 'public'
  UNION ALL
  SELECT 'INDEX' FROM pg_indexes WHERE schemaname = 'public'
  UNION ALL
  SELECT 'VIEW' FROM pg_views WHERE schemaname = 'public'
) t
GROUP BY object_type;
```

---

### Performance and Metadata Management

**Oracle Performance Views:**

```sql
-- Find slow queries
SELECT sql_id, executions, elapsed_time / 1000000 AS elapsed_sec, 
       ROUND(elapsed_time / executions / 1000000, 3) AS avg_sec_per_exec
FROM v$sqlarea
WHERE executions > 0
ORDER BY elapsed_time DESC
FETCH FIRST 10 ROWS ONLY;

-- Check table access statistics
SELECT table_name, num_rows, last_analyzed, avg_row_len
FROM user_tables
WHERE num_rows > 0
ORDER BY num_rows DESC;

-- Find missing or unused indexes
SELECT index_name, table_name, uniqueness, status
FROM user_indexes
WHERE status = 'UNUSABLE';
```

**Gather Schema Statistics (Oracle):**

```sql
-- Refresh table statistics (for CBO - Cost Based Optimizer)
ANALYZE TABLE employees COMPUTE STATISTICS;

-- Using DBMS_STATS package (preferred)
BEGIN
  DBMS_STATS.gather_table_stats(
    ownname => 'HR',
    tabname => 'EMPLOYEES',
    estimate_percent => 100
  );
END;
/

-- View statistics
SELECT table_name, num_rows, avg_row_len, blocks
FROM user_tables
WHERE table_name = 'EMPLOYEES';
```

**MySQL Query Performance:**

```sql
-- Enable query profiling
SET profiling = 1;
SELECT * FROM employees WHERE department_id = 10;
SHOW PROFILES;

-- View detailed profile
SHOW PROFILE FOR QUERY 1;
```

---

### Best Practices

- Query data dictionary regularly to audit schema changes and maintain documentation.
- Use USER_* views for current user objects; ALL_* for accessible objects; DBA_* for administrative queries.
- Monitor dynamic performance views (v$) to identify bottlenecks and locked objects.
- Disable and re-enable constraints before/after bulk loads for performance.
- Keep metadata current by gathering statistics (Oracle DBMS_STATS, MySQL ANALYZE TABLE).
- Document custom views, sequences, and synonyms for team reference.
- Check constraints and FK relationships before DDL changes to avoid breaking dependencies.
- Use data dictionary views in troubleshooting scripts and automation.

---

**Course**: Oracle 19c SQL Workshop  
**Last Updated**: November 21, 2025