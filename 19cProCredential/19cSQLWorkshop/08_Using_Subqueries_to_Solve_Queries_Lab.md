## Lesson 8 Lab – Using Subqueries to Solve Queries (or: letting your query do its own research)

And look, there’s only so far you can get with hard‑coded values. Eventually HR wants things like “anyone who earns more than the average” or “everyone who works with whoever you just typed”. That’s when you stop guessing numbers and let **subqueries** figure them out.

You will:

- Use subqueries to look up unknown criteria (dates, salaries, departments).
- Use `IN`, `>`, and `> ANY` with subqueries.
- Combine multiple subqueries in a single `WHERE` clause.
- Use substitution variables with subqueries for flexible filters.

Use the **`ora1`** connection and the **SQL1** labs folder.

---

## Task 1 – Coworkers of a Given Employee (Prompted Name)

**Goal:** Prompt for an employee’s last name and list coworkers in the **same department**, excluding that employee.

Example: If the user enters `Zlotkey`, show everyone who works in Zlotkey’s department **except** Zlotkey.

1. Start by hard‑coding the name to design the query:

   ```sql
   SELECT last_name,
          hire_date
   FROM   employees
   WHERE  department_id = (
             SELECT department_id
             FROM   employees
             WHERE  last_name = 'Zlotkey'
          )
   AND    last_name <> 'Zlotkey';
   ```

2. Once that works, swap `'Zlotkey'` for a substitution variable and reuse it in both places:

   ```sql
   SELECT last_name,
          hire_date
   FROM   employees
   WHERE  department_id = (
             SELECT department_id
             FROM   employees
             WHERE  last_name = &&name
          )
   AND    last_name <> '&name';

   UNDEFINE name
   ```

3. Run the script (`Run Script` / F5) so the `UNDEFINE` executes as well.
4. When prompted for `name`, type `Zlotkey`, then try again with `Abel` or others.

- `&&name` prompts once and remembers the value.
- `UNDEFINE name` clears it so you’re prompted next time.

---

## Task 2 – Employees Earning More Than the Average Salary

**Goal:** Use a subquery with `AVG` to find everyone whose salary is **above average**.

1. First, find the average salary:

   ```sql
   SELECT AVG(salary)
   FROM   employees;
   ```

2. Now use that in a subquery:

   ```sql
   SELECT employee_id,
          last_name,
          salary
   FROM   employees
   WHERE  salary > (
             SELECT AVG(salary)
             FROM   employees
          )
   ORDER  BY salary;
   ```

This returns only those employees whose `salary` is greater than the overall average.

---

## Task 3 – Employees in Departments with a “u” in Someone’s Last Name – `lab_08_03.sql`

**Goal:** Find employees who work in a department where **someone’s** last name contains a lowercase `u`.

1. New worksheet.
2. Query:

   ```sql
   SELECT employee_id,
          last_name,
          salary
   FROM   employees
   WHERE  department_id IN (
             SELECT DISTINCT department_id
             FROM   employees
             WHERE  last_name LIKE '%u%'
          );
   ```

3. Save as `lab_08_03.sql` and run it.

This returns everyone in any department that has at least one `u` in a last name.

---

## Task 4 – Employees in a Location (Prompted Location ID) – `lab_08_04.sql`

**Goal:** Use a subquery against `DEPARTMENTS` to filter `EMPLOYEES` by `location_id`.

Requirements:

- Show: `last_name`, `department_id`, `job_id`.
- Departments must have a given `location_id` (e.g., 1700).
- Prompt for the location ID.

1. New worksheet.
2. Query:

   ```sql
   SELECT last_name,
          department_id,
          job_id
   FROM   employees
   WHERE  department_id IN (
             SELECT department_id
             FROM   departments
             WHERE  location_id = &loc_id
          );
   ```

3. Save as `lab_08_04.sql` and run it.
4. When prompted, enter `1700` (or another valid location).

---

## Task 5 – Employees Who Report to King

**Goal:** Use a subquery to find King’s `employee_id`, then list everyone whose `manager_id` equals that value.

1. New worksheet.
2. Subquery to identify King:

   ```sql
   SELECT employee_id
   FROM   employees
   WHERE  last_name = 'King';
   ```

3. Use it inline:

   ```sql
   SELECT last_name,
          salary
   FROM   employees
   WHERE  manager_id = (
             SELECT employee_id
             FROM   employees
             WHERE  last_name = 'King'
          );
   ```

This should show the employees who report directly to King.

---

## Task 6 – Employees in the Executive Department

**Goal:** Use a subquery to map department **name** (`Executive`) to its `department_id`.

Requirements:

- Show: `department_id`, `last_name`, `job_id` for employees in department `Executive`.

1. New worksheet.
2. Query:

   ```sql
   SELECT department_id,
          last_name,
          job_id
   FROM   employees
   WHERE  department_id IN (
             SELECT department_id
             FROM   departments
             WHERE  department_name = 'Executive'
          );
   ```

3. Run it; you should see employees in department 90 (in the standard HR data).

---

## Task 7 – Employees Who Earn More Than Any Salary in Department 60

**Goal:** Use a multiple‑row subquery with `> ANY`.

Requirements:

- Show employees whose salary is **greater than any** salary found in department 60.

1. New worksheet.
2. First, inspect salaries in department 60:

   ```sql
   SELECT salary
   FROM   employees
   WHERE  department_id = 60;
   ```

3. Use `> ANY`:

   ```sql
   SELECT last_name,
          salary
   FROM   employees
   WHERE  salary > ANY (
             SELECT salary
             FROM   employees
             WHERE  department_id = 60
          );
   ```

This returns employees whose salary is greater than at least one salary in department 60 (which effectively means greater than the **minimum** if the list is non‑empty).

---

## Task 8 – More Than Average Salary AND in a “u” Department – `lab_08_08.sql`

**Goal:** Combine multiple subqueries: employees who **earn more than the average salary** and **work in a department** with any employee whose last name includes `u`.

1. Base your query on `lab_08_03.sql`.
2. Add a second condition using the AVG subquery from Task 2:

   ```sql
   SELECT employee_id,
          last_name,
          salary
   FROM   employees
   WHERE  department_id IN (
             SELECT DISTINCT department_id
             FROM   employees
             WHERE  last_name LIKE '%u%'
          )
   AND    salary > (
             SELECT AVG(salary)
             FROM   employees
          );
   ```

3. Save as `lab_08_08.sql` and run it.

This final query returns only employees who:

- Work in a department that has **at least one `u`‑containing last name**, and
- Earn **more than the company‑wide average salary**.

---

## What This Lab Leaves You With

After completing these tasks, you should be able to:

- Use subqueries to look up unknown values (dates, salaries, department IDs, manager IDs).
- Write single‑row subqueries with `=` and `>` and multiple‑row ones with `IN` and `> ANY`.
- Combine multiple subqueries in a single `WHERE` clause.
- Use substitution variables with subqueries for flexible, reusable filters.

Your queries can now **discover their own criteria** instead of relying on guesswork or copy‑paste from previous results—vastly improving both correctness and your dignity.
