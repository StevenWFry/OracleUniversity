## Lesson 4 - Using %TYPE and Composite Data Types (in which we stop hard-coding and start trusting the database)

Welcome to the part of PL/SQL where you stop guessing data sizes and let Oracle do the work. It is liberating. It is also mildly terrifying because it removes your illusion of control.

By the end of this lesson, you should be able to:

- Use `%TYPE` to match column or variable types
- Explain Boolean, LOB, and composite data types
- Declare and print bind variables
- Use substitution variables and AUTOPRINT

---

## 1. %TYPE (the anti-hardcoding superhero)

`%TYPE` lets you declare a variable using the data type of a **table column** or **another variable**.

Example from a table column:

```sql
DECLARE
  v_emp_lname employees.last_name%TYPE;
BEGIN
  NULL;
END;
/
```

Example from another variable:

```sql
DECLARE
  v_balance     NUMBER(7,2);
  v_min_balance v_balance%TYPE;
BEGIN
  NULL;
END;
/
```

Why use it? Because when a column changes size, your code does not spontaneously combust.

---

## 2. The "Kochhar" Problem (aka, why %TYPE exists)

If you define a variable too small, this happens:

```sql
DECLARE
  v_lname VARCHAR2(5);
BEGIN
  SELECT last_name
  INTO v_lname
  FROM employees
  WHERE employee_id = 101;
END;
/
```

Result: **character string buffer too small** because `Kochhar` is 7 letters and your variable is a tiny, sad box.

With `%TYPE`, the variable always matches the column size, even if the column changes later.

---

## 3. Using Multiple %TYPE Variables

If you select multiple columns, you need multiple variables, and order matters:

```sql
DECLARE
  v_fname employees.first_name%TYPE;
  v_lname employees.last_name%TYPE;
BEGIN
  SELECT first_name, last_name
  INTO v_fname, v_lname
  FROM employees
  WHERE employee_id = 101;

  DBMS_OUTPUT.PUT_LINE(v_fname || ' ' || v_lname);
END;
/
```

---

## 4. Boolean Variables (true, false, or existential dread)

Boolean values are:

- `TRUE`
- `FALSE`
- `NULL`

You can combine them with `AND`, `OR`, and `NOT`.

---

## 5. Large Object (LOB) Types (for data that refuses to be small)

- **CLOB**: character large object
- **BLOB**: binary large object (images, files)
- **BFILE**: binary file stored **outside** the database
- **NCLOB**: national character LOB

LOBs can store huge amounts of data (up to terabytes, because why not).

---

## 6. Composite Data Types (rows and columns, now in bulk)

Composite types let you store more than one value at a time:

- **Records**: store a full row
- **Collections**: store a column of values

We will go deep into these later. For now, just know they exist and are waiting to be unleashed.

---

## 7. Bind Variables (variables that survive outside the block)

PL/SQL variables **die** when the block ends. Bind variables do not.

Create them in the host environment:

```sql
VARIABLE b_sal NUMBER

BEGIN
  SELECT salary
  INTO :b_sal
  FROM employees
  WHERE employee_id = 100;
END;
/

PRINT b_sal
```

Rules:

- Declare with `VARIABLE` in SQL*Plus or SQL Developer
- Refer to them with a leading colon `:b_sal`
- Use `PRINT` to display their values

If you have multiple bind variables, `PRINT` shows them all.

---

## 8. AUTOPRINT (because typing PRINT is beneath you)

```sql
SET AUTOPRINT ON
```

When AUTOPRINT is enabled, SQL Developer prints bind variable values automatically.

---

## 9. Substitution Variables (interactive drama)

Substitution variables prompt for input at runtime:

```sql
SELECT salary
FROM employees
WHERE employee_id = &emp_id;
```

- Use `SET VERIFY ON` to see old/new values
- Use `SET VERIFY OFF` to hide that output
- For strings, include quotes in the substitution (`'&name'`)

---

## 10. Quiz Recap (the sneaky test)

True statements:

- `%TYPE` declares a variable based on a **single column** or **another variable**
- `%TYPE` is prefixed with a table column or variable name

False statements:

- `%TYPE` declares a variable based on **multiple columns** (that is `%ROWTYPE`)

---

## 11. Wrap-Up

You now know how to let the database define your variable types, how to work with bind variables, and why `%TYPE` prevents future you from screaming into the void.

Next up: **Practice 3**, where you test identifiers, declarations, `%TYPE`, bind variables, and execution. Yes, it is a lot. No, you are not allowed to panic.