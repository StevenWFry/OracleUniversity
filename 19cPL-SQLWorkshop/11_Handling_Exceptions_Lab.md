## Lesson 9 Lab - Handling Exceptions (Predefined Exceptions)

This practice builds a PL/SQL block that handles predefined exceptions while retrieving an employee by salary and writing messages to the `MESSAGES` table.

You will:

- Recreate the `MESSAGES` table
- Select a single employee by salary
- Handle `NO_DATA_FOUND` and `TOO_MANY_ROWS`
- Log results into `MESSAGES`

---

## 1. Recreate the MESSAGES Table

Run:

- `lab_06_01.sql`

This drops and recreates the `MESSAGES` table.

---

## 2. Build the PL/SQL Block

```sql
DECLARE
  v_ename  employees.last_name%TYPE;
  vm_sal   employees.salary%TYPE := 6000;
BEGIN
  SELECT last_name
  INTO v_ename
  FROM employees
  WHERE salary = vm_sal;

  INSERT INTO messages (results)
  VALUES (v_ename || ' - ' || TO_CHAR(vm_sal));

EXCEPTION
  WHEN NO_DATA_FOUND THEN
    INSERT INTO messages (results)
    VALUES ('No employee with a salary of ' || TO_CHAR(vm_sal));

  WHEN TOO_MANY_ROWS THEN
    INSERT INTO messages (results)
    VALUES ('More than one employee with a salary of ' || TO_CHAR(vm_sal));

  WHEN OTHERS THEN
    INSERT INTO messages (results)
    VALUES ('Some other error occurred.');
END;
/
```

---

## 3. Verify Output

```sql
SELECT *
FROM messages;
```

- With `vm_sal := 6000`, you should see: **More than one employee with a salary of 6000**
- With `vm_sal := 2000`, you should see: **No employee with a salary of 2000**

---

## 4. Wrap-Up

You now have a clean example of handling predefined exceptions without explicit cursors, and writing human-readable results to a table.