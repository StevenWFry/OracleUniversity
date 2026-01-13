## Lesson 18 Lab - Creating Triggers

This practice builds row and statement triggers and calls procedures from triggers to enforce salary rules and business-hour restrictions. Think of it as HR policy, but with a tiny, judgmental robot.

Before starting:

- Run `cleanup_18.sql`

---

## 1. Create CHECK_SALARY Procedure

Use solution 9 (Task 1) or create manually:

```sql
CREATE OR REPLACE PROCEDURE check_salary (
  p_job  IN employees.job_id%TYPE,
  p_sal  IN employees.salary%TYPE
) IS
  v_min_sal jobs.min_salary%TYPE;
  v_max_sal jobs.max_salary%TYPE;
BEGIN
  SELECT min_salary, max_salary
  INTO v_min_sal, v_max_sal
  FROM jobs
  WHERE job_id = p_job;

  IF p_sal NOT BETWEEN v_min_sal AND v_max_sal THEN
    RAISE_APPLICATION_ERROR(
      -20201,
      'Invalid salary ' || p_sal ||
      '. Salaries for job ' || p_job ||
      ' must be between ' || v_min_sal ||
      ' and ' || v_max_sal
    );
  END IF;
END check_salary;
/
```

---

## 2. Create CHECK_SALARY_TRIG

```sql
CREATE OR REPLACE TRIGGER check_salary_trig
BEFORE INSERT OR UPDATE OF job_id, salary ON employees
FOR EACH ROW
BEGIN
  check_salary(:NEW.job_id, :NEW.salary);
END;
/
```

Test cases (let the trigger be the bad cop here):

- Add Eleanor Bay (dept 30) with invalid salary via `emp_pkg.add_employee`
- Update emp 115 salary to 2000 and job to HR_REP (both should fail)
- Update emp 115 salary to 2800 (should succeed)

---

## 3. Update Trigger to Fire Only When Values Change

```sql
CREATE OR REPLACE TRIGGER check_salary_trig
BEFORE INSERT OR UPDATE OF job_id, salary ON employees
FOR EACH ROW
BEGIN
  IF :NEW.job_id != :OLD.job_id OR :NEW.salary != :OLD.salary THEN
    check_salary(:NEW.job_id, :NEW.salary);
  END IF;
END;
/
```

Test (and yes, the trigger is now picky on purpose):

- Add Eleanor Bay (salary 5000, job IT_PROG) — success
- Update IT_PROG salaries +2000 — fails if salary exceeds max
- Update Eleanor Bay salary to 9000 — success
- Update job to ST_MAN — fails if salary too high

---

## 4. Create DELETE_M_TRIG (Business Hours Block)

```sql
CREATE OR REPLACE TRIGGER delete_m_trig
BEFORE DELETE ON employees
DECLARE
  v_day  VARCHAR2(3);
  v_hour NUMBER;
BEGIN
  v_day  := TO_CHAR(SYSDATE, 'DY');
  v_hour := TO_NUMBER(TO_CHAR(SYSDATE, 'HH24'));

  IF v_day NOT IN ('SAT', 'SUN') AND v_hour BETWEEN 9 AND 18 THEN
    RAISE_APPLICATION_ERROR(
      -20500,
      'Employee records cannot be deleted during business hours.'
    );
  END IF;
END;
/
```

Test with a DELETE statement. If you are outside business hours, you may need to temporarily invert the condition to see the error. This is the database saying, "Not now, buddy."

---

## Wrap-Up

You built:

- A reusable validation procedure
- A row trigger that enforces salary constraints
- A statement trigger that blocks deletes during business hours

Practice 18 complete. Your database now has the moral backbone of a stern librarian.
