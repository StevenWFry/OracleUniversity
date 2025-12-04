# Lesson 19/20 Lab – Manipulating Data Using Advanced Queries (Multi‑Table Inserts, MERGE, and Time Travel)

And look, up to now your DML has probably been “INSERT one row here, UPDATE one row there.” That’s adorable. This lab is where you start inserting into **multiple tables at once**, merging data sets, and casually rolling tables back in time.

You’ll practice:

- Multi‑table `INSERT ALL` (including pivoting inserts)  
- `MERGE` with external data  
- Flashback of dropped tables via the Recycle Bin  

Environment: `ora21`, `sql2`.

---

## Task 0 – Run `cleanup_19.sql` and start fresh

From the cleanup scripts directory:

```sql
@/home/oracle/labs/sql2/code_ex/cleanup_scripts/cleanup_19.sql
```

It drops any leftover lab tables. Errors about missing tables just mean you’re clean already.

---

## Task 1 – Multi‑table insert: `SAL_HISTORY`, `MGR_HISTORY`, `SPECIAL_SAL`

### 1.1 Create the target tables

Run the starter script:

```sql
@/home/oracle/labs/sql2/labs/lab_19_01.sql
```

This creates:

- `SAL_HISTORY` – `(employee_id, hire_date, salary)`  
- `MGR_HISTORY` – `(employee_id, manager_id, salary)`  
- `SPECIAL_SAL` – `(employee_id, salary)`

Verify structures:

```sql
DESC sal_history;
DESC mgr_history;
DESC special_sal;
```

### 1.2 Source query: employees with `employee_id < 125`

Base data set:

```sql
SELECT employee_id   AS empid,
       hire_date     AS hiredate,
       salary        AS sal,
       manager_id    AS mgr
FROM   employees
WHERE  employee_id < 125;
-- 25 rows
```

### 1.3 Conditional multi‑table insert

Now route each row into one or more target tables based on `sal`:

```sql
INSERT ALL
  WHEN sal > 20000 THEN
    INTO special_sal (employee_id, salary)
    VALUES (empid, sal)
  ELSE
    INTO sal_history (employee_id, hire_date, salary)
    VALUES (empid, hiredate, sal)
    INTO mgr_history (employee_id, manager_id, salary)
    VALUES (empid, mgr, sal)
SELECT employee_id   AS empid,
       hire_date     AS hiredate,
       salary        AS sal,
       manager_id    AS mgr
FROM   employees
WHERE  employee_id < 125;
```

Check the results:

```sql
SELECT * FROM special_sal;
SELECT * FROM sal_history;
SELECT * FROM mgr_history;
```

You should see:

- 1 high‑salary row in `SPECIAL_SAL` (for the > 20,000 case)  
- 24 rows in `SAL_HISTORY`  
- 24 rows in `MGR_HISTORY`  

One source query, 49 rows inserted across three tables. Efficient *and* slightly unsettling.

---

## Task 2 – Pivoting insert: `SALES_WEEK_DATA` → `EMP_SALES_INFO`

### 2.1 Build and populate `SALES_WEEK_DATA`

Run the scripts:

```sql
@/home/oracle/labs/sql2/labs/lab_19_06_a.sql  -- creates SALES_WEEK_DATA
@/home/oracle/labs/sql2/labs/lab_19_06_b.sql  -- inserts one row
```

Check:

```sql
DESC sales_week_data;

SELECT *
FROM   sales_week_data;
```

You should see columns like:

- `id`, `week_id`, `mon_qty`, `tue_qty`, `wed_qty`, `thu_qty`, `fri_qty` (names may vary slightly)

### 2.2 Create `EMP_SALES_INFO`

Run:

```sql
@/home/oracle/labs/sql2/labs/lab_19_06_e.sql
```

Then:

```sql
DESC emp_sales_info;
```

Structure:

- `id`, `week`, `qty_sales`

### 2.3 Pivot the single row into five rows

Take the one weekly row and split it into five day‑rows:

