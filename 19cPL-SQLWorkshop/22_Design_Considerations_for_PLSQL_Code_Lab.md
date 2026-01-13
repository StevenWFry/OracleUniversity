## Lesson 20 Lab - Design Considerations for PL/SQL Code

This practice builds a bulk-fetch package and an `add_employee` procedure that logs with an autonomous transaction. In other words, performance and accountability, like a very polite security camera.

Before starting:

- Run `cleanup_20.sql`

---

## 1. Add BULK FETCH Support to EMP_PKG

In the package specification:

- Define a nested table type of `employees%ROWTYPE`
- Add `get_employees(p_dept_id)` procedure

In the package body:

- Declare a private collection variable (`emp_table`)
- Implement `get_employees` using `SELECT ... BULK COLLECT INTO emp_table`

Compile spec and body.

---

## 2. Add SHOW_EMPLOYEES Procedure

In the package specification:

- Add `show_employees` procedure (no parameters)

In the package body:

- If `emp_table` has data, loop and call `print_employee`

Compile and test:

```sql
BEGIN
  emp_pkg.get_employees(30);
  emp_pkg.show_employees;
END;
/

BEGIN
  emp_pkg.get_employees(60);
  emp_pkg.show_employees;
END;
/
```

---

## 3. Create LOG Table and Sequence

Run solution `solution11.sql` (Task 1) to create:

- `log_newemp` table
- `log_newemp_seq` sequence

---

## 4. Add Local Autonomous Logger

In the `add_employee` procedure (the one that does the real insert):

- Add a local procedure `audit_newemp`
- Mark it with `PRAGMA AUTONOMOUS_TRANSACTION`
- Insert into `log_newemp` using the sequence
- Commit inside the local procedure

Then call `audit_newemp` *before* the insert.

Compile the package body.

---

## 5. Test the Logger

Run the `add_employee` procedure twice, for example:

- Max Smart
- Clark Kent

Then check the log:

```sql
SELECT * FROM log_newemp;
```

---

## 6. Rollback Test

Run:

```sql
ROLLBACK;
```

Then verify:

- `employees` rows are gone
- `log_newemp` rows remain

That is the autonomous transaction doing its job while the main transaction sulks.

---

## Wrap-Up

You just:

- Bulk-fetched employees into a collection
- Printed results from a package table
- Logged operations with an autonomous transaction
- Proved that rollback does not erase the audit trail

Practice 20 complete.
