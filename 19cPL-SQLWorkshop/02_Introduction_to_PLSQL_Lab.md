## Lesson 2 Lab - Introduction to PL/SQL (in which blocks live or die)

Welcome to Practice 2. This lab is short but important, like a short film that teaches you to respect semicolons.

You will:

- Identify which PL/SQL blocks execute successfully
- Create and run a simple block that outputs `Hello World`
- Save the script to the lab folder

---

## 1. Know Your Folders (where things actually live)

Working directory:

- `home/oracle/labs/plsf`

Solutions directory:

- `home/oracle/labs/plsfsoln`

You will save your scripts in the labs folder and compare with the solutions when needed.

---

## 2. Which Blocks Run? (the survival test)

Try each block in a worksheet and see what happens.

**A. This runs**

```sql
BEGIN
  NULL;
END;
/
```

**B. This fails** (missing `BEGIN` and executable section)

```sql
DECLARE
  v_test NUMBER;
END;
/
```

**C. This fails** (has `DECLARE` and `BEGIN`, but no executable statement)

```sql
DECLARE
  v_test NUMBER;
BEGIN
END;
/
```

**D. This runs** (but prints nothing because the variable is not initialized)

```sql
DECLARE
  v_test VARCHAR2(20);
BEGIN
  DBMS_OUTPUT.PUT_LINE(v_test);
END;
/
```

Summary:

- **A and D** execute successfully
- **B and C** fail because the executable section is missing or empty

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

Save your script as:

- `lab22solution.sql`

Suggested location:

- `home/oracle/labs/plsf`

---

## 5. Wrap-Up

You have confirmed which blocks execute, written a working anonymous block, and saved it to the lab folder. This is the foundation for everything that follows. No pressure.