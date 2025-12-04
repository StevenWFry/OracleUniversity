## Lesson 4 Lab – Using Single-Row Functions to Customize Output (or: teaching your columns to dress nicely)

And look, your data currently shows up in whatever format the database finds convenient: dates in weird formats, salaries with too many decimals, names shouting in all caps. This lab is about making that output look intentional.

You will:

- Display the system date in a friendly way.
- Use numeric functions like `ROUND`.
- Use character functions like `INITCAP`, `LENGTH`, `LPAD`, and `RPAD`.
- Use date functions like `MONTHS_BETWEEN` and arithmetic on dates.
- Calculate years and months/weeks of service.

Use the **`ora1`** connection and the **SQL1** labs folder for all scripts in this lesson.

---

## Task 1 – Show the System Date

**Goal:** Confirm you can call a built‑in date function and alias the result.

1. Open a new worksheet.
2. Display the current system date with the column labeled `Date`:

   ```sql
   SELECT SYSDATE AS "Date"
   FROM   dual;
   ```

3. Run it and verify that it shows today’s date.

This is the “what year is it?” sanity check for all later date calculations.

---

## Task 2 – New Salary with a 15.5% Increase

**Goal:** Use arithmetic and `ROUND` to compute whole‑number salaries and save the query to a file.

HR wants a report that shows:

- `employee_id`
- `last_name`
- `salary`
- `salary` increased by **15.5%**, rounded to a whole number, labeled `New Salary`.

1. New worksheet.
2. Write the query:

   ```sql
   SELECT employee_id,
          last_name,
          salary,
          ROUND(salary * 1.155, 0) AS "New Salary"
   FROM   employees;
   ```

3. Save the statement as `lab_04_02.sql` in the SQL1 labs folder.
4. Close and reopen the file, then run it to confirm it still works.

---

## Task 3 – (Covered in Task 2) – Execute `lab_04_02.sql`

If you haven’t already, open `lab_04_02.sql` from disk and execute it. Confirm the `New Salary` column shows integer values.

---

## Task 4 – Show the Increase Amount as a Separate Column

**Goal:** Add a new column that shows the **difference** between the new and old salaries, and save to a new file.

Use the same calculation from Task 2 and compute the increase as:

> `New Salary – salary`

1. Copy the query from `lab_04_02.sql` into a new worksheet.
2. Add a new column that repeats the `ROUND` expression and subtracts the old salary:

   ```sql
   SELECT employee_id,
          last_name,
          salary,
          ROUND(salary * 1.155, 0)              AS "New Salary",
          ROUND(salary * 1.155, 0) - salary     AS increase
   FROM   employees;
   ```

3. Run the query and confirm the `increase` column shows the raise amount.
4. Save as `lab_04_04.sql` and re‑run to verify.

Note: you **cannot** use the alias `"New Salary"` inside the same `SELECT` list expression; you must repeat the calculation or use a subquery.

---

## Task 5 – Name Formatting and Length for A/M Last Names

**Goal:** Use character and length functions, plus `LIKE` filtering.

HR wants, for employees whose last name starts with `A` or `M`:

- Last name with first letter uppercase, others lowercase.
- Length of the last name.
- Sorted by last name.

1. New worksheet.
2. Write the query:

   ```sql
   SELECT INITCAP(last_name)      AS "Name",
          LENGTH(last_name)       AS "Length"
   FROM   employees
   WHERE  last_name LIKE 'A%'
      OR  last_name LIKE 'M%'
   ORDER  BY last_name;
   ```

3. Run it and confirm the name formatting and lengths.

### 5b – Prompt for the Starting Letter

Modify the query so that the starting letter is **not hard‑coded**, but is prompted from the user.

```sql
SELECT INITCAP(last_name) AS "Name",
       LENGTH(last_name)  AS "Length"
FROM   employees
WHERE  last_name LIKE '&start_letter%'
ORDER  BY last_name;
```

- When prompted for `start_letter`, enter `H` to see all last names starting with H.

Run it as an **individual statement** (green triangle) for clearer output.

### 5c – Make the Prompt Case-Insensitive

Right now, entering `h` (lowercase) will not match names stored as `Higgins`. Fix that by normalizing both sides to the same case.

