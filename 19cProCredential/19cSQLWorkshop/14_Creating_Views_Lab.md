# Lesson 14 Lab – Creating Views (Now With Fewer Lawsuits Than Just Granting Everyone `SELECT` On Everything)

And look, sometimes you don’t want to give people access to the **real** table.  
You want to give them a carefully curated illusion of truth – like a social‑media profile, but for data.  
That is exactly what this lab is about: weaponising views to show people *just enough* without letting them wreck the actual tables.

In this lab you will:

- Build a simple projection view over `EMPLOYEES`
- Build a filtered view with `WITH CHECK OPTION` and watch it yell at you
- Peek at view definitions in `USER_VIEWS`
- Drop the views when you’re done lying, responsibly

All of this is in the `ora21` / `sql2` environment.

---

## Task 1 – Create a polite façade over `EMPLOYEES`

HR would like a version of `EMPLOYEES` that shows:

- Employee number  
- Employee last name (renamed to `EMPLOYEE`)  
- Department number  

and hides literally everything else that might cause awkward conversations.

```sql
CREATE OR REPLACE VIEW employees_vu AS
SELECT employee_id,
       last_name   AS employee,
       department_id
FROM   employees;
```

Verify that the view actually returns data and not just good intentions:

```sql
SELECT * 
FROM   employees_vu;
-- Expect 107 rows (same count as EMPLOYEES)
```

HR’s favourite report:

```sql
SELECT employee,
       department_id
FROM   employees_vu;
```

Congratulations: you’ve successfully built a corporate filter bubble.

---

## Task 2 – Build a “you may not leave” view for department 80

Department 80 wants a view that:

- Only shows their people  
- Renames the columns for nostalgia’s sake: `EMPNO`, `EMPLOYEE`, `DEPTNO`  
- **Prevents** anyone from reassigning those employees to another department *through the view*

So basically a tiny authoritarian regime backed by SQL.

```sql
CREATE VIEW dept80 AS
SELECT employee_id   AS empno,
       last_name     AS employee,
       department_id AS deptno
FROM   employees
WHERE  department_id = 80
WITH CHECK OPTION CONSTRAINT emp_dept_80;
```

Check the structure:

```sql
DESC dept80;
```

And the data:

```sql
SELECT * 
FROM   dept80;
```

Now try to smuggle Abel out of department 80:

```sql
UPDATE dept80
SET    deptno = 50
WHERE  employee = 'Abel';
```

You should get a glorious error along the lines of:

> ORA-01402: view WITH CHECK OPTION where-clause violation

Translation: *“Nice try, but no one escapes department 80 through this view.”*

---

## Task 3 – Use the provided script to create `DEPT50`

Because nothing says “enterprise” like making you run yet another script.

Run the course script to create a similar view for department 50:

```sql
@/home/oracle/labs/sql2/labs/lab_14_07.sql
-- Creates view DEPT50
```

Confirm that it’s alive:

```sql
DESC dept50;

SELECT * 
FROM   dept50;
```

If you see rows and not error messages, you’re winning.

---

## Task 4 – Stare directly into `USER_VIEWS`

And look, views are just stored queries with nice names.  
Oracle actually keeps the view text in `USER_VIEWS`, so you can see exactly what someone wrapped in that “harmless” abstraction.

List all views you own and their definitions:

```sql
SELECT view_name,
       text
FROM   user_views;
```

Tips:

- In SQL Developer, use **Run Script (F5)** so the entire `TEXT` column prints instead of making you side‑scroll like it’s 1998.
- You should see at least:
  - `EMPLOYEES_VU`
  - `DEPT80`
  - `DEPT50`
  - `EMP_DETAILS_VIEW` (prebuilt in the schema)

If any of the view definitions surprise you, congratulations, you now understand why DBAs don’t trust developers.

---

## Task 5 – Drop the evidence

Views don’t store data; they just point at tables.  
Which means you can drop them without deleting the underlying data – like uninstalling the app without touching the database it was quietly talking to.

Clean up:

```sql
DROP VIEW employees_vu;
DROP VIEW dept80;
DROP VIEW dept50;
```

Double‑check they’re gone:

```sql
SELECT view_name
FROM   user_views;
```

If they no longer appear, the lies are gone, but the base tables remain – ready for the next time someone decides the best way to handle access control is “create another view and hope for the best.”

Lesson 14 Lab – Creating Views
In this lab you:

Create simple and filtered views
Add WITH CHECK OPTION to stop sneaky updates
Peek at view definitions in the data dictionary
Drop the views when you’re done lying to people
Assume you’re connected as ora21 in the sql2 environment.

Task 1 – Create a basic projection view
Hide most of EMPLOYEES, just expose IDs, names, and departments.

CREATE OR REPLACE VIEW employees_vu AS
SELECT employee_id,
       last_name AS employee,
       department_id
FROM   employees;
Verify:

SELECT * FROM employees_vu;
-- Expect 107 rows
And a simpler report:

SELECT employee, department_id
FROM   employees_vu;
Task 2 – Create a filtered view with WITH CHECK OPTION
Department 80 wants “their” slice of the data, and they really don’t want you quietly reassigning people through the view.

CREATE VIEW dept80 AS
SELECT employee_id   AS empno,
       last_name     AS employee,
       department_id AS deptno
FROM   employees
WHERE  department_id = 80
WITH CHECK OPTION CONSTRAINT emp_dept_80;
Inspect structure and contents:

DESC dept80;

SELECT * FROM dept80;
Now attempt to “promote” Abel to department 50 through the view:

UPDATE dept80
SET    deptno = 50
WHERE  employee = 'Abel';
You should get a WITH CHECK OPTION where-clause violation error, proving the guard rail is working.

Task 3 – Create another departmental view from a script
Run the provided script to build a similar view for department 50:

@/home/oracle/labs/sql2/labs/lab_14_07.sql
-- Creates view DEPT50
Confirm it exists and returns rows:

DESC dept50;
SELECT * FROM dept50;
Task 4 – Spy on your views via the data dictionary
Use USER_VIEWS to see every view you own and the SQL that defines it:

SELECT view_name,
       text
FROM   user_views;
Notes:

In SQL Developer, use Run Script (F5) to see the full TEXT column without playing horizontal-scroll roulette.
You should see at least EMPLOYEES_VU, DEPT80, DEPT50, and EMP_DETAILS_VIEW (the last one is prebuilt in the schema).
Task 5 – Clean up your mess (drop the views)
When HR is done experimenting with carefully curated reality, drop the views:

DROP VIEW employees_vu;
DROP VIEW dept80;
DROP VIEW dept50;
Re‑run the USER_VIEWS query to confirm they’re gone:

SELECT view_name
FROM   user_views;
If it’s empty or only shows the prebuilt views, congratulations: your views are gone, but the underlying tables are still very much there, waiting for the next round of creative obfuscation.