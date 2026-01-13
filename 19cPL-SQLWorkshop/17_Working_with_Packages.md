## Lesson 17 - Working with Packages (in which packages learn new tricks)

Welcome to Lesson 15. This is the advanced package chapter: overloading, forward declarations, initialization blocks, and persistent package state. Basically, the stuff you only notice once things go wrong.

By the end of this lesson, you should be able to:

- Overload package procedures and functions
- Use forward declarations
- Create initialization blocks
- Manage persistent package state

---

## 1. Overloading (same name, different signature)

Overloading lets you create multiple subprograms with the same name, as long as the parameter list differs in **number, order, or data type family**.

Example:

```sql
PROCEDURE get_emp(p_id IN NUMBER, p_name OUT VARCHAR2);
PROCEDURE get_emp(p_fname IN VARCHAR2, p_name OUT VARCHAR2);
```

You cannot overload:

- Standalone subprograms
- Parameters that differ only by mode (IN vs OUT)
- Return type only (for functions)
- Same type family (e.g., VARCHAR and VARCHAR2)

---

## 2. Default Parameters and Overloading (a warning)

If overloaded versions both have defaults, Oracle can’t decide which one to call. You will get:

- `Too many declarations of GET_EMP match this call`

Avoid conflicting defaults unless you enjoy ambiguity errors.

---

## 3. Forward Declarations

If a subprogram is used before it is defined in a package body, you must declare it first. Forward declarations exist to keep the compiler calm.

---

## 4. Initialization Block (runs on first reference)

Package variables can be initialized in a block at the end of the package body:

```sql
BEGIN
  v_tax := 0.15;
END;
```

The initialization block runs once per session, when the package is first referenced.

---

## 5. Persistent Package State

Package variables and cursor state are stored **per session** in the UGA. That means:

- Two sessions can see different values in the same package variable
- Cursor position persists across subprogram calls

Example: if one function fetches 3 rows, the next function starts at row 4.

Use `PRAGMA SERIALLY_REUSABLE` to avoid persistent state when needed.

---

## 6. Using Packaged Functions in SQL

Call with package prefix:

```sql
SELECT employee_id, tax_pkg.tax(salary)
FROM employees;
```

Packaged functions follow the same SQL restrictions as standalone functions.

---

## 7. Wrap-Up

You now know how to:

- Overload package subprograms safely
- Use forward declarations to avoid compile errors
- Initialize package variables once per session
- Understand and control persistent state

Next up: **Practice 15**, where you create overloaded subprograms, add initialization blocks, and manage package state.