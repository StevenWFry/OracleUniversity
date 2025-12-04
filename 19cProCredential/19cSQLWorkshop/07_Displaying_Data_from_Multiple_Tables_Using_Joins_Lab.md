## Lesson 7 Lab – Displaying Data from Multiple Tables Using JOINs (or: getting your tables into the same room)

And look, your HR data is scattered across `EMPLOYEES`, `DEPARTMENTS`, `LOCATIONS`, `COUNTRIES`, and various lookup tables that have never met each other. This lab is where you make them mingle politely via joins.

You will:

- Join tables with equijoins using `ON` and `USING`.
- Use `NATURAL JOIN` when it’s actually safe.
- Write self‑joins to relate employees and colleagues/managers.
- Use nonequijoins for salary grades.
- Apply additional `WHERE` conditions on top of joins.

Use the **`ora1`** connection and the **SQL1** labs folder.

---

## Task 1 – Department Addresses (NATURAL JOIN)

**Goal:** Join `LOCATIONS` and `COUNTRIES` to list department addresses.

Requirements:

- Show: `location_id`, `street_address`, `city`, `state_province`, `country_name`.
- Use a `NATURAL JOIN`.

1. New worksheet.
2. Query:

   ```sql
   SELECT location_id,
          street_address,
          city,
          state_province,
          country_name
   FROM   locations
   NATURAL JOIN countries;
   ```

3. Run it and verify that each row shows an address and its country.

This works because `LOCATIONS` and `COUNTRIES` share `country_id` with the same name/type.

---

## Task 2 – Employees and Their Departments (USING)

**Goal:** Use an equijoin with `USING` to show employee/dept info.

Requirements:

- Show: `last_name`, `department_id`, `department_name`.
- Join `EMPLOYEES` and `DEPARTMENTS`.

1. New worksheet.
2. Query:

   ```sql
   SELECT last_name,
          department_id,
          department_name
   FROM   employees
   JOIN   departments USING (department_id);
   ```

3. Run it; you should see one row per employee who has a department.

If you qualify `department_id` (e.g., `employees.department_id`) in the `SELECT`, it will error because it’s part of the `USING` clause—don’t add a prefix there.

---

## Task 3 – Employees in Toronto (Three‑Table Join)

**Goal:** Join `EMPLOYEES`, `DEPARTMENTS`, and `LOCATIONS` and filter by city.

Requirements:

- Show: last name, job ID, department number, department name.
- Only employees whose `city` is **Toronto**.

1. New worksheet.
2. Use table aliases:

   ```sql
   SELECT e.last_name,
          e.job_id,
          e.department_id,
          d.department_name
   FROM   employees   e
   JOIN   departments d
          ON e.department_id = d.department_id
   JOIN   locations   l
          ON d.location_id   = l.location_id
   WHERE  LOWER(l.city) = 'toronto';
   ```

3. Run it; you should see the Toronto employees (typically in Marketing / department 20).

---

## Task 4 – Employees and Managers (Self‑Join) – `lab_07_04.sql`

**Goal:** Self‑join `EMPLOYEES` to list each employee and their manager.

Requirements:

- Columns (with labels):
  - Employee last name → `Employee`
  - Employee number → `Emp#`
  - Manager last name → `Manager`
  - Manager number → `Mgr#`
- Save as `lab_07_04.sql`.

1. New worksheet.
2. Query:

   ```sql
   SELECT w.last_name  AS "Employee",
          w.employee_id AS "Emp#",
          m.last_name  AS "Manager",
          m.employee_id AS "Mgr#"
   FROM   employees w
   JOIN   employees m
          ON w.manager_id = m.employee_id;
   ```

3. Save as `lab_07_04.sql`.
4. Run it to confirm each worker is paired with their manager.

Note that the top manager (King) does **not** appear because he has no `manager_id`.

---

## Task 5 – Include King with a LEFT JOIN – `lab_07_05.sql`

**Goal:** Modify the previous query so **all** employees appear, including those with no manager.

1. Copy the query from `lab_07_04.sql` into a new worksheet.
2. Change the join to a `LEFT JOIN` and sort by employee ID:

   ```sql
   SELECT w.last_name   AS "Employee",
          w.employee_id AS "Emp#",
          m.last_name   AS "Manager",
          m.employee_id AS "Mgr#"
   FROM   employees w
   LEFT  JOIN employees m
          ON w.manager_id = m.employee_id
   ORDER  BY w.employee_id;
   ```

