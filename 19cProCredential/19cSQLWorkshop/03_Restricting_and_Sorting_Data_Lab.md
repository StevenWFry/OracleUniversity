## Lesson 3 Lab – Restricting and Sorting Data (or: teaching your queries to stop oversharing)

And look, your `SELECT` statements are currently oversharing like a social network with no privacy settings. This lab fixes that. You’ll practice **filtering**, **sorting**, and throwing in **substitution variables** so HR can ask “who’s overpaid?” without calling you every five minutes.

You will:

- Filter rows using `WHERE`, comparison operators, `BETWEEN`, `IN`, `LIKE`, `IS NULL`, and `IS NOT NULL`.
- Sort rows with `ORDER BY` (by name, salary, column position, and more).
- Use `DISTINCT` and column aliases.
- Use substitution variables to prompt for thresholds and manager IDs.
- Save queries into `lab_03_XX.sql` scripts in the SQL1 labs folder.

Use the **`ora1`** connection and the **SQL1** labs for this lesson.

---

## Task 1 – Employees Earning More Than 12,000

**Goal:** Simple filter and saving to a script file.

1. In SQL Developer, open a new worksheet on your `ora1` connection.
2. Write a query to display last name and salary for employees who earn **more than 12000**:

   ```sql
   SELECT last_name,
          salary
   FROM   employees
   WHERE  salary > 12000;
   ```

3. Save it as `lab_03_01.sql` in the SQL1 labs folder.
4. Re‑open the file and run it to confirm it works.

---

## Task 2 – Department for Employee 176

**Goal:** Filter by a specific employee ID.

1. New worksheet.
2. Show last name and department of employee **176**:

   ```sql
   SELECT last_name,
          department_id
   FROM   employees
   WHERE  employee_id = 176;
   ```

3. Verify you get Taylor in department 80 (assuming standard HR data).

---

## Task 3 – High and Low Salaries (NOT BETWEEN)

**Goal:** Reuse an earlier query and invert the salary range.

1. Open `lab_03_01.sql`.
2. Modify it to show employees whose salary is **not** between 5000 and 12000:

   ```sql
   SELECT last_name,
          salary
   FROM   employees
   WHERE  salary NOT BETWEEN 5000 AND 12000;
   ```

3. Save as `lab_03_03.sql`.
4. Run it; you should see only the high and low outliers.

Keep this file open—you’ll reuse it later.

---

## Task 4 – Matos and Taylor by Hire Date

**Goal:** Multiple columns, filter by last name, sort by date.

1. New worksheet.
2. Create a report showing last name, job ID, and hire date for employees **Matos** and **Taylor**, ordered by hire date:

   ```sql
   SELECT last_name,
          job_id,
          hire_date
   FROM   employees
   WHERE  last_name IN ('Matos', 'Taylor')
   ORDER  BY hire_date ASC;
   ```

(You can omit `ASC` if you like; ascending is the default.)

---

## Task 5 – Employees in Departments 20 or 50

**Goal:** Use `IN` and sort by name.

1. New worksheet.
2. Show last name and department ID for employees in **departments 20 or 50**, sorted by last name:

   ```sql
   SELECT last_name,
          department_id
   FROM   employees
   WHERE  department_id IN (20, 50)
   ORDER  BY last_name ASC;
   ```

Enjoy the feeling of not writing `OR` three times.

---

## Task 6 – Employees in 20 or 50 Earning 5,000–12,000

**Goal:** Combine `BETWEEN`, `IN`, and aliases.

1. Start from `lab_03_03.sql` and adapt it.
2. Create a query that shows **last name** and **salary** for employees:

   - Salary **between 5000 and 12000**, and
   - `department_id` in (20, 50).

3. Give the columns readable aliases:

   ```sql
   SELECT last_name AS "Employee",
          salary    AS "Monthly Salary"
   FROM   employees
   WHERE  salary BETWEEN 5000 AND 12000
   AND    department_id IN (20, 50);
   ```

4. Save as `lab_03_06.sql` and run it.

---

## Task 7 – Employees Hired in 2010

**Goal:** Filter on a date range.

1. New worksheet.
2. Display last name and hire date for employees hired in **2010**.
3. Use a date range that includes the entire year:

   ```sql
   SELECT last_name,
          hire_date
   FROM   employees
   WHERE  hire_date >= DATE '2010-01-01'
   AND    hire_date <  DATE '2011-01-01';
   ```

(If you use Oracle’s `DD-MON-RR` format, be sure you get the dates right.)

---

## Task 8 – Employees Without a Manager

**Goal:** Use `IS NULL`.

1. New worksheet.
2. Show last name and job ID of employees who **do not have a manager**:

   ```sql
   SELECT last_name,
          job_id
   FROM   employees
   WHERE  manager_id IS NULL;
   ```

