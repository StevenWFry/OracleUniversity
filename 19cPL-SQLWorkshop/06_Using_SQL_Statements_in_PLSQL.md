## Lesson 6 - Using SQL in PL/SQL (in which SQL moves in and becomes a roommate)

Welcome to Lesson 6, the finale of Unit 1. This is where SQL stops visiting and starts living inside your PL/SQL block, eating your groceries, and leaving cursor attributes on the counter.

By the end of this lesson, you should be able to:

- Determine which SQL statements can be used in PL/SQL
- Use the `INTO` clause to capture query results
- Manipulate data with DML inside PL/SQL
- Use transaction control statements
- Differentiate implicit vs explicit cursors
- Use SQL cursor attributes

---

## 1. SQL Inside PL/SQL (the rules of the house)

SQL includes:

- **DDL** (Data Definition Language): `CREATE`, `ALTER`, `DROP`
- **DML** (Data Manipulation Language): `INSERT`, `UPDATE`, `DELETE`, `MERGE`
- **DCL** (Data Control Language): `GRANT`, `REVOKE`

In PL/SQL blocks:

- **DDL is generally not included**
- **DML is absolutely included**
- **Transaction control** (`COMMIT`, `ROLLBACK`, `SAVEPOINT`) is allowed

---

## 2. SELECT INTO (the PL/SQL handshake)

When you run a `SELECT` in PL/SQL, you **must** use `INTO`, and your query must return **exactly one row**.

```sql
DECLARE
  v_fname employees.first_name%TYPE;
BEGIN
  SELECT first_name
  INTO v_fname
  FROM employees
  WHERE employee_id = 200;
END;
/
```

If it returns more than one row: **ORA-01422**. If it returns no rows: **ORA-01403**.

---

## 3. Multiple Columns, Multiple Variables

Column order must match variable order:

```sql
DECLARE
  v_hiredate employees.hire_date%TYPE;
  v_salary   employees.salary%TYPE;
BEGIN
  SELECT hire_date, salary
  INTO v_hiredate, v_salary
  FROM employees
  WHERE employee_id = 100;
END;
/
```

---

## 4. Group Functions (allowed, but only in SQL)

You cannot use group functions directly in PL/SQL expressions, but you **can** use them inside SQL statements embedded in PL/SQL:

```sql
DECLARE
  v_sum_sal NUMBER;
BEGIN
  SELECT SUM(salary)
  INTO v_sum_sal
  FROM employees
  WHERE department_id = 60;
END;
/
```

---

## 5. Naming Conventions Matter (or chaos wins)

Do not name variables the same as columns. It is confusing and can cause errors because:

- Column names take precedence in SQL statements
- Variable names take precedence over function names

Use prefixes (`v_`, `c_`, etc.) to avoid ambiguity and keep your sanity.

---

## 6. DML in PL/SQL (yes, it looks normal)

**INSERT**:

```sql
BEGIN
  INSERT INTO employees (employee_id, last_name, email, hire_date, job_id)
  VALUES (300, 'Smith', 'SMITH', SYSDATE, 'IT_PROG');
END;
/
```

**UPDATE**:

```sql
DECLARE
  v_increase NUMBER := 800;
BEGIN
  UPDATE employees
  SET salary = salary + v_increase
  WHERE job_id = 'ST_CLERK';
END;
/
```

**DELETE**:

```sql
DECLARE
  v_empid NUMBER := 176;
BEGIN
  DELETE FROM employees
  WHERE employee_id = v_empid;
END;
/
```

**MERGE** works the same way, just longer and more dramatic.

---

## 7. Cursors (the invisible helpers)

A **cursor** is a pointer to the private memory area that stores information about a SQL statement.

Two types:

- **Implicit cursors**: created automatically for every SQL statement
- **Explicit cursors**: created and managed by you (later chapter)

---

## 8. Implicit Cursor Attributes (the three spies)

After any SQL statement, you can ask:

- `SQL%FOUND` - did it affect at least one row?
- `SQL%NOTFOUND` - did it affect zero rows?
- `SQL%ROWCOUNT` - how many rows were affected?

Example:

```sql
DECLARE
  v_rows VARCHAR2(50);
BEGIN
  DELETE FROM employees
  WHERE employee_id = 165;

  v_rows := SQL%ROWCOUNT || ' row(s) deleted';
  DBMS_OUTPUT.PUT_LINE(v_rows);
END;
/
```

Important: these attributes reflect **only the most recent SQL statement**.

---

## 9. Demo Insight (because reality bites)

If you run a DELETE that matches no rows, the statement still succeeds. It just affects **0** rows. `SQL%ROWCOUNT` will tell you the truth.

If you do a DELETE and then an UPDATE, `SQL%ROWCOUNT` will report the UPDATE unless you captured it immediately after the DELETE.

---

## 10. Quiz Recap

Statement: *SELECT statements in PL/SQL require INTO and can return one or more rows.*

Answer: **False**. INTO is required, but the query must return **exactly one row**.

---

## 11. Wrap-Up

You now know how SQL behaves inside PL/SQL, how to use INTO, how to manipulate data, and how to interrogate the implicit cursor after the fact.

Next up: the Practice for Lesson 5, where you actually do all of this with real data and no safety net.