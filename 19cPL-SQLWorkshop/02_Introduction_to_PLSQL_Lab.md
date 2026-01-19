## Lesson 2 Lab - Introduction to PL/SQL (in which blocks live or die)

Welcome to Practice 2. This lab is short but important, like a short film that teaches you to respect semicolons.

You will:

- Identify which PL/SQL blocks execute successfully
- Create and run a simple block that outputs `Hello World`
- Save the script to the lab folder using the same naming the Activity Guide uses

---

## 1. Know Your Folders (where things actually live)

Working directory:

- `/home/oracle/labs/plsf/labs`

Solutions directory:

- `/home/oracle/labs/plsf/soln`

You will save your scripts in the labs folder and compare with the solutions when needed.

---

## 2. Which Blocks Run? (the survival test)

Try each block from the Activity Guide in a worksheet and see what happens.

**A**

```sql
BEGIN
  COMMIT;
END;
/
```

**B**

```sql
DECLARE
  v_amount INTEGER(10);
END;
/
```

**C**

```sql
DECLARE
BEGIN
END;
/
```

**D**

```sql
SET SERVEROUTPUT ON;

DECLARE
  v_amount INTEGER(10);
BEGIN
  DBMS_OUTPUT.PUT_LINE(v_amount);
END;
/
```

According to the official Activity Guide commentary (and also reality):

- **A and D** execute successfully
- **B** fails because the executable section is missing
- **C** technically has `DECLARE` and `BEGIN`, but with no executable statement the block is invalid in this form

---

## 3. Create Hello World (the ritual)

Create and run a simple block that prints `Hello World`:

```sql
SET SERVEROUTPUT ON

BEGIN
  DBMS_OUTPUT.PUT_LINE('Hello World');
END;
/
```

Run it as a script (`F5`). You should see the output.

---

## 4. Save the Script

Save your script using the same naming as the Activity Guide:

- File name: `lab_02_02_soln.sql`
- Location: `/home/oracle/labs/plsf/labs`

---

## 5. Wrap-Up

You have confirmed which blocks execute, written a working anonymous block, and saved it to the lab folder. This is the foundation for everything that follows. No pressure.
