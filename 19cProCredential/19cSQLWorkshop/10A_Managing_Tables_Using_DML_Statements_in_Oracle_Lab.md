## Lesson 10 Lab – Manipulating Data (or: how to break things and then un‑break them)

And look, this is the first lab where you’re not just *looking* at data—you’re actually **changing** it. This is where good habits (and `COMMIT`/`ROLLBACK`) matter, because `DELETE` does not ask “are you sure?”

You will:

- Insert rows into `MY_EMPLOYEE` in several different ways.
- Use substitution variables to build a reusable insert script.
- Update and delete rows and verify changes.
- Use `COMMIT`, `ROLLBACK`, and `SAVEPOINT` to control transactions.

Before starting, run `cleanup_10.sql` so the environment is clean.

---

## Setup – Run the Cleanup Script

1. In SQL Developer, navigate to:
   - `home/oracle/labs/sql1/code_ex/cleanup_scripts/cleanup_10.sql`.
2. Open it and choose your `myconnection` (or equivalent HR) connection.
3. Run it as a **script** (F5).  
   This resets the objects used in this lab.

---

## Task 1 – Initialize `MY_EMPLOYEE` with a Script

1. Open `home/oracle/labs/sql1/labs/lab_10_01.sql`.
2. Run it as a **script** using your HR connection.
3. This creates (or recreates) the `MY_EMPLOYEE` table.

Confirm its structure:

```sql
DESC my_employee;
```

You should see: `ID`, `LAST_NAME`, `FIRST_NAME`, `USERID`, `SALARY`.

---

## Task 2 – Insert First Row (No Column List)

**Goal:** Insert the first sample row **without** listing columns.

1. In a new worksheet, run:

   ```sql
   INSERT INTO my_employee
   VALUES (1, 'Patel', 'Ralph', 'rpatel', 895);
   ```

2. Run as a statement. You should see `1 row inserted.`

Because no column list is provided, values must match the table’s column order.

---

## Task 3 – Insert Second Row (With Column List)

**Goal:** Insert a row while explicitly naming columns.

1. Add another `INSERT` using column names:

   ```sql
   INSERT INTO my_employee (id, last_name, first_name, userid, salary)
   VALUES (2, 'Dancs', 'Betty', 'bdancs', 860);
   ```

2. Run it and confirm `1 row inserted.`

---

## Task 4 – Verify Current Data

1. New worksheet (or reuse one).
2. Query the table:

   ```sql
   SELECT *
   FROM   my_employee;
   ```

You should see two rows: Patel and Dancs.

---

## Task 5 – Dynamic Insert Script with Substitution Variables – `lab_10_06.sql`

**Goal:** Write a reusable script that prompts for all column values.

Requirements:

- Prompt for: `ID`, `LAST_NAME`, `FIRST_NAME`, `USERID`, `SALARY`.
- Do not list columns in the `INSERT` (use table’s default order).

1. In a new worksheet, write:

   ```sql
   INSERT INTO my_employee
   VALUES (&id, '&last_name', '&first_name', '&userid', &salary);
   ```

2. Save this as `lab_10_06.sql` in `home/oracle/labs/sql1/labs`.

3. Run `lab_10_06.sql` **twice** to insert the next two sample rows. For example:

   - First run: `3, 'Biri', 'Ben', 'bbiri', 1100`.
   - Second run: `4, 'Newman', 'Chad', 'cnewman', 750`.

4. Re‑query:

   ```sql
   SELECT *
   FROM   my_employee;
   ```

You should now see four rows.

---

## Task 6 – Commit the Inserts

Make the current data permanent:

```sql
COMMIT;
```

Run as a statement. All four rows are now committed and visible in other sessions.

---

## Task 7 – Update and Delete Data

### 7a – Change last name of employee 3 to Drexler

1. Update:

   ```sql
   UPDATE my_employee
   SET    last_name = 'Drexler'
   WHERE  id = 3;
   ```

