## Lesson 19 Lab - Compound, DDL, and Event Triggers

This practice implements a salary integrity rule that causes a mutating table error, then fixes it by caching job salary ranges in a package. Think of it as teaching your triggers not to step on their own toes.

Before starting:

- Run `cleanup_19.sql`

---

## 1. Add SET_SALARY to EMP_PKG

Update your `emp_pkg` to include:

- `set_salary(p_jobid, p_minsal)` procedure
- Cursor over employees for that job
- Update each salary to the new minimum

Compile the package spec and body.

---

## 2. Create UPDATE_MIN_SALARY_TRIG

Row trigger on `jobs`:

```sql
CREATE OR REPLACE TRIGGER update_min_salary_trig
AFTER UPDATE OF min_salary ON jobs
FOR EACH ROW
BEGIN
  emp_pkg.set_salary(:NEW.job_id, :NEW.min_salary);
END;
/
```

---

## 3. Trigger the Mutating Table Error (On Purpose)

Run:

- Query IT_PROG employees (salary, min salary)
- Update `jobs.min_salary` for IT_PROG + 1000

Result:

- Mutating table error (the check_salary trigger reads `jobs` while it is being updated).

Yes, this is the database equivalent of tripping over your own shoelaces.

---

## 4. Create JOBS_PKG Cache

Build a package that stores job min/max salary in memory:

- `initialize`
- `get_min_sal(job_id)`
- `get_max_sal(job_id)`
- `set_min_sal(job_id, value)`
- `set_max_sal(job_id, value)`

Use an index-by table keyed on job_id.

---

## 5. Update CHECK_SALARY to Use JOBS_PKG

Modify the `check_salary` procedure to read min/max from `jobs_pkg` instead of querying `jobs`.

No direct query = no mutating table error.

---

## 6. Initialize Package State Before DML

Create a statement trigger on `jobs`:

```sql
CREATE OR REPLACE TRIGGER init_job_pkg_trig
BEFORE INSERT OR UPDATE ON jobs
BEGIN
  jobs_pkg.initialize;
END;
/
```

---

## 7. Retest

Repeat the IT_PROG update:

- Update min_salary + 1000
- Requery IT_PROG employees

Expected:

- No mutating table error
- Salaries updated correctly

---

## Wrap-Up

You just:

- Created a trigger that broke things
- Diagnosed a mutating table issue
- Fixed it with a package cache and a statement trigger

This is the database version of cleaning up your own mess like a responsible adult.