```sql
SELECT INITCAP(last_name) AS "Name",
       LENGTH(last_name)  AS "Length"
FROM   employees
WHERE  UPPER(last_name) LIKE UPPER('&start_letter') || '%'
ORDER  BY last_name;
```

- Now, entering `h` or `H` returns the same rows.

---

## Task 6 – Months Worked for Each Employee

**Goal:** Use `MONTHS_BETWEEN` and `ROUND` to calculate months of service, and sort by the result.

Requirements:

- Show `last_name`.
- Show number of **months employed** as `MONTHS_WORKED`.
- Use `MONTHS_BETWEEN(SYSDATE, hire_date)`.
- Round to the nearest whole month.
- Sort by `MONTHS_WORKED`.

1. New worksheet.
2. Query:

   ```sql
   SELECT last_name,
          ROUND(MONTHS_BETWEEN(SYSDATE, hire_date)) AS MONTHS_WORKED
   FROM   employees
   ORDER  BY MONTHS_WORKED;
   ```

3. Run it. Your `MONTHS_WORKED` values may differ from the book’s, because they depend on **today’s date**.

If you experiment with double‑quoted aliases (`"MONTHS_WORKED"`), remember to use the **exact same case and quotes** in `ORDER BY`.

---

## Task 7 – Salary Left-Padded with Dollar Signs

**Goal:** Use `LPAD` to format salary as a fixed-width, currency-like field.

HR wants a report that shows:

- `last_name`
- `salary` formatted to **15 characters**, left‑padded with `$`, column labeled `SALARY`.

1. New worksheet.
2. Write and run:

   ```sql
   SELECT last_name,
          LPAD(TO_CHAR(salary), 15, '$') AS SALARY
   FROM   employees;
   ```

Each salary appears right‑aligned with leading `$` padding, total width 15 characters.

---

## Task 8 – Salaries Represented as Asterisks

**Goal:** Use `RPAD` and numeric calculations to turn salaries into tiny bar charts.

Requirement:

- Show `last_name`.
- Show a column where each `*` (asterisk) represents **$1,000 of salary**.
- Sort data in **descending** order of salary.
- Label the column `SALARIES_IN_ASTERISK` (or similar).

A reasonable implementation:

```sql
SELECT last_name,
       salary,
       RPAD('', ROUND(salary / 1000), '*') AS SALARIES_IN_ASTERISK
FROM   employees
ORDER  BY salary DESC;
```

Notes:

- `salary / 1000` converts salary to “thousands”.
- `ROUND` gives a whole number of stars—no half‑asterisks.
- `RPAD('', n, '*')` creates a string with `n` asterisks.

You can tweak the rounding strategy (up, down, truncation) depending on how generous you want the visualization to be.

---

## Task 9 – Tenure in Weeks for Department 90

**Goal:** Use date arithmetic and `TRUNC` to measure duration of employment.

HR wants:

- `last_name`.
- Number of **weeks employed** as `TENURE`.
- Only for employees in **department 90**.
- `TENURE` truncated to 0 decimal places.
- Results sorted by `TENURE` in **descending** order.

1. New worksheet.
2. Write:

   ```sql
   SELECT last_name,
          TRUNC( (SYSDATE - hire_date) / 7 ) AS TENURE
   FROM   employees
   WHERE  department_id = 90
   ORDER  BY TENURE DESC;
   ```

Explanation:

- `SYSDATE - hire_date` = number of days employed.
- Divide by 7 → weeks employed.
- `TRUNC` removes the fractional part.

Values will shift over time as `SYSDATE` moves forward.

---

## What This Lab Leaves You With

After completing this lab, you should be able to:

- Use **numeric**, **character**, and **date** functions in `SELECT` statements.
- Format names, salaries, and dates so reports don’t look like raw dumps.
- Calculate derived values like **new salaries**, **salary increases**, **months worked**, and **tenure in weeks**.
- Use padding and repetition (`LPAD`, `RPAD`) to create quick visualizations.

In short, your queries now produce output that looks less like machine noise and more like something a human might actually put into a slide deck—which is both a win and a terrible enabler of more slide decks.
