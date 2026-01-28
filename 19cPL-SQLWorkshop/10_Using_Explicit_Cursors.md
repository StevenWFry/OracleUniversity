## Lesson 9 - Using Explicit Cursors (in which you finally take the wheel)

Welcome to Lesson 9, where PL/SQL stops driving you around and hands you the steering wheel, the pedals, and a warning label. We are talking explicit cursors: what they are, why they exist, and how to avoid running them off the road.

By the end of this lesson, you should be able to:

- Distinguish implicit vs explicit cursors
- Explain why explicit cursors are useful
- Declare, open, fetch, and close explicit cursors
- Use cursor FOR loops
- Use cursor parameters
- Lock rows with `FOR UPDATE`
- Update the current row with `WHERE CURRENT OF`

---

## 0. Where This Lesson Fits (Student Guide Lesson 8 with fewer laser pointers)

In the Student Guide, explicit cursors are **Lesson 8: Using Explicit Cursors**. The agenda covers:

- Cursors in general (implicit vs explicit)
- Declaring, opening, fetching from, and closing explicit cursors
- Cursor records and cursor FOR loops (including subquery FOR loops)
- Explicit cursor attributes: `%ISOPEN`, `%FOUND`, `%NOTFOUND`, `%ROWCOUNT`
- Cursor parameters and reusing the same cursor for multiple active sets
- `FOR UPDATE` and `WHERE CURRENT OF` for row-level locking and updates

This file plus its Part 2 and lab align with that structure and directly back up **Activity Guide Practice 8**, where you write the “due for a raise” cursor FOR loop and the two-cursor department/employee report.

---

## 1. Implicit vs Explicit Cursors (autopilot vs manual)

- **Implicit cursors** are created and managed by PL/SQL for every SQL statement.
- **Explicit cursors** are created and managed by you.

Implicit cursor attributes you already know:

- `SQL%FOUND`
- `SQL%NOTFOUND`
- `SQL%ROWCOUNT`

Explicit cursor attributes are the same idea, but scoped to the cursor name:

- `emp_cur%FOUND`
- `emp_cur%NOTFOUND`
- `emp_cur%ROWCOUNT`
- `emp_cur%ISOPEN`

---

## 2. The Explicit Cursor Lifecycle (the longhand version)

Steps:

1) **Declare** the cursor
2) **Open** it
3) **Fetch** rows
4) **Exit** when no rows remain
5) **Close** it

Example:

```sql
DECLARE
  CURSOR emp_cur IS
    SELECT employee_id, last_name
    FROM employees;

  v_emp emp_cur%ROWTYPE;
BEGIN
  OPEN emp_cur;

  LOOP
    FETCH emp_cur INTO v_emp;
    EXIT WHEN emp_cur%NOTFOUND;

    DBMS_OUTPUT.PUT_LINE(v_emp.last_name);
  END LOOP;

  CLOSE emp_cur;
END;
/
```

Note: Fetch **before** your `EXIT WHEN`, or you will loop forever printing the first row like a haunted printer.

---

## 3. Cursor Records (match the cursor, not the table)

If your cursor selects only a few columns, declare the record using `%ROWTYPE` **of the cursor**:

```sql
DECLARE
  CURSOR emp_cur IS
    SELECT last_name, salary
    FROM employees;

  v_emp emp_cur%ROWTYPE;
BEGIN
  OPEN emp_cur;
  LOOP
    FETCH emp_cur INTO v_emp;
    EXIT WHEN emp_cur%NOTFOUND;

    DBMS_OUTPUT.PUT_LINE(v_emp.last_name || ' - ' || v_emp.salary);
  END LOOP;
  CLOSE emp_cur;
END;
/
```

This prevents the "wrong number of values" error and keeps your record aligned.

---

## 4. Cursor Parameters (same cursor, different departments)

```sql
DECLARE
  CURSOR emp_cur (p_dept NUMBER) IS
    SELECT last_name
    FROM employees
    WHERE department_id = p_dept;
BEGIN
  OPEN emp_cur(10);
  LOOP
    FETCH emp_cur INTO v_name;
    EXIT WHEN emp_cur%NOTFOUND;
    DBMS_OUTPUT.PUT_LINE('Dept 10: ' || v_name);
  END LOOP;
  CLOSE emp_cur;

  OPEN emp_cur(20);
  LOOP
    FETCH emp_cur INTO v_name;
    EXIT WHEN emp_cur%NOTFOUND;
    DBMS_OUTPUT.PUT_LINE('Dept 20: ' || v_name);
  END LOOP;
  CLOSE emp_cur;
END;
/
```

If you forget to close the cursor before reopening, you get: **cursor already open**. It is not forgiving.

---

## 5. Cursor FOR Loops (the shortcut)

Cursor FOR loops handle open, fetch, exit, and close automatically, and they define the record for you.

```sql
DECLARE
  CURSOR emp_cur (p_dept NUMBER) IS
    SELECT last_name
    FROM employees
    WHERE department_id = p_dept;
BEGIN
  FOR emp_rec IN emp_cur(90) LOOP
    DBMS_OUTPUT.PUT_LINE(emp_rec.last_name);
  END LOOP;
END;
/
```

You can go even shorter and skip the cursor declaration entirely:

```sql
BEGIN
  FOR emp_rec IN (
    SELECT last_name
    FROM employees
    WHERE department_id = 90
  ) LOOP
    DBMS_OUTPUT.PUT_LINE(emp_rec.last_name);
  END LOOP;
END;
/
```

This is the "just get it done" version of explicit cursors.

---

## 6. Locking Rows (FOR UPDATE) and WHERE CURRENT OF

Use `FOR UPDATE` when you need to lock rows and update the exact row you just fetched:

```sql
DECLARE
  CURSOR emp_cur IS
    SELECT employee_id, salary
    FROM employees
    WHERE department_id = 50
    FOR UPDATE;
BEGIN
  FOR emp_rec IN emp_cur LOOP
    UPDATE employees
    SET salary = salary + 100
    WHERE CURRENT OF emp_cur;
  END LOOP;
END;
/
```

`WHERE CURRENT OF` updates the **current row** from the cursor, which is safer and cleaner than re-typing the primary key.

---

## 7. Wrap-Up

You now know how to:

- Build explicit cursors
- Control the fetch loop without skipping rows
- Use cursor FOR loops to save your sanity
- Parameterize cursors and reuse them
- Lock rows and update them safely

Next up: **Practice 8**, where you will use explicit cursors like a responsible adult and not accidentally print "King" 107 times.
