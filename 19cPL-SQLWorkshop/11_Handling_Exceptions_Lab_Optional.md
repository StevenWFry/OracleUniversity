## Lesson 9 Lab - Handling Exceptions (ORA-2292)

This practice uses a non-predefined Oracle error (ORA-2292) and maps it to a named exception so you can handle it cleanly.

You will:

- Declare a custom exception
- Map it to ORA-2292 (integrity constraint, child record found)
- Attempt to delete a department
- Print a friendly message instead of crashing

---

## 1. Build the Exception Block

```sql
SET SERVEROUTPUT ON

DECLARE
  e_childrecord_exists EXCEPTION;
  PRAGMA EXCEPTION_INIT(e_childrecord_exists, -2292);
BEGIN
  DBMS_OUTPUT.PUT_LINE('Deleting department 40');

  DELETE FROM departments
  WHERE department_id = 40;

EXCEPTION
  WHEN e_childrecord_exists THEN
    DBMS_OUTPUT.PUT_LINE('Cannot delete this department. There are employees in this department. Child records exist.');
END;
/
```

Expected output:

- Deleting department 40
- Cannot delete this department. There are employees in this department. Child records exist.

---

## 2. Wrap-Up

You now know how to trap a non-predefined Oracle error and turn it into a readable message instead of a runtime tantrum.