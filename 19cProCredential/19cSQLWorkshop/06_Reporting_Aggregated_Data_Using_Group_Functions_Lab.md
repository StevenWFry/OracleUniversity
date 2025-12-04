## Lesson 6 Lab – Reporting Aggregated Data Using Group Functions (or: how to interrogate your data in bulk)

And look, there comes a moment when HR stops asking “what does this one employee make?” and starts asking “how much are we paying **all** of them?” or “which manager is secretly hoarding the payroll budget?”. That’s where group functions and `GROUP BY` earn their keep.

You will:

- Use `AVG`, `SUM`, `MIN`, `MAX`, and `COUNT`.
- Group by job, manager, and department.
- Use `HAVING` to filter groups on aggregated values.
- Build year and department “matrix” reports with `DECODE`.

Use the **`ora1`** connection and the **SQL1** labs folder.

---

## Quick Concept Check (6‑1)

These are the practice true/false statements and their answers:

1. **Group functions work across many rows to produce one result per group.**  
   → `TRUE` – that’s literally their job.

2. **Group functions include WHERE and calculations.**  
   → `FALSE` – `WHERE` is a clause, not a group function.

3. **The WHERE clause restricts rows before inclusion in a group calculation.**  
   → `TRUE` – `WHERE` filters rows before grouping.

Now on to the fun part: making the aggregates do real work.

---

## Task 4 – Max, Min, Sum, and Average Salary (All Employees)

**Goal:** Use multiple group functions and round the results.

HR wants:

- Highest salary (MAX)
- Lowest salary (MIN)
- Total salary (SUM)
- Average salary (AVG)
- All rounded to whole numbers.

1. New worksheet.
2. Query:

   ```sql
   SELECT ROUND(MAX(salary), 0) AS "Maximum",
          ROUND(MIN(salary), 0) AS "Minimum",
          ROUND(SUM(salary), 0) AS "Sum",
          ROUND(AVG(salary), 0) AS "Average"
   FROM   employees;
   ```

3. Save as `lab_06_04.sql` and run it.

If you see an “invalid identifier” error on the alias, double‑check your double quotes.

---

## Task 5 – Max/Min/Sum/Avg Salary by Job

**Goal:** Add `GROUP BY` so you get one aggregated row **per job type**.

1. Copy the query from Task 4 into a new worksheet.
2. Add `job_id` to the `SELECT` list and group by it:

   ```sql
   SELECT job_id,
          ROUND(MAX(salary), 0) AS "Maximum",
          ROUND(MIN(salary), 0) AS "Minimum",
          ROUND(SUM(salary), 0) AS "Sum",
          ROUND(AVG(salary), 0) AS "Average"
   FROM   employees
   GROUP  BY job_id;
   ```

3. Save as `lab_06_05.sql`.
4. Run it and confirm you now get one row per `job_id`.

This is your first “per job type” summary.

---

## Task 6 – Count Employees by Job, Then Prompt for a Job

### 6a – Count per job

**Goal:** Use `COUNT` and `GROUP BY` to see how many people have each job.

1. New worksheet.
2. Query:

   ```sql
   SELECT job_id,
          COUNT(*) AS job_count
   FROM   employees
   GROUP  BY job_id;
   ```

3. Run it. You should see counts: three `IT_PROG`, two `AD_VP`, one `AD_PRES`, etc.

### 6b – Prompt for a job and count only that one

Now make it interactive so HR can ask for one job at a time.

1. Add a `WHERE` clause with a substitution variable:

   ```sql
   SELECT job_id,
          COUNT(*) AS job_count
   FROM   employees
   WHERE  job_id = '&job_title'
   GROUP  BY job_id;
   ```

2. Save as `lab_06_06.sql`.
3. Run it and, when prompted for `job_title`, enter `IT_PROG`.
   - You should see a count of 3.

If you run it again with a different job ID, you’ll get its count instead.

---

## Task 7 – How Many Managers Do We Have?

**Goal:** Count **distinct** managers without listing them individually.

1. New worksheet.
2. Use `COUNT(DISTINCT ...)`:

   ```sql
   SELECT COUNT(DISTINCT manager_id) AS "Number of Managers"
   FROM   employees;
   ```

3. Run it; you should see something like `8`.

This ignores rows where `manager_id` is `NULL`.

---

## Task 8 – Difference Between Highest and Lowest Salary

**Goal:** Use `MAX` and `MIN` in a calculation.

