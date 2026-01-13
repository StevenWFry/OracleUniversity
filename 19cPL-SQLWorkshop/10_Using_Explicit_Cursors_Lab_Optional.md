## Lesson 8 Lab (Optional) - Top N Salaries with Explicit Cursors

This optional practice uses an explicit cursor to load the top N salaries into a table. You will also test edge cases like N = 0 or N > employee count.

---

## 1. Create TOP_SALARIES Table

Run the script:

- `lab_08_02.sql`

This creates a table called `TOP_SALARIES` with a single column, `SALARY`.

---

## 2. Build the Cursor Block

Declare:

- `v_num` = the number of top salaries to collect (example: 5)
- `v_sal` = a salary holder
- `c_emp_cursor` = salaries in descending order

```sql
DECLARE
  v_num NUMBER(3) := 5;
  v_sal employees.salary%TYPE;

  CURSOR c_emp_cursor IS
    SELECT salary
    FROM employees
    ORDER BY salary DESC;
BEGIN
  OPEN c_emp_cursor;
  FETCH c_emp_cursor INTO v_sal;

  WHILE c_emp_cursor%ROWCOUNT <= v_num
    AND c_emp_cursor%FOUND LOOP

    INSERT INTO top_salaries (salary)
    VALUES (v_sal);

    FETCH c_emp_cursor INTO v_sal;
  END LOOP;

  CLOSE c_emp_cursor;
END;
/
```

Note: if you want **unique salaries only**, change the cursor to:

```sql
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC;
```

---

## 3. Verify Results

```sql
SELECT *
FROM top_salaries;
```

If you did not use `DISTINCT`, duplicates are expected.

---

## 4. Edge Case Tests

Try:

- `v_num := 0`
- `v_num := 110` (greater than total employees)

After each test, clear the table:

```sql
DELETE FROM top_salaries;
COMMIT;
```

---

## 5. Wrap-Up

You have now:

- Used an explicit cursor to populate a table
- Controlled loop exit with `%ROWCOUNT` and `%FOUND`
- Tested boundary cases without infinite loops

Optional lab complete. Go reward yourself.