3. Save as `lab_07_05.sql`.
4. Run it; you should now see King with a `NULL` manager.

---

## Task 6 – Employees and Colleagues in the Same Department – `lab_07_06.sql`

**Goal:** Self‑join to show employees and all colleagues in the **same department**.

Requirements:

- Show:
  - Department ID → `Department`
  - Employee last name → `Employee`
  - Colleague last name → `Colleague`
- Exclude rows where the employee is listed as their own colleague.
- Sort by department, employee, colleague.

1. New worksheet.
2. Query:

   ```sql
   SELECT e.department_id    AS "Department",
          e.last_name        AS "Employee",
          c.last_name        AS "Colleague"
   FROM   employees e
   JOIN   employees c
          ON e.department_id = c.department_id
   WHERE  e.employee_id <> c.employee_id
   ORDER  BY e.department_id,
             e.last_name,
             c.last_name;
   ```

3. Save as `lab_07_06.sql`.
4. Run it and inspect a few departments.

If you temporarily remove the `WHERE e.employee_id <> c.employee_id`, you’ll see each employee paired with themselves—exactly what that condition removes.

---

## Task 7 – Job Grades and Salaries (Nonequijoin)

**Goal:** Use `JOB_GRADES` to assign a grade based on salary.

Steps:

1. First, explore `JOB_GRADES`:

   ```sql
   DESC job_grades;
   ```

   You should see:
   - `grade_level`
   - `lowest_sal`
   - `highest_sal`

2. Now join employees to their pay grade:

   ```sql
   SELECT e.last_name   AS employee,
          e.job_id,
          d.department_name AS department,
          e.salary,
          j.grade_level
   FROM   employees   e
   JOIN   departments d
          ON e.department_id = d.department_id
   JOIN   job_grades  j
          ON e.salary BETWEEN j.lowest_sal AND j.highest_sal;
   ```

3. Run it; each employee should show a `grade_level` matching their salary band.

This `BETWEEN` join condition makes it a **nonequijoin**.

---

## Task 8 – Employees Hired After Davies

**Goal:** Use a self‑join to compare hire dates against a specific employee (Davies).

Requirements:

- Show: employee last name, employee hire date.
- Only employees hired **after** employee `Davies`.

1. New worksheet.
2. Query (one approach):

   ```sql
   SELECT e.last_name,
          e.hire_date
   FROM   employees e
   JOIN   employees d
          ON d.last_name = 'Davies'
   WHERE  e.hire_date > d.hire_date;
   ```

3. Optionally include Davies’ own hire date for reference:

   ```sql
   SELECT e.last_name,
          e.hire_date,
          d.hire_date AS davies_hire_date
   FROM   employees e
   JOIN   employees d
          ON d.last_name = 'Davies'
   WHERE  e.hire_date > d.hire_date;
   ```

Everyone listed has a hire date later than Davies.

---

## Task 9 – Employees Hired Before Their Managers – `lab_07_09.sql`

**Goal:** Find employees whose hire date is **earlier** than their manager’s.

Requirements:

- Show:
  - Worker last name
  - Worker hire date
  - Manager last name
  - Manager hire date
- Save as `lab_07_09.sql`.

1. New worksheet.
2. Query:

   ```sql
   SELECT w.last_name  AS worker,
          w.hire_date  AS worker_hire_date,
          m.last_name  AS manager,
          m.hire_date  AS manager_hire_date
   FROM   employees w
   JOIN   employees m
          ON w.manager_id = m.employee_id
   WHERE  w.hire_date < m.hire_date;
   ```

3. Save as `lab_07_09.sql` and run it.

These are the people who were hired before their bosses—which is sometimes fine, and sometimes a juicy little bit of org history.

---

## What This Lab Leaves You With

After completing these tasks, you should be able to:

- Use `NATURAL JOIN`, `USING`, and `ON` to join related tables.
- Perform self‑joins to relate employees to managers and colleagues.
- Use nonequijoins (`BETWEEN`) to map salaries to grade ranges.
- Combine joins with `WHERE` filters to answer questions like “who works in Toronto?” or “who was hired before their manager?”.

You’re now fully capable of making your normalized schema behave like a single, coherent source of truth—which is equal parts powerful and dangerous.