```sql
INSERT ALL
  INTO emp_sales_info (id, week, qty_sales)
  VALUES (id, week_id, mon_qty)
  INTO emp_sales_info (id, week, qty_sales)
  VALUES (id, week_id, tue_qty)
  INTO emp_sales_info (id, week, qty_sales)
  VALUES (id, week_id, wed_qty)
  INTO emp_sales_info (id, week, qty_sales)
  VALUES (id, week_id, thu_qty)
  INTO emp_sales_info (id, week, qty_sales)
  VALUES (id, week_id, fri_qty)
SELECT id,
       week_id,
       mon_qty,
       tue_qty,
       wed_qty,
       thu_qty,
       fri_qty
FROM   sales_week_data;
```

Verify:

```sql
SELECT *
FROM   emp_sales_info
ORDER  BY id, week;
```

One row in, five rows out. You’ve just done a pivot without using the `PIVOT` keyword, which is both clever and mildly cursed.

---

## Task 3 – Merge external employee data into `EMP_HIST`

### 3.1 Create external table `EMP_DATA`

Run:

```sql
@/home/oracle/labs/sql2/labs/lab_19_07.sql
```

This builds an external table `EMP_DATA` over the `emp.dat` file in the `EMP_DIR` directory.

Check:

```sql
DESC emp_data;
SELECT * FROM emp_data;
```

You’ll see past employee first/last names and email addresses.

### 3.2 Create `EMP_HIST` and adjust `EMAIL` length

Run:

```sql
@/home/oracle/labs/sql2/labs/lab_19_08.sql
```

Then widen `EMAIL`:

```sql
ALTER TABLE emp_hist
  MODIFY email VARCHAR2(45);
```

### 3.3 `MERGE` external data into `EMP_HIST`

Rules:

- If `EMP_DATA` row matches `EMP_HIST` (same `first_name` and `last_name`), update `email` in `EMP_HIST`.  
- If it doesn’t match, insert the row into `EMP_HIST`.

```sql
MERGE INTO emp_hist f
USING emp_data h
   ON (f.first_name = h.first_name
       AND f.last_name = h.last_name)
WHEN MATCHED THEN
  UPDATE SET f.email = h.email
WHEN NOT MATCHED THEN
  INSERT (first_name, last_name, email)
  VALUES (h.first_name, h.last_name, h.email);
```

Check how many rows you now have:

```sql
SELECT COUNT(*) FROM emp_hist;
SELECT * FROM emp_hist FETCH FIRST 10 ROWS ONLY;
```

You’ve just combined current and external historical data without writing a single loop. Somewhere, a PL/SQL package just felt obsolete.

---

## Task 4 – Flashback a dropped table (Recycle Bin sorcery)

### 4.1 Create and describe `EMP2`

Create a simple table:

```sql
CREATE TABLE emp2 (
  id         NUMBER,
  last_name  VARCHAR2(25),
  first_name VARCHAR2(25),
  dept_id    NUMBER
);

DESC emp2;
```

### 4.2 Drop the table and find it in the Recycle Bin

```sql
DROP TABLE emp2;
```

Look into the Recycle Bin:

```sql
SELECT original_name,
       operation,
       droptime
FROM   recyclebin
WHERE  original_name = 'EMP2';
```

You should see `EMP2` listed as a dropped object.

### 4.3 Flashback the table to before the drop

```sql
FLASHBACK TABLE emp2 TO BEFORE DROP;
```

Confirm the resurrection:

```sql
DESC emp2;
SELECT * FROM emp2;
```

The structure is back, and any rows that existed at drop time are back too. Use this power wisely, preferably before someone starts rebuilding the table manually.

---

## Task 5 – Create `EMP3` from script

Finally, create another table for later adventures:

```sql
@/home/oracle/labs/sql2/labs/lab_19_13.sql
```

Check:

```sql
DESC emp3;
```

No complex operations here—it’s just setting the stage for subsequent exercises.

---

By the end of this lab you’ve:

- Used `INSERT ALL` for conditional and pivoting multi‑table inserts  
- Merged external and internal data with `MERGE`  
- Dropped and flashbacked a table via the Recycle Bin  

Which means your DML is no longer “row by row”; it’s now “entire data sets at once.” Congratulations: you’re officially dangerous. Just don’t run any of this in production without a good backup and a very understanding DBA.  

