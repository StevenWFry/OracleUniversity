## Lesson 22 Lab - Managing Dependencies

This practice uses the dependency tree utilities to trace object relationships, then recompiles invalid objects. Think of it as triage for your schema.

Before starting:

- Run `cleanup_22.sql`

---

## 1. Build Dependency Trees

Confirm these exist:

- `add_employee` procedure
- `valid_dept_id` function

Run `utldtree.sql` from the PLPU labs directory to create:

- `DEPTREE_TEMPTAB`
- `DEPTREE_FILL`
- `DEPTREE`
- `IDEPTREE`

Then build the tree for `add_employee`:

```sql
EXECUTE deptree_fill('PROCEDURE', 'ORA61', 'ADD_EMPLOYEE');
SELECT * FROM ideptree;
```

Repeat for `valid_dept_id`:

```sql
EXECUTE deptree_fill('FUNCTION', USER, 'VALID_DEPT_ID');
SELECT * FROM ideptree;
```

Expected:

- `add_employee` depends on `valid_dept_id`

---

## 2. Create an Invalid Object

Make a copy of `employees`:

```sql
CREATE TABLE emps AS SELECT * FROM employees;
```

Then alter `employees`:

```sql
ALTER TABLE employees ADD (new_col NUMBER);
```

Check invalid objects:

```sql
SELECT object_name, object_type, status
FROM user_objects
WHERE status = 'INVALID';
```

---

## 3. Add RECOMPILE to COMPILE_PKG

In the package spec:

- Add `recompile` procedure

In the body:

- Loop through `USER_OBJECTS` where `status = 'INVALID'`
- Skip `PACKAGE BODY`
- Use dynamic SQL to `ALTER ... COMPILE`

Compile the package.

---

## 4. Recompile Invalid Objects

Run:

```sql
EXECUTE compile_pkg.recompile;
```

Recheck invalid objects:

```sql
SELECT object_name, object_type, status
FROM user_objects
WHERE status = 'INVALID';
```

If only package bodies remain invalid, compile them manually.

---

## Wrap-Up

You just:

- Used `DEPTREE_FILL` and `IDEPTREE` to inspect dependencies
- Forced invalid objects into existence (politely)
- Recompiled invalid objects with dynamic SQL

Practice 22 complete.
