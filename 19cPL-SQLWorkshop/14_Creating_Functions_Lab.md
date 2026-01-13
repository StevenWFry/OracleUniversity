## Lesson 12 Lab - Creating Functions

This practice creates and invokes functions, then uses a validation function inside a procedure to insert employees safely.

Before starting:

- Run `cleanup_12.sql`

---

## 1. Create GET_JOB Function

```sql
CREATE OR REPLACE FUNCTION get_job (
  p_jobid IN jobs.job_id%TYPE
) RETURN jobs.job_title%TYPE IS
  v_title jobs.job_title%TYPE;
BEGIN
  SELECT job_title
  INTO v_title
  FROM jobs
  WHERE job_id = p_jobid;

  RETURN v_title;
END get_job;
/
```

Test with a bind variable:

```sql
VARIABLE b_title VARCHAR2(35)
EXECUTE :b_title := get_job('SA_REP')
PRINT b_title
```

Expected: **Sales Representative**

---

## 2. Create GET_ANNUAL_COMP Function

```sql
CREATE OR REPLACE FUNCTION get_annual_comp (
  p_sal IN employees.salary%TYPE,
  p_com IN employees.commission_pct%TYPE
) RETURN NUMBER IS
BEGIN
  RETURN (NVL(p_sal, 0) * 12) + (NVL(p_com, 0) * NVL(p_sal, 0) * 12);
END get_annual_comp;
/
```

Use it in a SELECT:

```sql
SELECT last_name, salary, commission_pct,
       get_annual_comp(salary, commission_pct) AS annual_comp
FROM employees
WHERE department_id = 30;
```

---

## 3. Create VALID_DEPTID Function

```sql
CREATE OR REPLACE FUNCTION valid_deptid (
  p_deptid IN departments.department_id%TYPE
) RETURN BOOLEAN IS
  v_dummy PLS_INTEGER;
BEGIN
  SELECT 1
  INTO v_dummy
  FROM departments
  WHERE department_id = p_deptid;

  RETURN TRUE;
EXCEPTION
  WHEN NO_DATA_FOUND THEN
    RETURN FALSE;
END valid_deptid;
/
```

---

## 4. Create ADD_EMPLOYEE Procedure

```sql
CREATE OR REPLACE PROCEDURE add_employee (
  p_fname   IN employees.first_name%TYPE,
  p_lname   IN employees.last_name%TYPE,
  p_email   IN employees.email%TYPE,
  p_job     IN employees.job_id%TYPE DEFAULT 'SA_REP',
  p_mgr     IN employees.manager_id%TYPE DEFAULT 145,
  p_sal     IN employees.salary%TYPE DEFAULT 1000,
  p_comm    IN employees.commission_pct%TYPE DEFAULT 0,
  p_deptid  IN employees.department_id%TYPE
) IS
BEGIN
  IF valid_deptid(p_deptid) THEN
    INSERT INTO employees (
      employee_id, first_name, last_name, email, hire_date, job_id,
      manager_id, salary, commission_pct, department_id
    ) VALUES (
      employees_seq.NEXTVAL, p_fname, p_lname, p_email, SYSDATE, p_job,
      p_mgr, p_sal, p_comm, p_deptid
    );
  ELSE
    RAISE_APPLICATION_ERROR(-20299, 'Invalid department ID');
  END IF;
END add_employee;
/
```

Invoke with named notation for the department:

```sql
EXECUTE add_employee('Joe', 'Harris', 'JHARRIS', p_deptid => 80)
```

Verify:

```sql
SELECT first_name, last_name, job_id, salary, department_id
FROM employees
WHERE last_name = 'Harris';
```

---

## Wrap-Up

You now have:

- Functions that return values into SQL expressions
- A validation function for department IDs
- A procedure that inserts employees with defaults and safety checks

Practice 12 complete.