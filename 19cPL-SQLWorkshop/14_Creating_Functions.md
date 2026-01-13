## Lesson 14 - Creating Functions (in which values come home)

Welcome to Lesson 12 (Unit 4), where functions finally get their moment. Functions are like procedures, but with a strict rule: they must return a value. Think of them as the database equivalent of boomerangs.

By the end of this lesson, you should be able to:

- Differentiate between a procedure and a function
- Describe the uses of functions
- Create stored functions
- Invoke a function
- Remove a function
- Explain basic SQL Developer debugger functionality

---

## 1. What Is a Function?

A function is a **named PL/SQL block** that:

- Can accept parameters
- Must return a single value
- Can be used inside expressions (including SQL)

Procedures perform actions. Functions return values. That is the whole vibe.

---

## 2. Procedures vs Functions (quick reality check)

- **Procedure**: invoked as a statement, no return clause
- **Function**: invoked in an expression, must have a return clause

Procedures can use `RETURN;` to exit. Functions must use `RETURN value;` to return data.

---

## 3. Creating a Function

```sql
CREATE OR REPLACE FUNCTION get_sal (p_empid NUMBER)
  RETURN NUMBER
IS
  v_sal employees.salary%TYPE;
BEGIN
  SELECT salary
  INTO v_sal
  FROM employees
  WHERE employee_id = p_empid;

  RETURN v_sal;
END get_sal;
/
```

Note:

- The `RETURN` type appears in the header
- The function must return a value at least once

---

## 4. Invoking a Function

You do **not** invoke a function like a procedure. You must use it in an expression.

Examples:

```sql
BEGIN
  DBMS_OUTPUT.PUT_LINE(get_sal(100));
END;
/
```

```sql
VARIABLE v_sal NUMBER
EXECUTE :v_sal := get_sal(100)
PRINT v_sal
```

```sql
SELECT employee_id, get_sal(employee_id)
FROM employees
WHERE department_id = 60;
```

Yes, functions can appear in SQL. But there are rules.

---

## 5. Functions in SQL (restrictions)

User-defined functions in SQL must:

- Be stored in the database
- Use **IN** parameters only
- Accept and return SQL-compatible types

And they **must not**:

- Contain DML when called from SELECT
- Commit or rollback
- Modify the same table they are being called from

Example of what **not** to do:

```sql
-- function inserts into EMPLOYEES
-- then called from UPDATE on EMPLOYEES
```

Oracle will throw a mutating table error faster than you can say "side effects."

---

## 6. Named, Positional, and Default Parameters

Functions accept:

- Positional notation
- Named notation
- Mixed notation

Example with defaults:

```sql
tax_new(p_id => 100, p_allowance => 500)
```

Defaults let you skip optional values and still stay sane.

---

## 7. SQL Developer Debugger (preview only)

You can compile **for debug** and step through function logic:

- Step into loops
- Watch variable values
- See record contents in real time

Full debugger labs arrive in the next chapter. Consider this the trailer.

---

## 8. Dropping and Viewing Functions

Drop:

```sql
DROP FUNCTION function_name;
```

View source:

```sql
SELECT text
FROM user_source
WHERE type = 'FUNCTION'
  AND name = 'TAX_NEW'
ORDER BY line;
```

If you see `wrapped`, the source is intentionally obfuscated.

---

## 9. Quiz Recap

True statements about functions:

- Must contain a return clause in the header
- Must return a single value
- Must contain at least one return statement

False:

- "Does not contain a return clause" (absolutely does)

---

## 10. Wrap-Up

You now know how to:

- Create and invoke stored functions
- Use functions inside expressions and SQL
- Avoid side-effect errors
- Inspect and drop functions

Next up: **Practice 12**, where you build functions that query, return values, and run inside SQL statements.