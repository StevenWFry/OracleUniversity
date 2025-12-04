## Lesson 13 Lab – Creating Sequences, Synonyms, and Indexes (or: accessorizing your schema for speed and sanity)

And look, your tables are fine, but they get a lot better once you add **automatic IDs**, **shorter names**, and **indexes** so queries don’t crawl. This lab is where you wire all of that up in Oracle.

You will:

- Create and use sequences for primary keys.
- Query `USER_SEQUENCES` for sequence metadata.
- Create and drop synonyms.
- Create indexes and verify them in the data dictionary.

Before you start, run `cleanup_13.sql` from `home/oracle/labs/sql2/code_ex/cleanup_scripts` to remove any leftover objects. It’s normal if it reports errors dropping things that don’t exist yet.

---

## Task 1 – Create `DEPT` Table for Testing

**Goal:** Create a simple `DEPT` table with an ID primary key.

1. In a new worksheet, create the table:

   ```sql
   CREATE TABLE dept (
     id   NUMBER(7)
          CONSTRAINT department_id_pk PRIMARY KEY,
     name VARCHAR2(25)
   );
   ```

2. Run it and confirm:

   ```sql
   DESC dept;
   ```

You should see columns `ID` (NOT NULL) and `NAME`.

---

## Task 2 – Create a Sequence for `DEPT.ID`

**Goal:** Build `DEPT_ID_SEQ` to generate IDs for the `DEPT` table.

Requirements:

- Start at **200**.
- Increment by **10**.
- Maximum value **1000**.

1. Create the sequence:

   ```sql
   CREATE SEQUENCE dept_id_seq
     START WITH 200
     INCREMENT BY 10
     MAXVALUE 1000;
   ```

2. Verify in the Sequences node in SQL Developer, or via:

   ```sql
   SELECT sequence_name,
          min_value,
          max_value,
          increment_by
   FROM   user_sequences
   WHERE  sequence_name = 'DEPT_ID_SEQ';
   ```

---

## Task 3 – Use the Sequence in Inserts – `lab_13_03.sql`

**Goal:** Insert two rows into `DEPT` using `DEPT_ID_SEQ.NEXTVAL`.

Rows to add:

- `Education`
- `Administration`

1. In a new worksheet, write:

   ```sql
   INSERT INTO dept
   VALUES (dept_id_seq.NEXTVAL, 'Education');

   INSERT INTO dept
   VALUES (dept_id_seq.NEXTVAL, 'Administration');
   ```

2. Save as `lab_13_03.sql` under `home/oracle/labs/sql2/labs`.
3. Run the script.
4. Confirm the inserts:

   ```sql
   SELECT *
   FROM   dept;
   ```

You should see IDs `200` and `210` assigned by the sequence.

---

## Task 4 – Script to Inspect Sequences – `lab_13_04.sql`

**Goal:** Show metadata about your sequences using `USER_SEQUENCES`.

Columns to display:

- `SEQUENCE_NAME`
- `MAX_VALUE`
- `INCREMENT_BY`
- `LAST_NUMBER`

1. New worksheet.
2. Write:

   ```sql
   SELECT sequence_name,
          max_value,
          increment_by,
          last_number
   FROM   user_sequences;
   ```

3. Save as `lab_13_04.sql`.
4. Run it and confirm `DEPT_ID_SEQ` appears with the configured values.

---

## Task 5 – Create and Inspect a Synonym

**Goal:** Create a synonym for the `EMPLOYEES` table and list synonyms in your schema.

1. Create a private synonym:

   ```sql
   CREATE SYNONYM emp1
   FOR   employees;
   ```

2. Verify via the dictionary:

   ```sql
   SELECT synonym_name,
          table_owner,
          table_name
   FROM   user_synonyms;
   ```

You should see `EMP1` pointing at `EMPLOYEES`.

3. Optionally test it:

   ```sql
   SELECT COUNT(*)
   FROM   emp1;
   ```

---

## Task 6 – Drop the Synonym

**Goal:** Clean up the synonym.

```sql
DROP SYNONYM emp1;
```

Re‑run the `USER_SYNONYMS` query to confirm it’s gone.

---

## Task 7 – Create a Non‑Unique Index on `DEPT.NAME`

**Goal:** Add an index to the `NAME` column of `DEPT` to speed lookups.

1. Create the index:

   ```sql
   CREATE INDEX dept_name_idx
   ON dept (name);
   ```

2. Inspect it:

   ```sql
   SELECT index_name,
          table_name,
          uniqueness
   FROM   user_indexes
   WHERE  table_name = 'DEPT';
   ```

`DEPT_NAME_IDX` should show up as a non‑unique index.

---

## Task 8 – Create `SALES_DEPT` with a Named PK Index

**Goal:** Create a `SALES_DEPT` table with a primary key and a named index `SALES_PK_IDX`.

Table structure:

- `TEAM_ID` – `NUMBER(3)` primary key, with index `SALES_PK_IDX`.
- `LOCATION` – `VARCHAR2(30)`.

1. Create the table and explicitly name the PK index:

   ```sql
   CREATE TABLE sales_dept (
     team_id  NUMBER(3),
     location VARCHAR2(30),

     CONSTRAINT sales_dept_pk
       PRIMARY KEY (team_id)
       USING INDEX (
         CREATE INDEX sales_pk_idx
         ON sales_dept (team_id)
       )
   );
   ```

2. Query index metadata:

   ```sql
   SELECT index_name,
          table_name,
          uniqueness
   FROM   user_indexes
   WHERE  table_name = 'SALES_DEPT';
   ```

`SALES_PK_IDX` should appear as a **UNIQUE** index on `SALES_DEPT`.

---

## Task 9 – Drop the Practice Objects

**Goal:** Remove the tables and sequence created in this practice.

1. Drop the tables and sequence:

   ```sql
   DROP TABLE sales_dept;
   DROP TABLE dept;
   DROP SEQUENCE dept_id_seq;
   ```

2. Optionally confirm they’re gone:

   ```sql
   SELECT table_name
   FROM   user_tables
   WHERE  table_name IN ('DEPT', 'SALES_DEPT');

   SELECT sequence_name
   FROM   user_sequences
   WHERE  sequence_name = 'DEPT_ID_SEQ';
   ```

All should return no rows.

---

## What This Lab Leaves You With

After completing this lab, you should be able to:

- Create sequences and use `NEXTVAL` to populate primary keys.
- Inspect sequence settings via `USER_SEQUENCES`.
- Create and drop synonyms, and find them via `USER_SYNONYMS`.
- Create indexes (including ones tied to primary keys) and verify them in `USER_INDEXES`.

You now know how to give your tables automatic IDs, shorter names, and faster lookups—all of which make your database nicer to use and slightly harder to blame when something goes wrong.
