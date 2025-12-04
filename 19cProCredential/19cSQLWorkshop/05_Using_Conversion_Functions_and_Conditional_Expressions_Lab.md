## Lesson 5 Lab – Using Conversion Functions and Conditional Expressions (or: teaching your data to speak fluent human)

And look, your database is perfectly happy returning raw salaries, cryptic job codes, and dates that look like they fell out of a log file. HR, however, would like sentences. And reviews. And grades. This lab is where you translate.

You will:

- Use `TO_CHAR` and `TO_DATE` for formatted output.
- Build sentence‑like strings with concatenation.
- Use `NEXT_DAY` and `ADD_MONTHS` for review dates.
- Use `NVL` to plug `NULL` holes.
- Use `CASE`, searched `CASE`, and `DECODE` to grade employees by job.

Use the **`ora1`** connection and the **SQL1** labs folder as before.

---

## Task 1 – Dream Salaries (three times the current salary)

**Goal:** Build a full English sentence per employee using `TO_CHAR` and concatenation.

Report for each employee:

- `last_name`
- `salary`
- A sentence of the form:

  > `King earns $24000 monthly but wants $72000.`

1. Open a new worksheet.
2. Write a query like:

   ```sql
   SELECT last_name,
          salary,
          last_name || ' earns ' ||
          TO_CHAR(salary, 'FML$99,999') || ' monthly but wants ' ||
          TO_CHAR(salary * 3, 'FML$99,999') || '.' AS "Dream Salaries"
   FROM   employees;
   ```

   Notes:
   - `FML` applies fill mode and local currency handling; you can also use `$99,999` if you prefer.
   - Adjust the format model width if your salaries are larger.

3. Run it and skim a few rows to make sure the sentences look sane.

Optionally, save as `lab_05_01.sql`.

---

## Task 2 – Six‑Month Review Date (first Monday after six months)

**Goal:** Use `ADD_MONTHS`, `NEXT_DAY`, and `TO_CHAR` to compute and format review dates.

For each employee, show:

- `last_name`
- `hire_date`
- `REVIEW` – the **first Monday after six months of service**, formatted like:

  > `Monday, the 31st of July 2000`

1. New worksheet.
2. Query:

   ```sql
   SELECT last_name,
          TO_CHAR(hire_date,
                  'FMDay, the DDspth "of" Month YYYY') AS hire_date_fmt,
          TO_CHAR(NEXT_DAY(ADD_MONTHS(hire_date, 6), 'MONDAY'),
                  'FMDay, the DDspth "of" Month YYYY') AS REVIEW
   FROM   employees;
   ```

3. Run it and confirm:
   - The `REVIEW` dates are always on a **Monday**.
   - They fall just after the date six months from `hire_date`.

---

## Task 3 – Commission or “No Commission”

**Goal:** Use `NVL` and `TO_CHAR` to mix numbers and text safely.

For each employee, display:

- `last_name`
- Commission amount, or the text `No Commission` if they don’t earn one.
- Label the column `COMM`.

1. New worksheet.
2. Since `NVL` requires compatible data types, convert the commission to text first:

   ```sql
   SELECT last_name,
          NVL(TO_CHAR(commission_pct, '0.99'), 'No Commission') AS COMM
   FROM   employees;
   ```

3. Check a few rows to confirm:
   - Commissioned employees show a numeric value.
   - Others show `No Commission`.

---

## Task 4 – Grading Employees by Job (Simple CASE)

**Goal:** Use a `CASE` expression to assign a grade based on `JOB_ID`.

Grading rules:

- `AD_PRES` → `A`
- `ST_MAN`  → `B`
- `IT_PROG` → `C`
- `SA_REP`  → `D`
- `ST_CLERK`→ `E`
- Anything else → `O` (other)

1. New worksheet.
2. Query:

   ```sql
   SELECT job_id,
          CASE job_id
            WHEN 'AD_PRES'  THEN 'A'
            WHEN 'ST_MAN'   THEN 'B'
            WHEN 'IT_PROG'  THEN 'C'
            WHEN 'SA_REP'   THEN 'D'
            WHEN 'ST_CLERK' THEN 'E'
            ELSE                 'O'
          END AS grade
   FROM   employees;
   ```

3. Run it and verify the grade letters line up with the job codes.

---

## Task 5 – Grading with a Searched CASE

**Goal:** Rewrite the previous logic as a **searched CASE** (conditions in each `WHEN`).

1. Starting from Task 4’s query, remove the `job_id` selector after `CASE`.
2. Move `job_id = ...` into each `WHEN` clause:

   ```sql
   SELECT job_id,
          CASE
            WHEN job_id = 'AD_PRES'  THEN 'A'
            WHEN job_id = 'ST_MAN'   THEN 'B'
            WHEN job_id = 'IT_PROG'  THEN 'C'
            WHEN job_id = 'SA_REP'   THEN 'D'
            WHEN job_id = 'ST_CLERK' THEN 'E'
            ELSE                          'O'
          END AS grade
   FROM   employees;
   ```

3. Run the query and confirm the results match Task 4.

This version is more flexible: you can add extra conditions (e.g., employee_id filters) inside each `WHEN` later.

---

## Task 6 – Grading with DECODE

**Goal:** Implement the same grading logic using Oracle’s `DECODE` function.

1. New worksheet.
2. Use `DECODE(job_id, ...)` to map job codes to grades:

   ```sql
   SELECT job_id,
          DECODE(job_id,
                 'AD_PRES',  'A',
                 'ST_MAN',   'B',
                 'IT_PROG',  'C',
                 'SA_REP',   'D',
                 'ST_CLERK', 'E',
                             'O') AS grade
   FROM   employees;
   ```

3. Run it and confirm the output matches the CASE‑based versions.

`DECODE` only supports equality checks and is Oracle‑specific, but it’s compact for this kind of mapping.

---

## What This Lab Leaves You With

After completing these tasks, you should be able to:

- Use `TO_CHAR` to build human‑readable sentences and formatted dates.
- Compute review dates using `ADD_MONTHS` and `NEXT_DAY`.
- Replace `NULL` values with meaningful text via `NVL`.
- Express IF‑THEN‑ELSE style logic using `CASE`, searched `CASE`, and `DECODE`.

Which means your SQL can now not only return data—it can **explain** it in full sentences, grade it, and schedule performance reviews. Use this power carefully.
