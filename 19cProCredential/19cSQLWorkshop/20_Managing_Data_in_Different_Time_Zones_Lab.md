# Lesson 20/21 Lab – Managing Data in Different Time Zones (Temporal Shenanigans)

And look, dates were already confusing before you added time zones, daylight saving, and interval math. This lab is about surviving all of that without silently corrupting your timestamps.

You’ll:

- Use `CURRENT_DATE`, `CURRENT_TIMESTAMP`, and `LOCALTIMESTAMP` under different session time zones  
- Compare `DBTIMEZONE` vs `SESSIONTIMEZONE`  
- Convert `DATE` to `TIMESTAMP` (and watch Oracle refuse an unsupported jump)  
- Use `EXTRACT` and intervals to classify and reward employees

Environment: `ora21`, `sql2`.

---

## Task 0 – Cleanup and date format sanity

Run the cleanup script:

```sql
@/home/oracle/labs/sql2/codex/sqlscripts/cleanup_20.sql
```

Then set a readable `NLS_DATE_FORMAT` for your session:

```sql
ALTER SESSION SET NLS_DATE_FORMAT = 'DD-MON-YYYY HH24:MI:SS';
```

This ensures all your date output actually shows the time portion.

---

## Task 1 – Time zone offsets and basic `CURRENT_*` functions

### 1.1 Find time zone offsets

Use `TZ_OFFSET` for a few named regions:

```sql
SELECT tz_offset('US/Pacific-New') AS pacific_new,
       tz_offset('Singapore')      AS singapore,
       tz_offset('Egypt')          AS egypt
FROM   dual;
```

Expect something like:

- US/Pacific‑New → `-08:00`  
- Singapore → `+08:00`  
- Egypt → `+02:00`

### 1.2 Set session time zone to US/Pacific-New

```sql
ALTER SESSION SET TIME_ZONE = 'US/Pacific-New';
```

Check the “current” datetime functions:

```sql
SELECT CURRENT_DATE,
       CURRENT_TIMESTAMP,
       LOCALTIMESTAMP
FROM   dual;
```

All three reflect the **session** time zone (Pacific‑New), and:

- `CURRENT_DATE`: `DATE` type  
- `CURRENT_TIMESTAMP`: `TIMESTAMP WITH TIME ZONE`  
- `LOCALTIMESTAMP`: `TIMESTAMP` (no time zone field)

### 1.3 Switch to Singapore time

```sql
ALTER SESSION SET TIME_ZONE = 'Singapore';
```

Check again:

```sql
SELECT CURRENT_DATE,
       CURRENT_TIMESTAMP,
       LOCALTIMESTAMP
FROM   dual;
```

You should see the timestamps jump ahead about 16 hours compared to US/Pacific (often a date change too), because Singapore is `+08:00`.

### 1.4 Compare database vs session time zones

```sql
SELECT dbtimezone,
       sessiontimezone
FROM   dual;
```

Typical: database stays at `+00:00`, session shows `Asia/Singapore`.  
One is where the **server** lives, the other is where **you** live.

---

## Task 2 – Extract the hire year for department 80

Use `EXTRACT` against the `EMPLOYEES` table:

```sql
SELECT last_name,
       EXTRACT(YEAR FROM hire_date) AS hire_year
FROM   employees
WHERE  department_id = 80;
```

You now have a quick view of when department 80’s folks joined, without resorting to string slicing hacks.

---

## Task 3 – `SAMPLE_DATES` and changing column types

### 3.1 Reset date format to date‑only

```sql
ALTER SESSION SET NLS_DATE_FORMAT = 'DD-MON-YYYY';
```

### 3.2 Create and populate `SAMPLE_DATES`

Run the provided script:

```sql
@/home/oracle/labs/sql2/labs/lab_20_06.sql   -- named lab_26.sql in the text
```

(Use the actual filename from your environment; in the transcript it’s `lab_26.sql`.)

Then:

```sql
SELECT *
FROM   sample_dates;
```

You should see a single `DATE` value (e.g., `07-FEB-2022`) in `DATE_COL`.

### 3.3 Change column type from `DATE` to `TIMESTAMP`

```sql
ALTER TABLE sample_dates
  MODIFY date_col TIMESTAMP;
```

Now the same data has fractional‑second precision available:

```sql
SELECT *
FROM   sample_dates;
```

You’ll see the time component too, thanks to the previous `NLS_DATE_FORMAT` change.

### 3.4 Try to jump straight to `TIMESTAMP WITH TIME ZONE`

Now attempt:

```sql
ALTER TABLE sample_dates
  MODIFY date_col TIMESTAMP WITH TIME ZONE;
```

You should get an error similar to:

> ORA‑01439: column to be modified must be empty to change datatype

The important bit: Oracle does **not** allow you to convert populated `TIMESTAMP` columns to `TIMESTAMP WITH TIME ZONE` with a simple `ALTER`—you’d need to use a staging column/new table approach.

---

## Task 4 – “Needs Review” flag using `EXTRACT` + `CASE`

Goal:  
> For each employee, show last name and “Needs Review” if hired in 2010, otherwise “Not this year”.

```sql
SELECT e.last_name,
       e.hire_date,
       CASE
         WHEN EXTRACT(YEAR FROM e.hire_date) = 2010
           THEN 'Needs Review'
         ELSE 'Not this year'
       END AS review
FROM   employees e
ORDER  BY e.hire_date;
```

This is the datetime equivalent of a performance review calendar—only less passive aggressive.

---

## Task 5 – Years of service awards using `TO_YMINTERVAL`

Goal:

- If employed **15+ years** → “15 years of service”  
- Else if employed **10+ years** → “10 years of service”  
- Else if employed **5+ years** → “5 years of service”  
- Else → “Maybe next year”

Use `TO_YMINTERVAL` so you’re not manually approximating years in days.

```sql
SELECT e.last_name,
       e.hire_date,
       SYSDATE AS today,
       CASE
         WHEN e.hire_date <= SYSDATE - TO_YMINTERVAL('15-0')
           THEN '15 years of service'
         WHEN e.hire_date <= SYSDATE - TO_YMINTERVAL('10-0')
           THEN '10 years of service'
         WHEN e.hire_date <= SYSDATE - TO_YMINTERVAL('05-0')
           THEN '5 years of service'
         ELSE 'Maybe next year'
       END AS awards
FROM   employees e
ORDER  BY e.hire_date;
```

The logic is:

- Take “today” (`SYSDATE`)  
- Subtract 15/10/5 years as proper **year‑month intervals**  
- Compare each employee’s `hire_date` to those boundaries

No need to pretend all years are 365 days long, which is how off‑by‑a‑day bugs are born.

---

By the end of this lab you’ve:

- Set and observed session time zones and used `CURRENT_DATE`, `CURRENT_TIMESTAMP`, and `LOCALTIMESTAMP` correctly  
- Compared `DBTIMEZONE` and `SESSIONTIMEZONE`  
- Converted a `DATE` column to `TIMESTAMP` and seen why `TIMESTAMP WITH TIME ZONE` is stricter  
- Used `EXTRACT` and interval arithmetic to classify employees by hire year and years of service  

Which means you’re now capable of handling time zones and calendar math like an adult, instead of pretending everything happens in UTC and no one ever hears of daylight saving again.  