### 7b – Raise low salaries to 1000

2. Update all employees with salary < 900:

   ```sql
   UPDATE my_employee
   SET    salary = 1000
   WHERE  salary < 900;
   ```

3. Verify:

   ```sql
   SELECT *
   FROM   my_employee;
   ```

You should see three employees at 1000 and one at 1100, and `id = 3` now has last name `Drexler`.

### 7c – Delete Betty Dancs

4. Delete:

   ```sql
   DELETE FROM my_employee
   WHERE  last_name = 'Dancs';
   ```

5. Confirm:

   ```sql
   SELECT *
   FROM   my_employee;
   ```

Betty should be gone.

### 7d – Commit the changes

```sql
COMMIT;
```

All current rows and updates are now permanent.

---

## Task 8 – Insert the Final Sample Row Using the Script

**Goal:** Use `lab_10_06.sql` to insert one more employee.

The remaining sample row is something like:

- `id = 5, last_name = 'Ropeburn', first_name = 'Audrey', userid = 'aropeburn', salary = 1550`.

1. Run `lab_10_06.sql` again and supply those values.
2. Verify:

   ```sql
   SELECT *
   FROM   my_employee;
   ```

You should now see four committed rows plus the new one (five total).

---

## Task 9 – Use SAVEPOINT and ROLLBACK

**Goal:** Practice rolling back part of a transaction.

1. Comment out any `COMMIT` in your worksheet so you don’t commit by accident.
2. Create a savepoint after adding the new row:

   ```sql
   SAVEPOINT step17;
   ```

3. Delete all rows:

   ```sql
   DELETE FROM my_employee;
   ```

4. Confirm the table is empty:

   ```sql
   SELECT *
   FROM   my_employee;
   ```

5. Roll back only the deletion:

   ```sql
   ROLLBACK TO step17;
   ```

6. Query again:

   ```sql
   SELECT *
   FROM   my_employee;
   ```

All four rows (including the last inserted one) should be back.

7. Now make everything permanent:

   ```sql
   COMMIT;
   ```

---

## Task 10 – Auto‑Generating USERID – `lab_10_24.sql`

**Goal:** Modify your insert script so `USERID` is generated automatically as:

- First letter of `FIRST_NAME` + first seven characters of `LAST_NAME`.
- All in lowercase.

The script should:

- Prompt for `ID`, `LAST_NAME`, `FIRST_NAME`, and `SALARY`.
- **Not** prompt for `USERID`.

1. Copy `lab_10_06.sql` into a new worksheet.
2. Change the `INSERT` to:

   ```sql
   INSERT INTO my_employee (id, last_name, first_name, userid, salary)
   VALUES (&id,
           '&&last_name',
           '&&first_name',
           LOWER(SUBSTR('&&first_name', 1, 1) || SUBSTR('&&last_name', 1, 7)),
           &salary);

   UNDEFINE first_name;
   UNDEFINE last_name;
   ```

3. Save this as `lab_10_24.sql`.
4. Run it and supply values for a new employee (e.g., `id = 6, last_name = 'Anthony', first_name = 'Mark', salary = 1230`).
5. Verify the generated `USERID`:

   ```sql
   SELECT *
   FROM   my_employee
   WHERE  id = 6;
   ```

You should see a `USERID` like `manthony` (first letter of first name + first 7 chars of last name) in lowercase.

---

## What This Lab Leaves You With

After completing these tasks, you should be able to:

- Insert rows with and without column lists, and with substitution variables.
- Update and delete rows, and verify effects with `SELECT` before committing.
- Use `COMMIT`, `ROLLBACK`, and `SAVEPOINT` to control when changes stick.
- Build dynamic scripts that derive values (like `USERID`) from other inputs.

You are now officially dangerous: you can not only change data but also undo those changes when you realize “maybe deleting everything wasn’t the vibe.”