1. New worksheet.
2. Compute `MAX(salary) - MIN(salary)` and label it `DIFFERENCE`:

   ```sql
   SELECT MAX(salary) - MIN(salary) AS DIFFERENCE
   FROM   employees;
   ```

3. Run it and note the gap between the best and worst paid employees.

---

## Task 9 – Lowest Paid Employee per Manager (with HAVING)

**Goal:**

- Show **manager_id** and **lowest salary** among their direct reports.
- Exclude managers that are `NULL`.
- Exclude managers whose lowest salary is **6000 or less**.
- Sort by that minimum salary in descending order.

1. New worksheet.
2. Start with the basic grouping:

   ```sql
   SELECT manager_id,
          MIN(salary) AS min_salary
   FROM   employees
   WHERE  manager_id IS NOT NULL
   GROUP  BY manager_id;
   ```

3. Add a `HAVING` clause to filter group results:

   ```sql
   SELECT manager_id,
          MIN(salary) AS min_salary
   FROM   employees
   WHERE  manager_id IS NOT NULL
   GROUP  BY manager_id
   HAVING MIN(salary) > 6000
   ORDER  BY min_salary DESC;
   ```

4. Run it; you should see only those managers whose **lowest‑paid** employee still earns more than 6000.

This is a great way to find managers whose teams are uniformly expensive.

---

## Task 10 – Count Hires per Year (2009–2012)

**Goal:** Produce a single‑row report showing:

- Total number of employees.
- Number hired in each year 2009, 2010, 2011, and 2012.

### 10a – Total employees

1. New worksheet.
2. Quick total:

   ```sql
   SELECT COUNT(*) AS total
   FROM   employees;
   ```

### 10b – Hires per year with DECODE

Now build one query that returns all counts in separate columns.

```sql
SELECT COUNT(*) AS total,
       SUM(DECODE(TO_CHAR(hire_date, 'YYYY'), '2009', 1, 0)) AS "2009",
       SUM(DECODE(TO_CHAR(hire_date, 'YYYY'), '2010', 1, 0)) AS "2010",
       SUM(DECODE(TO_CHAR(hire_date, 'YYYY'), '2011', 1, 0)) AS "2011",
       SUM(DECODE(TO_CHAR(hire_date, 'YYYY'), '2012', 1, 0)) AS "2012"
FROM   employees;
```

Explanation:

- `TO_CHAR(hire_date, 'YYYY')` converts hire dates to a 4‑digit year string.
- `DECODE(..., '2009', 1, 0)` returns `1` if the year matches, else `0`.
- `SUM` of those 1s/0s gives the count per year.

You should see counts like 2, 2, 3, 2 (depending on your sample data).

---

## Task 11 – Salary Matrix by Job and Department (20, 50, 80, 90)

**Goal:** Build a cross‑tab (matrix) report showing:

- One row per job.
- Columns for total salary in departments **20, 50, 80, and 90**.
- A final column for the **total salary per job** across those departments.

1. New worksheet.
2. Use `DECODE` to conditionally include salaries by department:

   ```sql
   SELECT job_id                          AS job,
          SUM(DECODE(department_id, 20, salary, 0)) AS dept_20,
          SUM(DECODE(department_id, 50, salary, 0)) AS dept_50,
          SUM(DECODE(department_id, 80, salary, 0)) AS dept_80,
          SUM(DECODE(department_id, 90, salary, 0)) AS dept_90,
          SUM(salary)                              AS job_total
   FROM   employees
   WHERE  department_id IN (20, 50, 80, 90)
   GROUP  BY job_id
   ORDER  BY job_id;
   ```

3. Run it and inspect:
   - Each row is a job.
   - Each department column shows the total salaries for that job in that department.
   - `job_total` is the combined salary across all four departments.

This is how you build a budget slide that simultaneously informs and horrifies.

---

## What This Lab Leaves You With

After completing these tasks, you should be able to:

- Use `AVG`, `SUM`, `MIN`, `MAX`, and `COUNT` to summarize data.
- Group results by columns using `GROUP BY` and understand the “must be in GROUP BY or aggregated” rule.
- Use `HAVING` to filter groups based on aggregated values.
- Build more complex reports such as year‑by‑year counts and department/job salary matrices using `DECODE`.

You can now answer questions like “Which manager has the most expensive team?” or “How much does department 80 really cost us?”—which is powerful, but also exactly how budgets get cut.
