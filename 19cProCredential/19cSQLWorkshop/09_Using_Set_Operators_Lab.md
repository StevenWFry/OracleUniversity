## Lesson 9 Lab – Using Set Operators (or: refereeing arguments between result sets)

And look, sometimes the question isn’t “what’s in this table?” but “what’s in **this** query and **not** in **that** one?” or “who’s in **either**?” or “who’s in **both**?”. That’s when you stop thinking row‑by‑row and start thinking in **sets**.

You will:

- Use `UNION` and `UNION ALL` to combine results.
- Use `INTERSECT` to find overlap.
- Use `MINUS` to find differences.
- Make sure column counts and data types line up.

Use the **`ora1`** connection and the **SQL1** labs folder.

---

## Task 1 – Departments Without ST_CLERK

**Goal:** List `department_id`s for departments that **do not** contain job `ST_CLERK`, using `MINUS`.

1. New worksheet.
2. Query:

   ```sql
   SELECT department_id
   FROM   departments

   MINUS

   SELECT department_id
   FROM   employees
   WHERE  job_id = 'ST_CLERK';
   ```

3. Run it; the result is the set of departments where nobody has job `ST_CLERK`.

---

## Task 2 – Countries with No Departments

**Goal:** List countries that have **no departments located in them**, using a set difference.

1. Start by listing all countries:

   ```sql
   SELECT country_id,
          country_name
   FROM   countries;
   ```

2. Build a subquery that finds countries **with** departments by joining `LOCATIONS`, `COUNTRIES`, and `DEPARTMENTS`:

   ```sql
   SELECT DISTINCT c.country_id,
                   c.country_name
   FROM   locations   l
   JOIN   countries   c ON l.country_id = c.country_id
   JOIN   departments d ON d.location_id = l.location_id;
   ```

3. Now subtract:

   ```sql
   SELECT country_id,
          country_name
   FROM   countries

   MINUS

   SELECT DISTINCT c.country_id,
                   c.country_name
   FROM   locations   l
   JOIN   countries   c ON l.country_id = c.country_id
   JOIN   departments d ON d.location_id = l.location_id;
   ```

4. Run it; you should see only the countries that have **no** departments.

---

## Task 3 – Employees in Departments 50 and 80 (UNION ALL)

**Goal:** Use two queries and `UNION ALL` to list everyone in departments 50 and 80.

Requirements:

- Show: `employee_id`, `job_id`, `department_id`.

1. New worksheet.
2. Query:

   ```sql
   SELECT employee_id,
          job_id,
          department_id
   FROM   employees
   WHERE  department_id = 50

   UNION ALL

   SELECT employee_id,
          job_id,
          department_id
   FROM   employees
   WHERE  department_id = 80;
   ```

3. Run it; this returns all employees in departments 50 **and** 80, with no deduping.

You could do this with a single `WHERE department_id IN (50,80)`, but the point here is practicing set operators.

---

## Task 4 – Sales Reps in the Sales Department (INTERSECT)

**Goal:** Use `INTERSECT` to find employees who are **both** sales reps (`SA_REP`) and in department 80 (Sales).

1. New worksheet.
2. Use `employee_id` as the common key:

   ```sql
   SELECT employee_id
   FROM   employees
   WHERE  job_id = 'SA_REP'

   INTERSECT

   SELECT employee_id
   FROM   employees
   WHERE  department_id = 80;
   ```

3. Run it; you’ll get the `employee_id`s of employees who satisfy **both** conditions.
4. If you want names as well, you can wrap this in an outer query:

   ```sql
   SELECT employee_id,
          last_name
   FROM   employees
   WHERE  employee_id IN (
             SELECT employee_id
             FROM   employees
             WHERE  job_id = 'SA_REP'

             INTERSECT

             SELECT employee_id
             FROM   employees
             WHERE  department_id = 80
          );
   ```

---

## Task 5 – Combined View of Employees and Departments (UNION with Placeholders)

**Goal:** Create a single report that shows:

- All employees: last name + department_id, with an empty department name.
- All departments: department_id + department_name, with an empty last name.

Requirements:

- Use `UNION` to combine them.
- Align column counts and data types.

1. New worksheet.
2. First part – employees (last name, department, null dept name):

   ```sql
   SELECT last_name,
          department_id,
          TO_CHAR(NULL) AS dept_name
   FROM   employees
   ```

3. Second part – departments (null last name, department id, real dept name):

   ```sql
   UNION

   SELECT TO_CHAR(NULL) AS last_name,
          department_id,
          department_name AS dept_name
   FROM   departments;
   ```

Full query:

```sql
SELECT last_name,
       department_id,
       TO_CHAR(NULL) AS dept_name
FROM   employees

UNION

SELECT TO_CHAR(NULL) AS last_name,
       department_id,
       department_name AS dept_name
FROM   departments;
```

4. Run it; you’ll see one combined result set with:

- Employee rows (last name + department_id, no department_name).
- Department rows (no last name, department_id + department_name).

Because this uses `UNION`, duplicate rows (if any) would be removed and the output sorted by `last_name` (the first column) by default.

---

## What This Lab Leaves You With

After completing these tasks, you should be able to:

- Use `MINUS` to remove one result set from another.
- Use `UNION` and `UNION ALL` to combine results.
- Use `INTERSECT` to find the overlap between result sets.
- Align column counts and types across component queries, adding placeholder columns when needed.

You can now answer questions like “which departments don’t have certain jobs?”, “which countries don’t have departments?”, and “who exactly is in both groups?”, which is the SQL equivalent of checking overlapping friend circles.