You should see just the top boss.

---

## Task 9 – Commissioned Employees, Sorted by Position

**Goal:** Use `IS NOT NULL` and `ORDER BY` column positions.

1. New worksheet.
2. Show last name, salary, and commission percentage for employees **who earn commissions**:

   ```sql
   SELECT last_name,
          salary,
          commission_pct
   FROM   employees
   WHERE  commission_pct IS NOT NULL
   ORDER  BY 2 DESC, 3 DESC;
   ```

Here:

- `2` = `salary` (sorted descending).
- `3` = `commission_pct` (also descending, used when salaries tie).

---

## Task 10 – Flexible Minimum Salary (Substitution Variable)

**Goal:** Use a substitution variable so HR can choose the salary threshold.

1. Open `lab_03_01.sql` and copy its query into a new worksheet.
2. Replace the hardcoded `12000` with a substitution variable:

   ```sql
   SELECT last_name,
          salary
   FROM   employees
   WHERE  salary > &min_salary;
   ```

3. Save as `lab_03_10.sql` and run it.
4. When prompted, enter `12000`.
   - Verify the output matches the “earn more than 12000” requirement.

You’ve just made the “who makes more than X?” report reusable.

---

## Task 11 – Report by Manager with Sort Choice

**Goal:** Use multiple substitution variables and dynamic ordering.

HR wants a report that prompts for:

- A **manager ID**.
- A **sort column** to order by.

The report should show:

- `employee_id`, `last_name`, `salary`, `department_id` for that manager’s employees.

1. New worksheet.
2. Write a query like:

   ```sql
   SELECT employee_id,
          last_name,
          salary,
          department_id
   FROM   employees
   WHERE  manager_id = &mgr_id
   ORDER  BY &order_col;
   ```

3. Test with the sample values from the guide, such as:
   - `mgr_id` = 103, `order_col` = `last_name`
   - `mgr_id` = 201, `order_col` = `salary`
   - `mgr_id` = 124, `order_col` = `employee_id`

You can rename the variables (`&order_col`, `&sort_col`, etc.); just be consistent.

If you’re annoyed by being prompted multiple times for the same column, use `&&` once and plain `&` later, plus `UNDEFINE` when you want to reset.

---

## Optional “If You Have Time” Tasks

These exercises tighten your grip on patterns and combined conditions.

### Task 12 – Third Letter is a Lowercase `a`

**Goal:** Use `_` wildcard with `LIKE`.

```sql
SELECT last_name
FROM   employees
WHERE  last_name LIKE '__a%';
```

- `__` = any two characters.
- `a`  = third character must be lowercase `a`.
- `%`  = any trailing characters.

### Task 13 – Last Names Containing Both `a` and `e`

**Goal:** Combine two pattern conditions.

```sql
SELECT last_name
FROM   employees
WHERE  last_name LIKE '%a%'
AND    last_name LIKE '%e%';
```

Everyone returned has at least one `a` and one `e` in their last name.

### Task 14 – Sales Reps and Stock Clerks, with Awkward Salaries Removed

**Goal:** Combine job filters and salary exclusions.

Requirements:

- Jobs: **sales rep** (`SA_REP`) or **stock clerk** (`ST_CLERK`).
- Salary is **not** 2500, 3500, or 7000.

```sql
SELECT last_name,
       job_id,
       salary
FROM   employees
WHERE  job_id IN ('SA_REP', 'ST_CLERK')
AND    salary NOT IN (2500, 3500, 7000);
```

Check that only the requested job types remain, with the awkward salaries filtered out.

### Task 15 – 20% Commission from lab_03_06

**Goal:** Reuse a prior query, change the filter, and save a new script.

1. Open `lab_03_06.sql`.
2. Copy its query into a new worksheet.
3. Modify it to show **last name**, **salary**, and **commission_pct** for employees whose commission is exactly **0.2 (20%)**:

   ```sql
   SELECT last_name,
          salary,
          commission_pct
   FROM   employees
   WHERE  commission_pct = 0.2;
   ```

4. Save as `lab_03_15.sql`.
5. Run it and verify all rows show `0.2` for `COMMISSION_PCT`.

---

## What This Lab Leaves You With

After completing these tasks, you should be able to:

- Filter rows precisely using `WHERE`, `BETWEEN`, `IN`, `LIKE`, and `NULL` checks.
- Sort data using `ORDER BY` on columns, aliases, and positions.
- Use substitution variables to make reports interactive and reusable.
- Build targeted HR reports (by salary, department, hire date, manager, and commission).

In other words, your queries now have opinions about **who** they return and **in what order**—which is exactly what you want, even if some of those opinions are “no one under 12,000, thank you very much.”
