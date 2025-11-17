# Substitution Variables and SQL*Plus Settings

## Table of Contents
1. [Substitution Variables (SQL*Plus/SQLcl)](#substitution-variables-sqlplus--sqlcl)
2. [VERIFY and ECHO](#verify-and-echo-sqlplus--sqlcl)
3. [SET Statements (SQL*Plus/SQLcl)](#set-statements-sqlplus--sqlcl)
4. [MySQL SET Statements](#mysql-set-statements)

### Substitution Variables (SQL*Plus / SQLcl)

- Purpose: Provide run-time values to scripts or prompt users; two flavors: substitution variables (&, &&) and bind variables (:).
- Context: Used in SQL*Plus, SQLcl, SQL Developer script execution.

**Basic substitution**:
```sql
-- Prompts for value each time & is used
SELECT * FROM employees WHERE department_id = &dept_id;

-- Persistent after first prompt (&&)
SELECT * FROM employees WHERE department_id = &&dept_id;
SELECT * FROM departments WHERE department_id = &&dept_id;
```

**Positional parameters (script)**:
```sql
-- script: find_by_dept.sql
SELECT * FROM employees WHERE department_id = &1;

-- Run from SQL*Plus / SQLcl:
@find_by_dept.sql 30
```

**ACCEPT / DEFINE**:
```sql
-- Prompt with ACCEPT and reuse variable
ACCEPT dept PROMPT 'Enter department id: '
SELECT * FROM employees WHERE department_id = &dept;

-- Define without prompting
DEFINE dept = 50
SELECT * FROM employees WHERE department_id = &dept;
```

**Control substitution behavior**:
```sql
SET VERIFY OFF;    -- don't show before/after substitution
SET VERIFY ON;     -- show substitution
SET DEFINE OFF;    -- disable & substitution (useful for scripts with & literals)
```

**Bind variables (recommended for PL/SQL and re-use without parsing)**:
```sql
-- Declare a bind variable and assign value
VARIABLE b_emp_id NUMBER;
EXEC :b_emp_id := 101;

-- Use in query
SELECT * FROM employees WHERE emp_id = :b_emp_id;

-- In PL/SQL block
VARIABLE b_total NUMBER;
BEGIN
  SELECT COUNT(*) INTO :b_total FROM employees;
END;
PRINT b_total;
```

**Notes and tips**:
- Use bind variables (:) for performance and to avoid SQL injection in repeated executions.
- Use && for simple interactive scripts where re-prompting is undesirable.
- Use SET DEFINE OFF when your script contains literal ampersands (&) to avoid accidental substitution.
- Combine with q-quote or doubling single quotes when substitution text contains quotes.

### VERIFY and ECHO (SQL*Plus / SQLcl)
- **SET VERIFY**: Shows the SQL before and after substitution when ON.
- **SET ECHO**: When ON, echoes commands in a script as they are executed.
- **PROMPT / ECHO**: Print messages (PROMPT works in SQL*Plus; SQLcl supports ECHO).

**Examples**:
```sql
-- VERIFY ON shows substitution details
SET VERIFY ON
DEFINE dept = 10
SELECT * FROM employees WHERE department_id = &dept;
-- Output will show:
-- old  1: SELECT * FROM employees WHERE department_id = &dept
-- new  1: SELECT * FROM employees WHERE department_id = 10

-- Turn VERIFY off to suppress the before/after lines
SET VERIFY OFF

-- Use SET ECHO in scripts to display commands as they run
SET ECHO ON
-- script: show_dept.sql
PROMPT Running department query...
ACCEPT dept PROMPT 'Enter department id: '
SELECT * FROM employees WHERE department_id = &dept;
SET ECHO OFF

-- SQLcl's ECHO prints text (SQL*Plus uses PROMPT)
ECHO 'Script completed successfully.'
```

### SET Statements (SQL*Plus / SQLcl)
- Purpose: Control session/script environment and output formatting.
- Common SET options: PAGESIZE, LINESIZE, FEEDBACK, VERIFY, ECHO, SERVEROUTPUT, HEADING, TERMOUT, TIMING, DEFINE, SQLPROMPT, TRIMSPOOL.

**Examples**:
```sql
-- Pagination and line width
SET PAGESIZE 50
SET LINESIZE 120

-- Show/hide row count feedback
SET FEEDBACK ON
SET FEEDBACK OFF

-- Show before/after substitution (see Substitution Variables)
SET VERIFY ON
SET VERIFY OFF

-- Echo commands in scripts
SET ECHO ON
SET ECHO OFF

-- Enable DBMS_OUTPUT (Oracle)
SET SERVEROUTPUT ON SIZE 1000000
-- turn off when done
SET SERVEROUTPUT OFF

-- Show or hide column headings
SET HEADING ON
SET HEADING OFF

-- Suppress terminal output while spooling or running a script
SET TERMOUT OFF
SET TERMOUT ON

-- Enable timing information for statements
SET TIMING ON
SET TIMING OFF

-- Change the SQL prompt
SET SQLPROMPT "SQL> "

-- Control substitution character processing
SET DEFINE ON    -- enable &
SET DEFINE OFF   -- disable &

-- Trim trailing spaces when spooling output
SET TRIMSPOOL ON
SET TRIMSPOOL OFF

-- Example script header
SET PAGESIZE 100
SET LINESIZE 200
SET SERVEROUTPUT ON
SET VERIFY OFF
PROMPT Running department report...
SELECT department_id, COUNT(*) AS emp_count
FROM employees
GROUP BY department_id
ORDER BY department_id;
SET SERVEROUTPUT OFF
```

### MySQL SET Statements
- Purpose: Configure server/session variables and assign user variables in MySQL.
- Notes: Use SET for both session/global system variables and user-defined variables. Changes to GLOBAL variables require appropriate privileges and affect new connections.

**Syntax highlights**:
- SET var_name = value;            -- session variable
- SET GLOBAL var_name = value;     -- global variable
- SET @user_var := value;          -- user-defined variable
- SET NAMES 'charset' [COLLATE 'collation']; -- client character set

**Examples**:
```sql
-- Set a session variable
SET sql_mode = 'TRADITIONAL';

-- Set a global variable (requires SUPER or SYSTEM_VARIABLES_ADMIN)
SET GLOBAL max_connections = 500;

-- Set a user-defined variable
SET @min_salary := 60000;
SELECT * FROM employees WHERE salary >= @min_salary;

-- Set character set for client connection
SET NAMES 'utf8mb4';

-- Set time zone for the session
SET time_zone = '+00:00';

-- Disable autocommit for transaction control
SET autocommit = 0;
START TRANSACTION;
-- ... transactional statements ...
COMMIT;
SET autocommit = 1;

-- Show current value
SHOW VARIABLES LIKE 'max_connections';
SHOW VARIABLES LIKE 'sql_mode';

-- Set multiple variables in one statement
SET SESSION sql_mode = 'STRICT_TRANS_TABLES,NO_AUTO_VALUE_ON_ZERO',
    session time_zone = 'Europe/London';
```

---

**Last Updated**: November 17, 2025
**Course**: Oracle 19c SQL Workshop