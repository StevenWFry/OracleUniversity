## Lesson 11 Lab (Part 2) - Creating Procedures with Parameters and Exceptions

This practice walks through creating and invoking procedures for insert, update, delete, and query operations, with exception handling and OUT parameters.

Before starting:

- Run `cleanup_11.sql` from `home/oracle/labs/plpu/code_examples`

---

## 1. Disable the Trigger (if needed)

If `SECURE_EMPLOYEES` is enabled and blocks DML:

- Right-click it under **Triggers**
- Choose **Disable**

---

## 2. Create ADD_JOB Procedure

```sql
CREATE OR REPLACE PROCEDURE add_job (
  p_jobid    IN jobs.job_id%TYPE,
  p_jobtitle IN jobs.job_title%TYPE
) IS
BEGIN
  INSERT INTO jobs (job_id, job_title)
  VALUES (p_jobid, p_jobtitle);

  COMMIT;
END add_job;
/
```

Invoke:

```sql
EXECUTE add_job('IT_DBA', 'Database Administrator')
```

Verify:

```sql
SELECT *
FROM jobs
WHERE job_id = 'IT_DBA';
```

Test duplicate:

```sql
EXECUTE add_job('ST_MAN', 'Stock Manager')
```

Expected error: unique constraint on JOB_ID.

---

## 3. Create UPDATE_JOB Procedure

```sql
CREATE OR REPLACE PROCEDURE upd_job (
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
/
```

Invoke and verify:

```sql
EXECUTE upd_job('IT_DBA', 'Data Administrator')

SELECT *
FROM jobs
WHERE job_id = 'IT_DBA';
```

Test exception:

```sql
EXECUTE upd_job('IT_WEB', 'Web Master')
```

---

## 4. Create DELETE_JOB Procedure

```sql
CREATE OR REPLACE PROCEDURE del_job (
  p_jobid IN jobs.job_id%TYPE
) IS
BEGIN
  DELETE FROM jobs
  WHERE job_id = p_jobid;

  IF SQL%NOTFOUND THEN
    RAISE_APPLICATION_ERROR(-20203, 'No jobs deleted.');
  END IF;
END del_job;
/
```

Invoke and verify:

```sql
EXECUTE del_job('IT_DBA')

SELECT *
FROM jobs
WHERE job_id = 'IT_DBA';
```

Test exception:

```sql
EXECUTE del_job('IT_WEB')
```

---

## 5. Create GET_EMPLOYEE Procedure (OUT parameters)

```sql
CREATE OR REPLACE PROCEDURE get_employee (
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
/
```

Invoke with bind variables:

```sql
VARIABLE v_salary NUMBER
VARIABLE v_job    VARCHAR2(15)

EXECUTE get_employee(120, :v_salary, :v_job)

PRINT v_salary
PRINT v_job
```

Test with a missing employee:

```sql
EXECUTE get_employee(300, :v_salary, :v_job)
```

Expected: NO_DATA_FOUND.

---

## Wrap-Up

You created and tested procedures with IN, OUT parameters, and exception handling across insert/update/delete/query tasks.