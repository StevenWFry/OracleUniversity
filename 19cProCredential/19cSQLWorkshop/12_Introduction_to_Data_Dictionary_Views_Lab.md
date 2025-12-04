## Lesson 12 Lab – Introduction to Data Dictionary Views (or: letting the database snitch on itself)

And look, when something goes wrong in SQL, half the time the answer is “there’s a constraint you didn’t know about” or “you’re querying the wrong table entirely.” The good news is: Oracle *knows* all of this. The bad news is: it only tells you if you ask the right data dictionary views.

You will:

- Use `USER_TABLES` and `ALL_TABLES` to see what tables exist.
- Explore column definitions via `USER_TAB_COLUMNS`.
- Inspect constraints via `USER_CONSTRAINTS` and `USER_CONS_COLUMNS`.
- Add and read comments on tables.
- Verify new tables and constraints created by scripts.

This lab assumes you’ve already switched to the `ORA21` schema (or equivalent) and run the `cleanup_12` prerequisites where needed.

---

## Task 1 – See Which Tables You Own (`USER_TABLES`)

**Goal:** List all tables in your own schema.

1. Open a new worksheet connected as `ORA21`.
2. Query `USER_TABLES`:

   ```sql
   SELECT table_name
   FROM   user_tables;
   ```

3. Run it and review the list: these are **your** tables.

---

## Task 2 – See All Accessible Tables, Excluding Your Own (`ALL_TABLES`)

**Goal:** See tables you can access that belong to **other** schemas.

1. In the same worksheet, query `ALL_TABLES` with a filter:

   ```sql
   SELECT owner,
          table_name
   FROM   all_tables
   WHERE  owner <> 'ORA21'
   ORDER  BY owner, table_name;
   ```

2. Run it. You should see a larger set of tables owned by schemas like `SYS`, `SYSTEM`, and others.

This view answers “what tables can I *see*, not just what tables do I own?”

---

## Task 3 – Script to Report Column Definitions – `lab_12_03.sql`

**Goal:** Build a script that, for any table, reports column names, data types, lengths, precision/scale, and nullability.

Requirements:

- Prompt for the table name.
- Use `USER_TAB_COLUMNS`.
- Uppercase the table name input.

1. New worksheet.
2. Write:

   ```sql
   SELECT column_name,
          data_type,
          data_length,
          data_precision AS "PRECISION",
          data_scale     AS "SCALE",
          nullable
   FROM   user_tab_columns
   WHERE  table_name = UPPER('&table_name')
   ORDER  BY column_id;
   ```

3. Save as `lab_12_03.sql` in `home/oracle/labs/sql2/labs` (or your SQL2 labs folder).
4. Run the script and, when prompted, enter `departments`.
5. Confirm that you see columns like `DEPARTMENT_ID`, `DEPARTMENT_NAME`, `MANAGER_ID`, `LOCATION_ID` with their types and nullability.

---

## Task 4 – Script to Report Constraint Details – `lab_12_04.sql`

**Goal:** For any table, show constraints and their columns from `USER_CONSTRAINTS` and `USER_CONS_COLUMNS`.

Requirements:

- Show: `COLUMN_NAME`, `CONSTRAINT_NAME`, `CONSTRAINT_TYPE`, `SEARCH_CONDITION`, `STATUS`.
- Join `USER_CONSTRAINTS` (alias `uc`) to `USER_CONS_COLUMNS` (alias `ucc`).
- Prompt for the table name.

1. New worksheet.
2. Write:

   ```sql
   SELECT ucc.column_name,
          uc.constraint_name,
          uc.constraint_type,
          uc.search_condition,
          uc.status
   FROM   user_constraints   uc
   JOIN   user_cons_columns  ucc
          ON uc.constraint_name = ucc.constraint_name
         AND uc.table_name     = ucc.table_name
   WHERE  uc.table_name = UPPER('&table_name')
   ORDER  BY uc.constraint_name,
             ucc.position;
   ```

3. Save as `lab_12_04.sql`.
4. Run it and enter `departments` (in any case; `UPPER` will normalize it).
5. Review the list of constraints and the columns they apply to.

---

## Task 5 – Add and Verify a Table Comment

**Goal:** Document the purpose of a table and verify the comment via `USER_TAB_COMMENTS`.

1. Add a comment to `DEPARTMENTS`:

   ```sql
   COMMENT ON TABLE departments IS
     'Company department information including name, code, and location.';
   ```

2. Verify via `USER_TAB_COMMENTS`:

   ```sql
   SELECT table_name,
          comments
   FROM   user_tab_comments
   WHERE  table_name = 'DEPARTMENTS';
   ```

You should see your new description attached to the `DEPARTMENTS` table.

(You can similarly use `USER_COL_COMMENTS` if you want to document individual columns.)

---

## Task 6 – Run `lab_12_06_tab.sql` to Create `DEPT2` and `EMP2`

**Goal:** Prepare for the remaining exercises by creating two new tables.

1. Open `home/oracle/labs/sql2/labs/lab_12_06_tab.sql`.
2. Run it as a script.
   - It drops `DEPT2` and `EMP2` if needed, then recreates them.

---

## Task 7 – Verify `DEPT2` and `EMP2` in `USER_TABLES`

**Goal:** Confirm that both tables exist in the data dictionary.

1. In a new worksheet, run:

   ```sql
   SELECT table_name
   FROM   user_tables
   WHERE  table_name IN ('DEPT2', 'EMP2');
   ```

2. You should see `DEPT2` and `EMP2` listed.

---

## Task 8 – Inspect Constraints on `DEPT2` and `EMP2`

**Goal:** Use `USER_CONSTRAINTS` to see the constraint names and types.

1. Query:

   ```sql
   SELECT constraint_name,
          constraint_type,
          table_name
   FROM   user_constraints
   WHERE  table_name IN ('DEPT2', 'EMP2')
   ORDER  BY table_name, constraint_name;
   ```

2. Review which tables have primary keys, foreign keys, or other constraints.

If you want the columns too, join to `USER_CONS_COLUMNS` as in Task 4.

---

## Task 9 – Confirm Object Types via `USER_OBJECTS`

**Goal:** Verify that `DEPT2` and `EMP2` exist as tables in your schema.

1. Query:

   ```sql
   SELECT object_name,
          object_type
   FROM   user_objects
   WHERE  object_name IN ('DEPT2', 'EMP2');
   ```

2. You should see both objects listed with `OBJECT_TYPE = 'TABLE'`.

---

## What This Lab Leaves You With

After completing this lab, you should be able to:

- Use `USER_TABLES` and `ALL_TABLES` to see which tables you own and which you can access.
- Use `USER_TAB_COLUMNS` to inspect column definitions.
- Use `USER_CONSTRAINTS` and `USER_CONS_COLUMNS` to understand how tables are protected by constraints.
- Add descriptive comments and retrieve them via `USER_TAB_COMMENTS`.
- Confirm that newly created tables and constraints have actually made it into the data dictionary.

In other words, you can now ask the database “what exactly are you doing?” and get a straight answer—at least about your schema. For everything else, there’s still email.
