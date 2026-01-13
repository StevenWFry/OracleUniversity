## Lesson 2 - Introduction to PL/SQL (in which SQL grows arms, legs, and opinions)

Welcome to Chapter 2: **PL/SQL Overview**. This is the part where SQL stops being a polite declarative request and starts doing actual work in a predictable, procedural way.

By the end of this lesson, you should be able to:

- Explain the need for PL/SQL
- Explain the benefits of PL/SQL
- Identify different types of PL/SQL blocks
- Output messages from PL/SQL

---

## 1. Why PL/SQL Exists (SQL is great, but also... limited)

SQL does one operation at a time and does not naturally group multiple steps into a single, logical unit of work. PL/SQL fixes that by enabling:

- **Modularization**: break complex work into small, reusable tasks
- **Security**: store logic in the database instead of in every app
- **Maintainability**: one stored object, one version, easy to update
- **Exception handling**: errors are inevitable; panic is optional

Imagine a login screen. The app collects a username and password, passes them into a **PL/SQL function**, and the function returns **TRUE** or **FALSE**. The application then decides whether to let the user in or to politely refuse.

---

## 2. What PL/SQL Is (and what it adds)

PL/SQL stands for **Procedural Language Extension to SQL**. It:

- Integrates procedural constructs with SQL
- Uses a **block structure** for executable code
- Adds variables, constants, data types, and control structures
- Supports reusable program units stored in the database

So yes, you get **loops**, **IF/THEN/ELSE**, and **CASE**, plus the ability to compile and reuse stored logic.

---

## 3. Anonymous Blocks vs Subprograms (practice wheels vs engines)

For the first part of the course, you will use **anonymous blocks**:

- Not stored in the database
- Compiled each time they run

Later, you will build **subprograms**:

- **Procedures**
- **Functions**
- **Triggers**

These are stored, compiled, and reusable.

---

## 4. Benefits of PL/SQL (the actual sales pitch)

- **Performance**: stored, compiled code runs efficiently
- **Result caching**: functions can reuse cached results
- **Portability**: runs anywhere Oracle runs
- **Backwards compatibility**: old code still works (mostly)
- **Exception handling**: define exactly what happens when things go wrong
- **Integration**: works seamlessly with Oracle tools

---

## 5. Runtime Architecture (two engines, one mission)

PL/SQL uses two execution engines:

- **PL/SQL Engine**: handles procedural logic
- **SQL Statement Executor**: handles SQL statements

Each statement goes to the right engine, so PL/SQL can orchestrate SQL instead of replacing it.

---

## 6. PL/SQL Block Structure (the three-section sandwich)

A PL/SQL block has up to three sections:

1) **Declarative** (optional) - variables, cursors, exceptions
2) **Executable** (mandatory) - the `BEGIN ... END` block
3) **Exception** (optional) - error handling

Minimum valid block:

```sql
BEGIN
  NULL;
END;
/
```

The `/` is the **block terminator** used in SQL*Plus and in scripts when mixing PL/SQL and SQL.

---

## 7. Anonymous Block Example (the first name grab)

```sql
DECLARE
  v_fname VARCHAR2(20);
BEGIN
  SELECT first_name
  INTO v_fname
  FROM employees
  WHERE employee_id = 100;
END;
/
```

Notes:

- `SELECT ... INTO` must return **one row**
- If it returns more than one, you get `ORA-01422: exact fetch returns more than requested number of rows`

---

## 8. Output Messages (because we like to see our variables)

Use `DBMS_OUTPUT.PUT_LINE` to print values. You must also enable output:

```sql
SET SERVEROUTPUT ON

DECLARE
  v_lname VARCHAR2(20);
BEGIN
  SELECT last_name
  INTO v_lname
  FROM employees
  WHERE employee_id = 100;

  DBMS_OUTPUT.PUT_LINE('The last name of the employee is ' || v_lname);
END;
/
```

Key points:

- `SET SERVEROUTPUT ON` is required **once per session**
- Use **Run Script** (`F5`) for PL/SQL blocks

---

## 9. Quality of Life Tips (small changes, big sanity)

- **Toggle line numbers**: right-click the gutter, or enable in Preferences
- **Format code**: right-click worksheet > Format
- **Snippets**: create reusable snippets for
  - `DBMS_OUTPUT.PUT_LINE()`
  - `SET SERVEROUTPUT ON`

---

## 10. Quick Check (the quiz in disguise)

A PL/SQL block must contain declarative, executable, and exception sections.

Answer: **False**. Only the **executable** section is mandatory.

---

## 11. Wrap-Up (and the next practice)

You now know:

- Why PL/SQL exists
- How blocks are structured
- How to output values

Next up: **Practice 2**, where you identify valid blocks and write a simple one of your own.