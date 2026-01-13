## Lesson 14 Lab - Creating Packages

This practice builds two packages: one that groups job procedures/functions, and another that demonstrates public vs private constructs.

Before starting:

- Run `cleanup_14.sql`

---

## 1. Create JOB_PKG Specification

```sql
CREATE OR REPLACE PACKAGE job_pkg IS
  PROCEDURE add_job (
    p_jobid    IN jobs.job_id%TYPE,
    p_jobtitle IN jobs.job_title%TYPE
  );

  PROCEDURE del_job (
    p_jobid IN jobs.job_id%TYPE
  );

  FUNCTION get_job (
    p_jobid IN jobs.job_id%TYPE
  ) RETURN jobs.job_title%TYPE;

  PROCEDURE upd_job (
    p_jobid    IN jobs.job_id%TYPE,
    p_jobtitle IN jobs.job_title%TYPE
  );
END job_pkg;
/
```

---

## 2. Create JOB_PKG Body

```sql
CREATE OR REPLACE PACKAGE BODY job_pkg IS

  PROCEDURE add_job (
    p_jobid    IN jobs.job_id%TYPE,
    p_jobtitle IN jobs.job_title%TYPE
  ) IS
  BEGIN
    INSERT INTO jobs (job_id, job_title)
    VALUES (p_jobid, p_jobtitle);

    COMMIT;
  END add_job;

  PROCEDURE del_job (
    p_jobid IN jobs.job_id%TYPE
  ) IS
  BEGIN
    DELETE FROM jobs
    WHERE job_id = p_jobid;

    IF SQL%NOTFOUND THEN
      RAISE_APPLICATION_ERROR(-20203, 'No jobs deleted.');
    END IF;
  END del_job;

  FUNCTION get_job (
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

  PROCEDURE upd_job (
    p_jobid    IN jobs.job_id%TYPE,
    p_jobtitle IN jobs.job_title%TYPE
  ) IS
  BEGIN
    UPDATE jobs
    SET job_title = p_jobtitle
    WHERE job_id = p_jobid;

    IF SQL%NOTFOUND THEN
      RAISE_APPLICATION_ERROR(-20202, 'No job updated.');
    END IF;
  END upd_job;

END job_pkg;
/
```

---

## 3. Drop Standalone Objects

Drop old standalone procedures/functions (add_job, upd_job, del_job, get_job) if they still exist.

---

## 4. Invoke JOB_PKG.ADD_JOB

```sql
EXECUTE job_pkg.add_job('IT_SYSAN', 'Systems Analyst')
```

Verify:

```sql
SELECT *
FROM jobs
WHERE job_id = 'IT_SYSAN';
```

---

## 5. Create EMP_PKG (Public/Private)

Package spec:

```sql
CREATE OR REPLACE PACKAGE emp_pkg IS
  PROCEDURE add_employee (
    p_fname  IN employees.first_name%TYPE,
    p_lname  IN employees.last_name%TYPE,
    p_email  IN employees.email%TYPE,
    p_job    IN employees.job_id%TYPE DEFAULT 'SA_REP',
    p_mgr    IN employees.manager_id%TYPE DEFAULT 145,
    p_sal    IN employees.salary%TYPE DEFAULT 1000,
    p_comm   IN employees.commission_pct%TYPE DEFAULT 0,
    p_deptid IN employees.department_id%TYPE
  );

  PROCEDURE get_employee (
    p_empid IN  employees.employee_id%TYPE,
    p_sal   OUT employees.salary%TYPE,
    p_job   OUT employees.job_id%TYPE
  );
END emp_pkg;
/
```

Package body (private function inside):

```sql
CREATE OR REPLACE PACKAGE BODY emp_pkg IS

  FUNCTION valid_deptid (
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

  PROCEDURE add_employee (
    p_fname  IN employees.first_name%TYPE,
    p_lname  IN employees.last_name%TYPE,
    p_email  IN employees.email%TYPE,
    p_job    IN employees.job_id%TYPE DEFAULT 'SA_REP',
    p_mgr    IN employees.manager_id%TYPE DEFAULT 145,
    p_sal    IN employees.salary%TYPE DEFAULT 1000,
    p_comm   IN employees.commission_pct%TYPE DEFAULT 0,
    p_deptid IN employees.department_id%TYPE
  ) IS
  BEGIN
    IF valid_deptid(p_deptid) THEN
      INSERT INTO employees (
        employee_id, first_name, last_name, email, hire_date,
        job_id, manager_id, salary, commission_pct, department_id
      ) VALUES (
        employees_seq.NEXTVAL, p_fname, p_lname, p_email, SYSDATE,
        p_job, p_mgr, p_sal, p_comm, p_deptid
      );
    ELSE
      RAISE_APPLICATION_ERROR(-20299, 'Invalid department ID');
    END IF;
  END add_employee;

  PROCEDURE get_employee (
    p_empid IN  employees.employee_id%TYPE,
    p_sal   OUT employees.salary%TYPE,
    p_job   OUT employees.job_id%TYPE
  ) IS
  BEGIN
    SELECT salary, job_id
    INTO p_sal, p_job
    FROM employees
    WHERE employee_id = p_empid;
  END get_employee;

END emp_pkg;
/
```

---

## 6. Invoke EMP_PKG.ADD_EMPLOYEE

Invalid dept test:

```sql
EXECUTE emp_pkg.add_employee('Jane', 'Harris', 'JAHARRIS', p_deptid => 15)
```

Expected: Invalid department ID.

Valid dept test:

```sql
BEGIN
  emp_pkg.add_employee('David', 'Smith', 'DASMITH', p_deptid => 80);
END;
/
```

Verify:

```sql
SELECT first_name, last_name, department_id
FROM employees
WHERE last_name = 'Smith';
```

---

## Wrap-Up

You created and invoked packages with public and private constructs, and replaced standalone objects with packaged versions.