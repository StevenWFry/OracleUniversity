## Lesson 2 Lab – Retrieving Data Using the SQL SELECT Statement (or: your first real interrogation of the HR tables)

And look, now that you’ve met `SELECT`, it’s time to actually use it on something other than contrived examples. In this lab you’ll poke at the HR schema, make mistakes on purpose, and then fix them before anyone from HR notices.

You will:

- Run simple `SELECT` statements and see what “successful” actually means.
- Debug a broken `SELECT` with multiple errors.
- Describe table structures with `DESCRIBE` / `DESC`.
- Retrieve all rows and columns from tables.
- Use `DISTINCT`, aliases, and concatenation.
- Save a query to a `.sql` file and test it.

Use the **`ora1`** connection in SQL Developer and the **SQL1** labs folder for this lesson.

---

## Task 1 – True/False: Do These SELECTs Actually Work?

**Goal:** Warm up by checking which statements are valid.

1. In SQL Developer, open a new SQL Worksheet for your `ora1` connection.
2. Run the following statement:

   ```sql
   SELECT last_name, salary
   FROM   employees;
   ```

   - If it runs without error and returns rows, mark it **TRUE** in your Activity Guide.

3. Replace it with the second practice statement from the guide and execute it as well.
   - If it runs successfully, mark that one **TRUE** too.

These are the easy questions. Enjoy them while they last.

---

## Task 2 – Debugging a Broken SELECT (Find the Four Errors)

**Goal:** Practice reading error messages and fixing basic mistakes.

1. Copy the “broken” `SELECT` statement from the Activity Guide into your worksheet.
2. Run it.
   - You should see an error like: `FROM keyword not found where expected` pointing at line 2.
3. Use the error position and your knowledge of the `EMPLOYEES` table to find the four problems. They include:
   - A **missing comma** between column names.
   - A **wrong column name** (e.g., `sal` instead of `salary`).
   - An **invalid operator** (not using `*` for multiplication).
   - An alias containing a **space** with **no double quotes**.
4. Fix all four issues and re‑run the statement until it executes cleanly.

If you can’t fix this kind of statement yet, your future debugging self will be *very* unhappy.

---

## Task 3 – Exploring DEPARTMENTS and EMPLOYEES

**Goal:** Learn how to inspect table structures and basic contents.

### 3a – Structure of `DEPARTMENTS`

1. In your worksheet, run:

   ```sql
   DESCRIBE departments;
   ```

2. Confirm you can see:
   - Column names
   - Data types
   - Nullability (whether `NULL` is allowed)

### 3b – Data in `DEPARTMENTS`

1. Run:

   ```sql
   SELECT *
   FROM   departments;
   ```

2. Note how many rows are returned (about eight in the standard HR sample).

### 3c – Structure of `EMPLOYEES`

1. Run:

   ```sql
   DESCRIBE employees;
   ```

2. Verify the presence of columns like `EMPLOYEE_ID`, `LAST_NAME`, `JOB_ID`, `HIRE_DATE`, `SALARY`, `COMMISSION_PCT`, `DEPARTMENT_ID`, etc.

This is your basic “what’s in this table?” toolkit.

---

## Task 4 – Employee Report with Aliases (and Save It to a File)

**Goal:** Create a small report query, then save it as a script file the HR team could theoretically use.

The HR department wants:

- Columns: `EMPLOYEE_ID`, `LAST_NAME`, `JOB_ID`, `HIRE_DATE`.
- `EMPLOYEE_ID` should appear **first**.
- `HIRE_DATE` should have the alias `StartDate`.

### 4a – Write the query

In a new worksheet, write:

```sql
SELECT employee_id,
       last_name,
       job_id,
       hire_date AS StartDate
FROM   employees;
```

Run it and confirm it works.

### 4b – Save as `lab_02_5b.sql`

1. Save this worksheet to:
   - `~/labs/sql1/labs/lab_02_5b.sql`  
     (or the exact path specified in your Activity Guide).
2. Close the worksheet.
3. Re‑open `lab_02_5b.sql` from disk and run it again to confirm the saved script works.

Congratulations, you’ve just created something HR could run without you standing next to them.

---

## Task 5 – Unique Job IDs Only

