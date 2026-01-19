## Lesson 10 - Handling Exceptions (in which PL/SQL admits it is not perfect)

Welcome to Unit 3, where the code finally faces consequences. This lesson is about exceptions: how they happen, how to trap them, and how to avoid turning your session into a crash report.

By the end of this lesson, you should be able to:

- Define PL/SQL exceptions
- Recognize unhandled exceptions
- Use different types of exception handlers
- Trap unanticipated errors
- Describe exception propagation
- Customize exception messages

---

## 0. Where This Lesson Fits (Student Guide Lesson 9, aka “things go wrong”)

In the Student Guide, this material is **Lesson 9: Handling Exceptions**. The agenda walks through:

- What exceptions are and why they matter
- Exception types: predefined, non-predefined, and user-defined
- Syntax and guidelines for exception sections
- Trapping predefined and non-predefined exceptions
- Functions for trapping exceptions (`SQLCODE`, `SQLERRM`)
- User-defined exceptions and `RAISE`
- Propagation and `RAISE_APPLICATION_ERROR`

The examples here line up with that structure and lead straight into **Activity Guide Practice 9**, where you handle `NO_DATA_FOUND`, `TOO_MANY_ROWS`, and “everything else” while logging to the `MESSAGES` table.

---

## 1. What Is an Exception?

An exception is a runtime error that stops normal execution.

Example: a SELECT returns more than one row:

```sql
SELECT last_name
INTO v_lname
FROM employees
WHERE first_name = 'John';
```

Result: **TOO_MANY_ROWS**. Oracle does not let you jam multiple rows into a single variable. This is a feature, not a bug.

---

## 2. Why Handling Matters (a demo disaster)

If a single statement fails and you do not handle it, the whole block fails. Even valid work is lost.

Example: insert two valid rows and one duplicate primary key. Without an exception handler, all inserts are rolled back.

With a handler:

```sql
EXCEPTION
  WHEN DUP_VAL_ON_INDEX THEN
    DBMS_OUTPUT.PUT_LINE('Hey! You have a duplicate id!');
```

Now valid rows are preserved and the block exits gracefully.

---

## 3. Exception Types (the three-headed beast)

1) **Predefined** (Oracle gives you the name)
   - `NO_DATA_FOUND`, `TOO_MANY_ROWS`, `DUP_VAL_ON_INDEX`, `ZERO_DIVIDE`, etc.

2) **Non-Predefined** (Oracle gives you the code, not the name)
   - Example: `ORA-01400` (cannot insert NULL)

3) **User-Defined** (you create it)
   - Useful when Oracle does not throw an error but you want one

---

## 4. Predefined Exceptions

```sql
EXCEPTION
  WHEN TOO_MANY_ROWS THEN
    DBMS_OUTPUT.PUT_LINE('Your select statement retrieved multiple rows. Consider using a cursor.');
```

These are the easiest to use because Oracle already mapped the name to the error.

---

## 5. Non-Predefined Exceptions (PRAGMA EXCEPTION_INIT)

When the error has a code but no name, you create the name:

```sql
DECLARE
  e_oops EXCEPTION;
  PRAGMA EXCEPTION_INIT(e_oops, -1400);
BEGIN
  INSERT INTO departments VALUES (900, NULL, NULL, NULL);
EXCEPTION
  WHEN e_oops THEN
    DBMS_OUTPUT.PUT_LINE('No nulls for deptname.');
END;
/
```

You can do this for any Oracle error code.

---

## 6. WHEN OTHERS (the safety net)

Always place it last:

```sql
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    ...
  WHEN TOO_MANY_ROWS THEN
    ...
  WHEN OTHERS THEN
    ...
```

It catches anything you did not explicitly handle.

---

## 7. SQLCODE and SQLERRM (logging the chaos)

Use these functions to capture unknown errors:

```sql
DECLARE
  v_code NUMBER;
  v_msg  VARCHAR2(200);
BEGIN
  ...
EXCEPTION
  WHEN OTHERS THEN
    v_code := SQLCODE;
    v_msg  := SQLERRM;
    INSERT INTO error_log (user_name, log_time, err_code, err_msg)
    VALUES (USER, SYSDATE, v_code, v_msg);
END;
/
```

Now you have a log instead of a shrug.

---

## 8. User-Defined Exceptions (raising your own alarm)

Example with `RAISE_APPLICATION_ERROR`:

```sql
BEGIN
  DELETE FROM employees WHERE employee_id = 99;

  IF SQL%NOTFOUND THEN
    RAISE_APPLICATION_ERROR(-20999, 'Nobody with that id!');
  END IF;
END;
/
```

Or define your own exception and handle it gracefully:

```sql
DECLARE
  e_no_id EXCEPTION;
BEGIN
  DELETE FROM employees WHERE employee_id = 99;

  IF SQL%NOTFOUND THEN
    RAISE e_no_id;
  END IF;
EXCEPTION
  WHEN e_no_id THEN
    DBMS_OUTPUT.PUT_LINE('No ID exists for that employee!');
END;
/
```

---

## 9. Exception Propagation (nested block drama)

Exceptions can be raised in an inner block and handled in an outer block. This is how you centralize error handling without rewriting it in every nested block.

---

## 10. RAISE vs RAISE_APPLICATION_ERROR

- `RAISE`: transfers control to a handler in the same block
- `RAISE_APPLICATION_ERROR`: returns a user-defined error code/message to the caller

Oracle reserves `-20000` to `-20999` for your custom errors.

---

## 11. Quiz Recap

Statement: *You can trap any error by including a corresponding handler within the exception-handling section.*

Answer: **True**.

---

## 12. Wrap-Up

You now know how to:

- Handle predefined and non-predefined exceptions
- Raise user-defined exceptions
- Log unexpected errors
- Avoid silent failures

Next up: **Practice 9**, where you handle Oracle exceptions and build your own.
