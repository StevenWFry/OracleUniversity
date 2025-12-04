# Lesson 15 Lab – Managing Schema Objects (Or: Negotiating With Your Constraints)

And look, this is the chapter where you stop being scared of constraints, temporary tables, and external files, and start using them deliberately instead of just hoping they behave.

In this lab you will:

- Add and drop primary/foreign key and check constraints  
- See what “deferrable” really means at commit time  
- Create external tables over flat files and Data Pump exports  

All work is in the `ora21` / `sql2` environment.

---

## Task 0 – Run the cleanup script (ritual sacrifice to the lab gods)

From the cleanup directory:

```sql
@/home/oracle/labs/sql2/code_ex/cleanup_scripts/cleanup_15.sql
```

Ignore errors about missing tables; that just means your environment is already clean.

---

## Task 1 – Build and populate `DEPT2`

Create `DEPT2` using the structure from the book (simplified here):

```sql
CREATE TABLE dept2 (
  id          NUMBER(7),
  name        VARCHAR2(25),
  location_id NUMBER(4)
);
```

Load it from `DEPARTMENTS`:

```sql
INSERT INTO dept2 (id, name, location_id)
SELECT department_id,
       department_name,
       location_id
FROM   departments;

COMMIT;
```

Quick sanity check:

```sql
SELECT * FROM dept2;
-- Expect 27 rows
```

---

## Task 2 – Build `EMP2`

Create a simple employee table that points at `DEPT2`:

```sql
CREATE TABLE emp2 (
  id        NUMBER(7),
  last_name VARCHAR2(25),
  first_name VARCHAR2(25),
  dept_id   NUMBER(7)
);
```

Confirm:

```sql
DESC emp2;
```

---

## Task 3 – Add primary keys (the “you must be unique” rule)

Add a table‑level primary key on `EMP2.ID`:

```sql
ALTER TABLE emp2
  ADD CONSTRAINT my_emp_id_pk
      PRIMARY KEY (id);
```

Add a primary key on `DEPT2.ID`:

```sql
ALTER TABLE dept2
  ADD CONSTRAINT my_dept_id_pk
      PRIMARY KEY (id);
```

At this point both tables have proper identities and are ready to be judged harshly by foreign keys.

---

## Task 4 – Add a foreign key from `EMP2` to `DEPT2`

Prevent employees from being assigned to imaginary departments:

```sql
ALTER TABLE emp2
  ADD CONSTRAINT my_emp_dept_id_fk
      FOREIGN KEY (dept_id)
      REFERENCES dept2 (id);
```

Now if you try to insert an `EMP2` row whose `dept_id` isn’t in `DEPT2.ID`, Oracle will complain, correctly.

---

## Task 5 – Add a `COMMISSION` column with a check constraint

HR wants commission percentages, but not the exciting negative ones.

```sql
ALTER TABLE emp2
  ADD commission NUMBER(2,2)
      CONSTRAINT my_emp_comm_ck
      CHECK (commission > 0);
```

This makes any non‑positive commission value a crime against the schema.

---

## Task 6 – Drop `EMP2` and `DEPT2` with no chance of resurrection

You’ve proven you can create constraints; now prove you can get rid of them.

```sql
DROP TABLE emp2  PURGE;
DROP TABLE dept2 PURGE;
```

Because of `PURGE`, these tables skip the Recycle Bin and go straight to “gone forever”.

---

## Task 7 – External table #1: `LIBRARY_ITEMS_EXT` with `ORACLE_LOADER`

You’ll build an external table over a flat file that already lives in the filesystem.

1. Create (or verify) the directory object:

   ```sql
   CREATE OR REPLACE DIRECTORY emp_dir
     AS '/home/oracle/labs/sql2/empdir';
   ```

   As the creator, `ora21` already has read/write on it.

2. Inspect the file in that directory (`library_items.dat`); it contains four comma‑separated fields per line.

