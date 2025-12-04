## Lesson 11 Lab – Introduction to Data Definition Language in Oracle (or: assembling and disassembling tables without panic)

And look, up to now you’ve mostly treated tables as immutable facts of life—like gravity or office politics. This lab is where you actually **create**, **reshape**, and **destroy** tables on purpose. The trick is doing it in a way that future you can still understand.

You will:

- Create new tables with `CREATE TABLE`.
- Create a table from an existing one with `CREATE TABLE AS SELECT` (CTAS).
- Verify structures with `DESC`.
- Alter tables to add/modify/drop columns.
- Mark columns as UNUSED and drop them.
- Set a table to `READ ONLY` and back to `READ WRITE`.
- Drop tables when you’re done.

Before starting, run the cleanup script.

---

## Setup – Run `cleanup_11.sql`

1. In SQL Developer, open:
   - `home/oracle/labs/sql1/code_ex/cleanup_scripts/cleanup_11.sql`.
2. Run it as a **script** (F5) with your `myconnection` connection.
   - You may see errors for tables that don’t exist yet; that’s fine—the point is to ensure a clean slate.

---

## Task 1 – Create `DEPT` (simple table with primary key) – `lab_11_01.sql`

**Goal:** Create a `DEPT` table with `ID` and `NAME`, `ID` as primary key.

1. New worksheet.
2. Write:

   ```sql
   CREATE TABLE dept (
     id   NUMBER(7)
          CONSTRAINT dept_id_pk PRIMARY KEY,
     name VARCHAR2(25)
   );
   ```

3. Save as `lab_11_01.sql` in `home/oracle/labs/sql1/labs`.
4. Run the script.
5. Verify structure:

   ```sql
   DESC dept;
   ```

You should see `ID` (NOT NULL) and `NAME`.

---

## Task 2 – Create `EMP` with a Foreign Key to `DEPT` – `lab_11_02.sql`

**Goal:** Create an `EMP` table with a foreign key referencing `DEPT`.

Required columns:

- `ID` – `NUMBER(7)`
- `LAST_NAME` – `VARCHAR2(25)`
- `FIRST_NAME` – `VARCHAR2(25)`
- `DEPT_ID` – `NUMBER(7)`, foreign key to `DEPT(ID)`

1. New worksheet.
2. Write:

   ```sql
   CREATE TABLE emp (
     id        NUMBER(7),
     last_name VARCHAR2(25),
     first_name VARCHAR2(25),
     dept_id   NUMBER(7),

     CONSTRAINT emp_id_pk
       PRIMARY KEY (id),
     CONSTRAINT emp_dept_id_fk
       FOREIGN KEY (dept_id)
       REFERENCES dept (id)
   );
   ```

3. Save as `lab_11_02.sql`.
4. Run the script.
5. Verify:

   ```sql
   DESC emp;
   ```

Should show `ID`, `LAST_NAME`, `FIRST_NAME`, `DEPT_ID` with the primary and foreign keys in place.

---

## Task 3 – Add a `COMMISSION` Column to `EMP`

**Goal:** Use `ALTER TABLE ... ADD` to extend the structure.

1. New worksheet.
2. Add a `COMMISSION` column:

   ```sql
   ALTER TABLE emp
   ADD (commission NUMBER(2,2));
   ```

3. Verify:

   ```sql
   DESC emp;
   ```

You should see `COMMISSION` as the last column.

---

## Task 4 – Widen `LAST_NAME` to 50 Characters

**Goal:** Use `ALTER TABLE ... MODIFY`.

1. Modify the column definition:

   ```sql
   ALTER TABLE emp
   MODIFY (last_name VARCHAR2(50));
   ```

2. Confirm:

   ```sql
   DESC emp;
   ```

`LAST_NAME` should now be `VARCHAR2(50)`.

---

## Task 5 – Drop the `FIRST_NAME` Column

**Goal:** Remove a column you no longer want.

1. Drop the column:

   ```sql
   ALTER TABLE emp
   DROP COLUMN first_name;
   ```

2. Confirm:

   ```sql
   DESC emp;
   ```

`FIRST_NAME` should be gone.

---

## Task 6 – Mark `DEPT_ID` as UNUSED

**Goal:** Hide a column without fully dropping it yet.

1. Mark as unused:

   ```sql
   ALTER TABLE emp
   SET UNUSED (dept_id);
   ```

2. Confirm with `DESC emp;`:

   - `DEPT_ID` will no longer appear in the description.

The column still exists under the hood, but is now logically “dead” to the schema.

---

## Task 7 – Drop All UNUSED Columns from `EMP`

**Goal:** Permanently remove columns previously marked as UNUSED.

1. Run:

   ```sql
   ALTER TABLE emp
   DROP UNUSED COLUMNS;
   ```

2. `DESC emp;` will show only `ID`, `LAST_NAME`, `COMMISSION`.

This is a common pattern for cleaning up large tables in stages.

---

## Task 8 – Create `EMPLOYEES2` with CTAS (CREATE TABLE AS SELECT)

**Goal:** Clone selected columns from `EMPLOYEES` into a new table.

Requirements:

- Source table: `EMPLOYEES`.
- Columns to include: `EMPLOYEE_ID`, `FIRST_NAME`, `LAST_NAME`, `SALARY`, `DEPARTMENT_ID`.
- New names: `ID`, `FIRST_NAME`, `LAST_NAME`, `SALARY`, `DEPT_ID`.

1. New worksheet.
2. Create the table:

   ```sql
   CREATE TABLE employees2 AS
   SELECT employee_id AS id,
          first_name,
          last_name,
          salary,
          department_id AS dept_id
   FROM   employees;
   ```

3. Verify structure and data:

   ```sql
   DESC employees2;
   SELECT * FROM employees2 FETCH FIRST 5 ROWS ONLY;
   ```

You should see the renamed columns with copied data.

---

## Task 9 – Set `EMPLOYEES2` to READ ONLY

**Goal:** Prevent DML/DDL changes on the table.

1. Make the table read‑only:

   ```sql
   ALTER TABLE employees2 READ ONLY;
   ```

2. Try a DML or `TRUNCATE`:

   ```sql
   TRUNCATE TABLE employees2;
   ```

   - Oracle should reject the operation with an error (updates not allowed on a read‑only table).

---

## Task 10 – Switch Back to READ WRITE and Truncate

**Goal:** Re‑enable changes and empty the table.

1. Set table back to read/write:

   ```sql
   ALTER TABLE employees2 READ WRITE;
   ```

2. Now truncate:

   ```sql
   TRUNCATE TABLE employees2;
   ```

3. Verify it’s empty:

   ```sql
   SELECT * FROM employees2;
   ```

You should see no rows.

---

## Task 11 – Drop the Practice Tables

**Goal:** Clean up `EMP`, `DEPT`, and `EMPLOYEES2`.

1. Run:

   ```sql
   DROP TABLE emp;
   DROP TABLE dept;
   DROP TABLE employees2;
   ```

2. If your Recycle Bin is enabled, the tables will linger there until purged; otherwise, they’re gone.

---

## What This Lab Leaves You With

After completing this lab, you should be able to:

- Create new tables with `CREATE TABLE` and CTAS.
- Inspect table structures with `DESC`.
- Use `ALTER TABLE` to add, modify, and drop columns, including UNUSED handling.
- Use `ALTER TABLE ... READ ONLY/READ WRITE` to protect tables during maintenance.
- Clean up by dropping tables when they’re no longer needed.

You now have hands‑on experience with the part of SQL that builds and reshapes the schema itself—essentially, the power tools of the database world. Use them carefully, or at least near a backup.
