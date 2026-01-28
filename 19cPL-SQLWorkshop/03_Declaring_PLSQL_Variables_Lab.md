## Lesson 3 Lab - Declaring PL/SQL Variables (in which names live or die)

Welcome to Practice 3. This lab is mostly about naming things correctly and initializing them so Oracle does not glare at you.

You will:

- Identify valid identifiers
- Identify valid variable declarations
- Modify and save a script with new variables
- Declare and print bind variables, using the same script flow as the Activity Guide

---

## 1. Valid vs Invalid Identifiers (name your children wisely)

Rules recap:

- Must be 30 characters or fewer
- Must start with a letter
- Can include `$`, `#`, and `_`
- Cannot include apostrophes or start with a special character

Examples (mirroring the Activity Guide set):

- `today` – valid
- `last_name` – valid
- `today's_date` – invalid (apostrophe)
- `Number_of_days_in_February_this_year` – invalid (too long)
- `Isleap$year` – valid
- `#number` – invalid (cannot start with `#`)
- `NUMBER#` – valid
- `number1to7` – valid

---

## 2. Valid vs Invalid Declarations (this is where people trip)

Examples (again, straight from the Activity Guide, lightly roasted):

**A. Valid**

```sql
number_of_copies PLS_INTEGER;
```

**B. Invalid** (constant not initialized)

```sql
PRINTER_NAME CONSTANT VARCHAR2(10);
```

**C. Invalid** (string missing quotes)

```sql
deliver_to VARCHAR2(10) := Johnson;
```

**D. Valid**

```sql
by_when DATE := CURRENT_DATE + 1;
```

Why:

- B is invalid because constants must be initialized at declaration time.
- C is invalid because string literals must be in single quotes.
- A and D are fine; Oracle does not object.

---

## 3. Anonymous Block Question (the quiz in disguise)

The Activity Guide gives you this block:

```sql
DECLARE
  v_fname VARCHAR2(20);
  v_lname VARCHAR2(15) DEFAULT 'fernandez';
BEGIN
  DBMS_OUTPUT.PUT_LINE(v_fname || ' ' || v_lname);
END;
/
```

Correct statement (also the official solution):

- **The block executes successfully and prints “fernandez.”**

`v_fname` is `NULL`, but concatenating `NULL` with a space and a last name still produces something printable, and `DEFAULT` is absolutely allowed for `VARCHAR2`.

---

## 4. Modify the Hello World Script

Open your `lab_02_02_soln.sql` script from Practice 2 and update it, just like the Activity Guide tells you to.

Add these variables:

```sql
DECLARE
  v_today    DATE := SYSDATE;
  v_tomorrow v_today%TYPE;
BEGIN
  v_tomorrow := v_today + 1;

  DBMS_OUTPUT.PUT_LINE('Hello World');
  DBMS_OUTPUT.PUT_LINE('Today is: ' || v_today);
  DBMS_OUTPUT.PUT_LINE('Tomorrow is: ' || v_tomorrow);
END;
/
```

Save as:

- `lab_03_04_soln.sql`

Run it to confirm output.

---

## 5. Add Bind Variables

Add bind variables at the top of the script, using the same names and pattern as the Activity Guide:

```sql
VARIABLE b_basic_percent NUMBER
VARIABLE b_pf_percent NUMBER
```

Inside the block, assign values:

```sql
:b_basic_percent := 45;
:b_pf_percent := 12;
```

Terminate with `/` and then print:

```sql
PRINT b_basic_percent
PRINT b_pf_percent

-- Or, if your tool supports it,
PRINT
```

This prints all bind variables at once.

---

## 6. Wrap-Up

You have now:

- Identified valid identifiers
- Fixed invalid declarations
- Built a date-based anonymous block
- Created and printed bind variables

That is Practice 3, and you survived it with your naming conventions intact.
