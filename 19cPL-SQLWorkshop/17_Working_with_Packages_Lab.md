## Lesson 15 Lab - Working with Packages

This practice modifies the `emp_pkg` package to add overloaded subprograms, forward declarations, and an initialization block that populates a PL/SQL table of valid department IDs.

Before starting:

- Run `cleanup15.sql`

---

## 1. Overload ADD_EMPLOYEE

In the package specification, add a second overload:

```sql
PROCEDURE add_employee (
  p_fname  IN employees.first_name%TYPE,
  p_lname  IN employees.last_name%TYPE,
  p_deptid IN employees.department_id%TYPE
);
```

Compile the spec.

In the package body, implement the overload to format the email and call the original procedure:

```sql
PROCEDURE add_employee (
  p_fname  IN employees.first_name%TYPE,
  p_lname  IN employees.last_name%TYPE,
  p_deptid IN employees.department_id%TYPE
) IS
  p_email employees.email%TYPE;
BEGIN
  p_email := UPPER(SUBSTR(p_fname, 1, 1) || SUBSTR(p_lname, 1, 7));
  emp_pkg.add_employee(p_fname, p_lname, p_email, p_deptid => p_deptid);
END add_employee;
```

Test:

```sql
EXECUTE emp_pkg.add_employee('Samuel', 'Joplin', 30)
```

Verify in EMPLOYEES.

---

## 2. Overload GET_EMPLOYEE

Add two overloaded functions to the spec:

```sql
FUNCTION get_employee (
  p_empid IN employees.employee_id%TYPE
) RETURN employees%ROWTYPE;

FUNCTION get_employee (
  p_fname IN employees.last_name%TYPE
) RETURN employees%ROWTYPE;
```

Implement both in the body:

```sql
FUNCTION get_employee (
  p_empid IN employees.employee_id%TYPE
) RETURN employees%ROWTYPE IS
  v_rec employees%ROWTYPE;
BEGIN
  SELECT *
  INTO v_rec
  FROM employees
  WHERE employee_id = p_empid;

  RETURN v_rec;
END get_employee;

FUNCTION get_employee (
  p_fname IN employees.last_name%TYPE
) RETURN employees%ROWTYPE IS
  v_rec employees%ROWTYPE;
BEGIN
  SELECT *
  INTO v_rec
  FROM employees
  WHERE last_name = p_fname;

  RETURN v_rec;
END get_employee;
```

---

## 3. Add PRINT_EMPLOYEE Procedure

Spec:

```sql
PROCEDURE print_employee (p_rec IN employees%ROWTYPE);
```

Body:

```sql
PROCEDURE print_employee (p_rec IN employees%ROWTYPE) IS
BEGIN
  DBMS_OUTPUT.PUT_LINE(
    p_rec.employee_id || ' ' || p_rec.first_name || ' ' ||
    p_rec.last_name || ' ' || p_rec.job_id || ' ' ||
    p_rec.salary || ' ' || p_rec.department_id
  );
END print_employee;
```

Invoke:

```sql
SET SERVEROUTPUT ON

BEGIN
  emp_pkg.print_employee(emp_pkg.get_employee(100));
  emp_pkg.print_employee(emp_pkg.get_employee('Joplin'));
END;
/
```

---

## 4. Add INIT_DEPARTMENTS and PL/SQL Table

In the spec:

```sql
PROCEDURE init_departments;
```

In the body, add a private associative array and procedure:

```sql
TYPE boolean_tab IS TABLE OF BOOLEAN INDEX BY BINARY_INTEGER;
valid_departments boolean_tab;

PROCEDURE init_departments IS
BEGIN
  FOR rec IN (SELECT department_id FROM departments) LOOP
    valid_departments(rec.department_id) := TRUE;
  END LOOP;
END init_departments;
```

Add an initialization block at the end of the package body:

```sql
BEGIN
  init_departments;
END;
```

---

## 5. Update VALID_DEPTID Function

Replace SELECT logic with:

```sql
RETURN valid_departments.EXISTS(p_deptid);
```

---

## 6. Test Department Validity

Attempt insert with invalid department:

```sql
EXECUTE emp_pkg.add_employee('James', 'Bond', 15)
```

Expected: Invalid department ID.

Insert department 15 and commit, then refresh state:

```sql
INSERT INTO departments (department_id, department_name)
VALUES (15, 'Security');
COMMIT;

EXECUTE emp_pkg.init_departments
EXECUTE emp_pkg.add_employee('James', 'Bond', 15)
```

---

## 7. Clean Up

```sql
DELETE FROM employees WHERE first_name = 'James' AND last_name = 'Bond';
DELETE FROM departments WHERE department_id = 15;
COMMIT;

EXECUTE emp_pkg.init_departments
```

---

## 8. Alphabetize Subprograms + Forward Declaration

After reorganizing subprograms, add a forward declaration for `valid_deptid` near the top of the package body:

```sql
FUNCTION valid_deptid (p_deptid IN departments.department_id%TYPE) RETURN BOOLEAN;
```

Compile to confirm no scope errors.

---

## Wrap-Up

You now have:

- Overloaded procedures and functions
- Initialization block
- Persistent package state via a PL/SQL table
- Forward declarations to keep the compiler calm

Practice 15 complete.