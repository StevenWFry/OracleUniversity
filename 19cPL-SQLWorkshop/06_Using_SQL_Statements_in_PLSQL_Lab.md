## Lesson 5 Lab - Using SQL in PL/SQL (in which SQL moves in and rearranges the furniture)

Welcome to Practice 5. This lab has you selecting, inserting, updating, and deleting inside PL/SQL blocks while tracking row counts like a forensic accountant.

You will:

- Select the maximum department ID
- Insert a new department
- Use SQL%ROWCOUNT to confirm changes
- Update and then delete the new department

---

## 0. Pre-Lab Cleanup

If you ran the code examples, run these first:

```sql
DROP TABLE employees2;
DROP TABLE copy_emp;
```

---

## 1. Find the Maximum Department ID

Create and run this block:

```sql
DECLARE
  v_max_deptno NUMBER;
BEGIN
  SELECT MAX(department_id)
  INTO v_max_deptno
  FROM departments;

  DBMS_OUTPUT.PUT_LINE('The maximum department_id is ' || v_max_deptno);
END;
/
```

Save as:

- `lab_05_01_soln.sql`

---

## 2. Insert a New Department

Modify the block to add a department:

```sql
DECLARE
  v_max_deptno NUMBER;
  v_dept_id    NUMBER;
  v_dept_name  departments.department_name%TYPE := 'Education';
BEGIN
  SELECT MAX(department_id)
  INTO v_max_deptno
  FROM departments;

  v_dept_id := v_max_deptno + 10;

  INSERT INTO departments (department_id, department_name, location_id)
  VALUES (v_dept_id, v_dept_name, NULL);

  DBMS_OUTPUT.PUT_LINE('The maximum department_id is ' || v_max_deptno);
  DBMS_OUTPUT.PUT_LINE('SQL%ROWCOUNT gives ' || SQL%ROWCOUNT);
END;
/

SELECT *
FROM departments
WHERE department_id = 280;
```

Save as:

- `lab_05_02_soln.sql`

---

## 3. Update and Delete the New Department

Create a new script:

```sql
BEGIN
  UPDATE departments
  SET location_id = 3000
  WHERE department_id = 280;
END;
/

SELECT *
FROM departments
WHERE department_id = 280;

BEGIN
  DELETE FROM departments
  WHERE department_id = 280;
END;
/
```

Save as:

- `lab_05_03_soln.sql`

---

## 4. Wrap-Up

You have now:

- Selected data with INTO
- Inserted a row and verified it
- Used SQL%ROWCOUNT to confirm DML
- Updated and deleted the same record

SQL now officially lives inside your PL/SQL block. Try to set boundaries.