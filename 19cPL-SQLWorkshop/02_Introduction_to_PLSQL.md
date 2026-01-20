## Lesson 2 - Introduction to PL/SQL (in which SQL grows arms, legs, and opinions)

Welcome to Chapter 2: **PL/SQL Overview**. This is the part where SQL stops being a polite declarative request and starts doing actual work in a predictable, procedural way.

By the end of this lesson, you should be able to:

- Explain the need for PL/SQL
- Explain the benefits of PL/SQL
- Identify different types of PL/SQL blocks
- Output messages from PL/SQL
- Relate what you are doing in the lab to the matching lesson and practice numbers in the Student Guide and Activity Guide

---

## 1. Why PL/SQL Exists (SQL is great, but also... limited)

SQL does one operation at a time and does not naturally group multiple steps into a single, logical unit of work. PL/SQL fixes that by enabling:

- **Modularization**: break complex work into small, reusable tasks
- **Security**: store logic in the database instead of in every app
- **Maintainability**: one stored object, one version, easy to update
- **Exception handling**: errors are inevitable; panic is optional

Imagine a login screen. The app collects a username and password, passes them into a **PL/SQL function**, and the function returns **TRUE** or **FALSE**. The application then decides whether to let the user in or to politely refuse.

---

## 1a. Limitations of Plain SQL (as the Student Guide politely reminds you)

The Student Guide opens this lesson by pointing out exactly where SQL starts to wobble on its own:

- SQL is **set-oriented**, not procedural.
- SQL statements are **sent one at a time** from the client to the database.
- SQL on its own has **no variables, no loops, no IF/ELSE**, and no built-in error handling logic.

So if you try to write "check this, then that, then maybe update that other thing" in pure SQL, you end up with:

- A pile of round-trips between app and database.
- Business rules scattered across multiple places.
- No central, versioned, debuggable unit of work.

PL/SQL is the fix: you wrap multiple SQL statements, variables, and logic into a single block and send that to the database as one coherent thought.

Plain SQL from a client (multiple calls):

```sql
UPDATE employees
SET salary = salary * 1.05
WHERE employee_id = 100;

UPDATE employees
SET salary = salary * 1.05
WHERE employee_id = 101;
```

Same idea as one PL/SQL unit of work:

```sql
BEGIN
  UPDATE employees
  SET salary = salary * 1.05
  WHERE employee_id IN (100, 101);
END;
/
```

---

## 2. What PL/SQL Is (and what it adds)

PL/SQL stands for **Procedural Language Extension to SQL**. It:

- Integrates procedural constructs with SQL
- Uses a **block structure** for executable code
- Adds variables, constants, data types, and control structures
- Supports reusable program units stored in the database

So yes, you get **loops**, **IF/THEN/ELSE**, and **CASE**, plus the ability to compile and reuse stored logic.

---

## 2a. About PL/SQL (the brochure version from the Student Guide)

If we translate the Student Guide bullet slide into human:

- PL/SQL is **tightly integrated with SQL** – you do not bolt it on; it lives in the database engine.
- It is **portable** across Oracle platforms – your code travels better than most budget airlines.
- It supports **structured, modular programming** via blocks, subprograms, and packages.
- It includes **exception handling**, so you can specify exactly what happens when something explodes.

In other words, PL/SQL lets you:

- Keep data logic **close to the data**.
- Write code that is **readable, debuggable, and reusable**.
- Avoid having every application re-implement the same business rules in five different languages.

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

## 5a. Block Types (anonymous vs stored, like the Student Guide slide but louder)

The Student Guide calls out three main block “flavors”:

- **Anonymous blocks**
  - Not stored in the data dictionary.
  - Compiled each time they run.
  - Great for labs, ad‑hoc scripts, and experimentation.

- **Stored subprograms**
  - **Procedures** and **functions** created with `CREATE PROCEDURE` / `CREATE FUNCTION`.
  - Stored in the database, versioned and granted like other objects.
  - Reused by applications, jobs, and other PL/SQL.

- **Triggers**
  - Blocks wired to events (`INSERT`, `UPDATE`, `DELETE`, DDL, logon/logoff, etc.).
  - Fire automatically when the event occurs.

For Unit 1 (lessons 2–5 in the Student Guide), you live almost entirely in anonymous blocks, getting used to the syntax and basic runtime behavior.

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

## 6a. Examining and Executing a Block (this is literally what Practice 2 tests)

The Student Guide walks you through a simple block and then the Activity Guide immediately asks, “Which of these blocks actually run?”

Typical things to check:

- Does it have a `BEGIN ... END;` **executable section**?
- Are all statements **terminated with semicolons**?
- Is there a `/` on a line by itself when you run it as a script?

Example “good citizen” block:

```sql
SET SERVEROUTPUT ON

DECLARE
  v_amount  INTEGER(10);
BEGIN
  v_amount := 100;
  DBMS_OUTPUT.PUT_LINE('Amount is ' || v_amount);
END;
/
```

This is the style you formalize in **Activity Guide Practice 2: Introduction to PL/SQL**, where you test multiple block variants and then write your own `Hello World` block (`lab_02_02_soln.sql`).

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

This directly lines up with the “enable output then run as a script” steps in the Student Guide and in **Activity Guide Practice 2**.

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