**Goal:** Use `DISTINCT` to eliminate duplicates.

The HR department wants a list of **all unique job IDs** in the company.

1. In a new worksheet, run the “plain” version:

   ```sql
   SELECT job_id
   FROM   employees;
   ```

   - Note the duplicate job IDs.

2. Now run the cleaned‑up version:

   ```sql
   SELECT DISTINCT job_id
   FROM   employees;
   ```

3. Confirm that each `JOB_ID` now appears **only once**.

`DISTINCT` is powerful—just remember it applies to the *whole* column list, not individually per column.

---

## Task 6 – Nicer Column Headings with Aliases (Optional “If You Have Time”) 

**Goal:** Practice using column aliases to produce human‑friendly headings.

1. Re‑open `lab_02_5b.sql` and copy the `SELECT` statement into a new worksheet.
2. Change the column list to use these aliases:

   - `EMPLOYEE_ID` → `"Emp #"`
   - `LAST_NAME`   → `"Employee"`
   - `JOB_ID`      → `job` (or `"Job"`)
   - `HIRE_DATE`   → `"Hire Date"`

   Example:

   ```sql
   SELECT employee_id AS "Emp #",
          last_name   AS "Employee",
          job_id      AS job,
          hire_date   AS "Hire Date"
   FROM   employees;
   ```

3. Run the statement and confirm the headings match the requested aliases.

This is the difference between “technically correct” output and something a human being might actually read.

---

## Task 7 – Concatenated Employee and Job Title

**Goal:** Combine columns and literals with the concatenation operator.

HR wants a report showing each employee and their job in a single column, formatted like:

> `Abel, SA_REP`

1. Write and run:

   ```sql
   SELECT last_name || ', ' || job_id AS "Employee and Title"
   FROM   employees;
   ```

2. Verify that the output uses one column with last name, comma, space, and job ID.

You’ve just created a tiny, text‑only form of an org report.

---

## Task 8 – THE_OUTPUT: All Columns, One Giant String

**Goal:** Get familiar with the data in `EMPLOYEES` and stretch your concatenation muscles.

You’re asked to:

- Display **all data** from the `EMPLOYEES` table.
- Separate each column’s value with a **comma and space**.
- Put everything into a single column called `THE_OUTPUT`.

### 8a – Start with a regular SELECT

1. Use the Connections tree to drag `EMPLOYEES` into the worksheet to generate a basic `SELECT *` template, or type the columns manually.
2. Make sure the columns are in the order specified in the Activity Guide (e.g., `EMPLOYEE_ID`, `FIRST_NAME`, `LAST_NAME`, `EMAIL`, `PHONE_NUMBER`, `HIRE_DATE`, `JOB_ID`, `MANAGER_ID`, `SALARY`, `COMMISSION_PCT`, `DEPARTMENT_ID`).
3. Run the normal multi‑column `SELECT` to confirm no syntax errors.

### 8b – Turn it into one concatenated column

1. Replace the column list with a single expression that concatenates all columns, for example:

   ```sql
   SELECT employee_id      || ', ' ||
          first_name       || ', ' ||
          last_name        || ', ' ||
          email            || ', ' ||
          phone_number     || ', ' ||
          hire_date        || ', ' ||
          job_id           || ', ' ||
          manager_id       || ', ' ||
          salary           || ', ' ||
          commission_pct   || ', ' ||
          department_id    AS THE_OUTPUT
   FROM   employees;
   ```

2. Run the query and inspect `THE_OUTPUT`.
   - Each row now appears as a single long text line containing all values.

Is this how you’d build a real report? Probably not. But it’s fantastic practice for concatenation and data inspection.

---

## What This Lab Gave You

After finishing this lab, you should be able to:

- Recognize valid and invalid `SELECT` statements and fix common coding errors.
- Use `DESCRIBE` / `DESC` and `SELECT *` to explore tables.
- Write simple `SELECT` queries with specific columns and aliases.
- Use `DISTINCT` to remove duplicates from result sets.
- Concatenate columns and literals to build readable output.
- Save and re‑run a query from a `.sql` file in SQL Developer.

Which means you’re now fully equipped to start writing queries that do more than just crash on line 2 with “FROM keyword not found where expected.” And that is genuine progress.
