## Lesson 9 - Using Explicit Cursors (Part 2: Attributes, Parameters, and Locks)

Welcome to the second half of explicit cursors. This is where you learn how to control them without accidentally printing the same row forever or locking your coworkers out of the database.

---

## 1. Cursor Attributes (your diagnostic dashboard)

Explicit cursor attributes tell you what just happened:

- `%ISOPEN` - is the cursor open?
- `%FOUND` - did the last fetch return a row?
- `%NOTFOUND` - did the last fetch return no row?
- `%ROWCOUNT` - how many rows have been fetched?

Example exit conditions:

```sql
EXIT WHEN emp_cur%ROWCOUNT > 10 OR emp_cur%NOTFOUND;
```

If you want only 10 rows, `%ROWCOUNT` is your bouncer.

---

## 2. %ISOPEN (do not fetch a closed cursor)

You can only fetch from an open cursor. So yes, check first:

```sql
IF NOT emp_cur%ISOPEN THEN
  OPEN emp_cur;
END IF;
```

It is a small check that prevents a large, embarrassing error.

---

## 3. Cursor FOR Loop with Subquery (no declaration needed)

Shortcut version using a subquery:

```sql
BEGIN
  FOR emp_rec IN (
    SELECT employee_id, last_name
    FROM employees
    WHERE department_id = 30
  ) LOOP
    DBMS_OUTPUT.PUT_LINE(emp_rec.employee_id || ' ' || emp_rec.last_name);
  END LOOP;
END;
/
```

No cursor declaration. No manual open/fetch/close. It is the fast-food version of explicit cursors.

---

## 4. Cursors with Parameters (same cursor, different active set)

```sql
DECLARE
  CURSOR emp_cur (p_dept NUMBER) IS
    SELECT last_name
    FROM employees
    WHERE department_id = p_dept;
BEGIN
  OPEN emp_cur(30);
  -- fetch and print
  CLOSE emp_cur;

  OPEN emp_cur(10);
  -- fetch and print
  CLOSE emp_cur;
END;
/
```

Parameters let you reuse the same cursor for different departments, without rewriting the SELECT.

---

## 5. FOR UPDATE and WHERE CURRENT OF (row-level control)

If you plan to update rows from a cursor, lock them first:

```sql
DECLARE
  CURSOR emp_cur IS
    SELECT employee_id, salary
    FROM employees
    WHERE department_id = 30
    FOR UPDATE;
BEGIN
  FOR emp_rec IN emp_cur LOOP
    UPDATE employees
    SET salary = salary + 500
    WHERE CURRENT OF emp_cur;
  END LOOP;

  COMMIT;
END;
/
```

Key points:

- `FOR UPDATE` locks the rows in the cursor's active set
- `WHERE CURRENT OF` updates the **current** fetched row
- **Always commit or rollback** to release the lock

---

## 6. WAIT and NOWAIT (don’t get stuck forever)

If another session holds a lock, your update can hang. Use `WAIT` or `NOWAIT`:

```sql
FOR UPDATE WAIT 10
```

or

```sql
FOR UPDATE NOWAIT
```

With `WAIT 10`, it retries for 10 seconds then returns control. With `NOWAIT`, it bails immediately. Either way, you avoid the "why is my session frozen" panic.

---

## 7. Lock Scope (FOR UPDATE OF)

If your cursor joins multiple tables, use `FOR UPDATE OF column_name` to lock only the target table’s rows. Otherwise, you may lock more than you intended, which is how rival DBAs are made.

---

## 8. Quiz Recap

Statement: *Explicit cursor functions enable a programmer to manually control explicit cursors in a PL/SQL block.*

Answer: **True**.

---

## 9. Wrap-Up

You now know how to:

- Use cursor attributes to manage control flow
- Reuse cursors with parameters
- Lock rows safely with `FOR UPDATE`
- Update the current row with `WHERE CURRENT OF`
- Avoid indefinite blocking with `WAIT` or `NOWAIT`

Next up: **Practice 8**, where you put all of this into motion and try not to lock the entire HR schema by accident.