3. Create the external table using the Oracle Loader access driver:

   ```sql
   CREATE TABLE library_items_ext (
     category_id NUMBER,
     book_id     NUMBER,
     book_price  NUMBER,
     quantity    NUMBER
   )
   ORGANIZATION EXTERNAL (
     TYPE ORACLE_LOADER
     DEFAULT DIRECTORY emp_dir
     ACCESS PARAMETERS (
       RECORDS DELIMITED BY NEWLINE
       FIELDS TERMINATED BY ','
     )
     LOCATION ('library_items.dat')
   )
   REJECT LIMIT UNLIMITED;
   ```

4. Query it like a normal table, even though all the rows live in the file:

   ```sql
   SELECT * 
   FROM   library_items_ext;
   ```

If that works, you’ve successfully convinced Oracle to treat a text file like a table.

---

## Task 8 – External table #2: `DEPT_ADDR_EXT` with `ORACLE_DATAPUMP`

Now you’ll create an external table backed by Data Pump export files, using a query that joins `LOCATIONS` and `COUNTRIES`.

Open `lab_15_10.sql`, replace the `TODO` placeholders, and save as `lab_15_10_solution.sql` with something equivalent to:

```sql
CREATE TABLE dept_addr_ext (
  location_id   NUMBER(4),
  street_address VARCHAR2(40),
  city          VARCHAR2(30),
  state_province VARCHAR2(25),
  country_name  VARCHAR2(40)
)
ORGANIZATION EXTERNAL (
  TYPE ORACLE_DATAPUMP
  DEFAULT DIRECTORY emp_dir
  LOCATION ('ora21_emp4.exp', 'ora21_emp5.exp')
) PARALLEL
AS
SELECT l.location_id,
       l.street_address,
       l.city,
       l.state_province,
       c.country_name
FROM   locations  l
NATURAL JOIN countries c;
```

Run the script:

```sql
@/home/oracle/labs/sql2/labs/lab_15_10_solution.sql
```

Check that the export files now exist in `emp_dir`, then query the external table:

```sql
SELECT * 
FROM   dept_addr_ext;
```

The rows now come from the `.exp` files, not directly from the base tables.

---

## Task 9 – Deferrable primary key on `EMP_BOOKS`

Time to watch a deferrable constraint wait politely and then explode at `COMMIT`.

1. Run the first script to create `EMP_BOOKS` with a **non‑deferrable** PK:

   ```sql
   @/home/oracle/labs/sql2/labs/lab_15_11_a.sql
   ```

   The definition looks like:

   ```sql
   CREATE TABLE emp_books (
     book_id   NUMBER
               CONSTRAINT emp_books_pk PRIMARY KEY,
     title     VARCHAR2(50),
     category  VARCHAR2(30)
   );
   ```

2. Attempt to insert duplicate `book_id` values:

   ```sql
   @/home/oracle/labs/sql2/labs/lab_15_11_b.sql
   ```

   You should immediately get an `ORA-00001: unique constraint (EMP_BOOKS_PK) violated`.  
   Roll back:

   ```sql
   ROLLBACK;
   ```

3. Make the primary key deferrable:

   ```sql
   ALTER TABLE emp_books
     DROP CONSTRAINT emp_books_pk;

   ALTER TABLE emp_books
     ADD CONSTRAINT emp_books_pk
         PRIMARY KEY (book_id)
         DEFERRABLE;
   ```

4. Defer the constraint:

   ```sql
   SET CONSTRAINT emp_books_pk DEFERRED;
   ```

5. Run the insert script that includes duplicated `book_id` values:

   ```sql
   @/home/oracle/labs/sql2/labs/lab_15_11_g.sql
   ```

   This time, all `INSERT`s succeed. A quick check:

   ```sql
   SELECT * FROM emp_books;
   ```

   You should see three rows, including duplicates.

6. Now **commit** and watch the constraint finally wake up:

   ```sql
   COMMIT;
   ```

   Oracle checks the deferred constraint, finds the duplicate key, raises the error, and rolls back the entire transaction. A follow‑up query:

   ```sql
   SELECT * FROM emp_books;
   ```

   shows that none of the rows survived.

And that’s the whole point of deferrable constraints: they let you be temporarily wrong, as long as you’re right by the time you hit `COMMIT`—a standard we should frankly apply more often outside of databases too.

