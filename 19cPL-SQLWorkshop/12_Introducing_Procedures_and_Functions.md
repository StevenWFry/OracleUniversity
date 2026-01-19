## Lesson 12 - Introducing Stored Procedures and Functions (in which anonymous blocks grow up)

Welcome to the last chapter of Unit 3, where PL/SQL stops freelancing and starts getting a stable job in the database. This is the intro to procedures and functions.

By the end of this lesson, you should be able to:

- Differentiate between anonymous blocks and subprograms
- Create and invoke a simple procedure
- Create and invoke a simple function
- Build a function that accepts parameters
- Explain the difference between procedures and functions

---

## 0. Where This Lesson Fits (Student Guide Lessons 10–12 in one comfy bundle)

In the Student Guide, this topic starts as **Lesson 10: Introducing Stored Procedures and Functions**, then continues in:

- Lesson 11: Creating Procedures (syntax, creation, compilation, parameter modes)
- Lesson 12: Creating Functions (syntax, SQL usage, side-effect guidelines)

This chapter gives you the “single-slide” version:

- What subprograms are and why we use them instead of endless anonymous blocks
- How to create and call a simple procedure
- How to create and call a simple function (including from SQL)
- How to pass parameters and explain the procedure vs function role split

The lab ties directly to **Activity Guide Practice 10**, where you convert an anonymous block into `greet`, then upgrade it to accept a name.

---

## 1. What Are Subprograms?

A subprogram is a **named PL/SQL block** stored in the database. It can be called with or without parameters and comes in two main flavors:

- **Procedure**: performs an action
- **Function**: returns a value

Subprograms have a **specification** and a **body**, and once compiled they can be reused without re-compiling every time.

---

## 2. Anonymous Block vs Subprogram

Anonymous blocks:

- Unnamed
- Compiled every time
- Not stored in the database
- Cannot be called by other apps

Subprograms:

- Named
- Compiled once and stored
- Callable by other apps
- Can take parameters

Functions **must return a value**. Procedures **may** return a value, but usually just do work.

---

## 3. Creating a Procedure

Example procedure with parameters:

```sql
CREATE OR REPLACE PROCEDURE insert_demo_proc (
  p_id   NUMBER,
  p_name VARCHAR2
) IS
BEGIN
  INSERT INTO demo_table (id, last_name)
  VALUES (p_id, p_name);
EXCEPTION
  WHEN DUP_VAL_ON_INDEX THEN
    DBMS_OUTPUT.PUT_LINE('Duplicate ID');
END insert_demo_proc;
/
```

Notes:

- Use `OR REPLACE` to recompile without re-granting privileges
- Parameters cannot specify sizes (no `VARCHAR2(20)`)
- Naming the END is optional but helpful

---

## 4. Invoking a Procedure

Procedures are invoked inside an anonymous block:

```sql
BEGIN
  insert_demo_proc(1, 'King');
  insert_demo_proc(2, 'Kochhar');
  insert_demo_proc(1, 'Gietz'); -- duplicate
  insert_demo_proc(3, 'Gietz');
END;
/
```

Each call is independent, so a failure in one call does not block the rest if the exception is handled inside the procedure.

---

## 5. Creating a Function

A function **must return a value**:

```sql
CREATE OR REPLACE FUNCTION get_emp (
  p_id NUMBER
) RETURN VARCHAR2 IS
  v_lname employees.last_name%TYPE;
BEGIN
  SELECT last_name
  INTO v_lname
  FROM employees
  WHERE employee_id = p_id;

  RETURN v_lname;
END get_emp;
/
```

---

## 6. Invoking a Function

You do **not** call a function like a procedure. You must capture its return value:

```sql
BEGIN
  DBMS_OUTPUT.PUT_LINE(get_emp(100));
END;
/
```

You can also use functions in SQL:

```sql
SELECT employee_id, get_emp(employee_id)
FROM employees;
```

Functions are designed for this; procedures are not.

---

## 7. Procedures vs Functions (quick recap)

- **Procedure**: performs an action
- **Function**: returns a value
- **Function** can be used in SQL expressions
- **Procedure** is called as a standalone statement

---

## 8. Wrap-Up

You now know:

- How to create and call procedures
- How to create and call functions
- Why functions must return values
- Why procedures are better for action-oriented tasks

Next up: **Practice 10**, where you convert an anonymous block into a procedure and then call it with parameters.
