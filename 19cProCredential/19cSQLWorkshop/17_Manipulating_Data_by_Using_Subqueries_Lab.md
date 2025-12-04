# Lesson 17 Lab – Manipulating Data by Using Subqueries (DML With Feelings)

And look, by now you’ve written enough subqueries to make a junior developer cry. This lab is about turning those same subqueries loose on **DML**: inserting, updating, and deleting data in ways that are powerful, terrifying, and—if you do it right—correct.

This practice is lighter on typing and heavier on “understand what’s going on,” but we’ll keep it in the same style as the other labs.

You’ll:

- Check your understanding of what subqueries *can* do in DML  
- See how `WITH CHECK OPTION` keeps inline views honest  
- Use correlated subqueries to update and delete rows via the demo scripts

Environment: `ora21`, `sql2`.

---

## Task 1 – Sanity check: what subqueries can do in DML

The warm‑up questions boil down to four core facts:

- Subqueries **are** used to retrieve data via inline views  
- Subqueries **can** be used to copy data from one table to another  
- Subqueries **can** drive updates based on values from other tables  
- Subqueries **can** drive deletes based on matches in other tables

In other words: if there’s a DML statement, there’s probably a way to wedge a subquery into it.

---

## Task 2 – Where can you stick a subquery in an `INSERT`?

You can use a subquery in place of the table name in the `INTO` clause of an `INSERT` statement—*as long as* the inline view is updatable.

Pattern:

```sql
INSERT INTO (
  SELECT ...
  FROM   ...
  WHERE  ...
)
VALUES (...);
```

Things must match:

- The `SELECT` list in the subquery  
- The number and types of values in the `VALUES` clause

If you mismatch them, Oracle will complain, as it should.

---

## Task 3 – `WITH CHECK OPTION`: preventing “stealth” violations

Core point from the practice:

- `WITH CHECK OPTION` **prohibits changing rows that would no longer satisfy the subquery’s `WHERE` clause**.

Mentally map it like this:

> “If this row wouldn’t show up in this view/inline view after your change, I’m not letting you do that change through me.”

Example pattern from the demo:

```sql
INSERT INTO (
  SELECT l.location_id,
         l.street_address,
         c.country_id,
         c.country_name,
         r.region_name
  FROM   locations l
  JOIN   countries c USING (country_id)
  JOIN   regions   r USING (region_id)
  WHERE  r.region_name = 'Europe'
  WITH CHECK OPTION
)
VALUES (3001, '1 Wall St', 'US', 'United States', 'Americas');
```

Result: `view WITH CHECK OPTION where-clause violation` – because “Americas” is not “Europe”, and the row would never appear in that inline view.

The multiple‑choice version of this was:

- *“The `WITH CHECK OPTION` keyword prohibits you from changing rows that are not in the subquery.”* → **True**

---

## Task 4 – Column counts must match

Another quiz point that sounds trivial until it isn’t:

> The `SELECT` list of the subquery used as an insert source must have the **same number of columns** as the `VALUES` / insert column list.

So if your inline view exposes three columns:

```sql
INSERT INTO (
  SELECT col1, col2, col3
  FROM   ...
)
VALUES (v1, v2, v3);  -- 3 values, not 2, not 4
```

Yes, it’s basic—but it’s also the source of half of those “ORA‑00913: too many values” errors.

---

## Task 5 – Correlated subqueries and `DELETE`

Key conceptual question:

> *“You can use a correlated subquery to delete only those rows that also exist in another table.”*  
> Answer: **True**.

Typical pattern (mirroring the lesson demo):

```sql
DELETE FROM empl6 e
WHERE  EXISTS (
         SELECT 1
         FROM   emp_history h
         WHERE  h.employee_id = e.employee_id
       );
```

This:

- Walks rows in `EMPL6`  
- Uses a correlated subquery against `EMP_HISTORY`  
- Deletes only where the `EXISTS` check succeeds

Which is a fancy way of saying “remove current employees who show up on the ‘former employees’ list.”

---

## Task 6 – Run the demo scripts and watch what happens

The practice explicitly tells you to run the demo files to *see* `WITH CHECK OPTION` and correlated DML in action. The exact filenames may vary in your environment, but the patterns look like this:

### 6.1 Demo: Insert through an inline view with `WITH CHECK OPTION`

1. Run the script that creates the inline view / table and shows a working insert with a matching region.
2. Then run the insert with the wrong region (e.g., `Americas`) and observe the error:

   - `ORA-01402: view WITH CHECK OPTION where-clause violation`

This is the runtime proof that the CHECK OPTION is doing its job.

### 6.2 Demo: Correlated `UPDATE`

Script pattern:

```sql
ALTER TABLE empl6
  ADD department_name VARCHAR2(30);

UPDATE empl6 e
SET    department_name = (
         SELECT d.department_name
         FROM   departments d
         WHERE  d.department_id = e.department_id
       );
```

Verify:

```sql
SELECT employee_id,
       department_id,
       department_name
FROM   empl6;
```

You should see `department_name` correctly populated from `DEPARTMENTS`.

### 6.3 Demo: Correlated `DELETE`

Script pattern:

```sql
DELETE FROM empl6 e
WHERE  EXISTS (
         SELECT 1
         FROM   emp_history h
         WHERE  h.employee_id = e.employee_id
       );
```

Followed by:

```sql
SELECT employee_id
FROM   empl6
WHERE  employee_id IN (
         SELECT employee_id
         FROM   emp_history
       );
```

That final query should return **no rows**, meaning you successfully removed duplicates between current and historical employees.

---

## What You’ve Actually Practiced

Even though this practice is mostly conceptual and script‑driven, you’ve reinforced that:

- Subqueries can sit in the `INTO` clause of `INSERT` and drive DML  
- `WITH CHECK OPTION` keeps view/inline view filters from being quietly violated  
- Correlated subqueries are the engine behind “update based on another table” and “delete if it’s also over there” logic

Which means you’re now at the stage where your DML can be both *powerful* and *precisely targeted*—a combination that’s fantastic when you know what you’re doing, and catastrophic when you don’t. So, you know, keep knowing what you’re doing.  

