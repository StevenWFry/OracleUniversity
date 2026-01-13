## Lesson 6 Lab - Writing Control Structures (in which loops earn their keep)

Welcome to Practice 6. This lab has you creating tables, inserting rows with logic, and building a tiny ASCII performance review system using asterisks. It is as silly as it sounds and yet extremely effective.

You will:

- Create the `MESSAGES` table
- Insert numbers 1..10 while skipping 6 and 8
- Create a copy of the EMP table with a STARS column
- Generate star ratings based on salary

---

## 1. Create the MESSAGES Table

Run the script:

- `lab0601.sql`

This creates a `MESSAGES` table with a single column called `RESULTS`.

---

## 2. Insert 1..10, Skip 6 and 8

Use a FOR loop and an IF condition:

```sql
BEGIN
  FOR i IN 1..10 LOOP
    IF i = 6 OR i = 8 THEN
      NULL;
    ELSE
      INSERT INTO messages (results)
      VALUES (i);
    END IF;
  END LOOP;

  COMMIT;
END;
/

SELECT *
FROM messages;
```

Expected output: 1..10 except 6 and 8.

---

## 3. Create the EMP Table with STARS

Run the script:

- `lab62.sql`

This creates a copy of the `EMPLOYEES` table and adds a `STARS` column (VARCHAR2(50)).

---

## 4. Generate Stars Based on Salary

Goal: one asterisk for every $1,000 in salary, rounded up.

```sql
DECLARE
  v_empno    emp.employee_id%TYPE := 176;
  v_asterisk emp.stars%TYPE := NULL;
  v_sal      emp.salary%TYPE;
BEGIN
  SELECT NVL(ROUND(salary/1000), 0)
  INTO v_sal
  FROM emp
  WHERE employee_id = v_empno;

  FOR i IN 1..v_sal LOOP
    v_asterisk := v_asterisk || '*';
  END LOOP;

  UPDATE emp
  SET stars = v_asterisk
  WHERE employee_id = v_empno;

  COMMIT;
END;
/

SELECT employee_id, salary, stars
FROM emp
WHERE employee_id = 176;
```

Save as:

- `lab602_sol.sql`

If the salary is 8600, you should see 9 asterisks.

---

## 5. Wrap-Up

You have now:

- Used loops with conditional logic
- Built a star rating based on salaries
- Updated and queried a table using PL/SQL

That concludes Practice 6.