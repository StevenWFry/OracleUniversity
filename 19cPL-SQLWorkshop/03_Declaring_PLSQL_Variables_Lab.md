## Lesson 3 Lab - Declaring PL/SQL Variables (in which names live or die)

Welcome to Practice 3. This lab is mostly about naming things correctly and initializing them so Oracle does not glare at you.

You will:

- Identify valid identifiers
- Identify valid variable declarations
- Modify and save a script with new variables
- Declare and print bind variables

---

## 1. Valid vs Invalid Identifiers (name your children wisely)

Rules recap:

- Must be 30 characters or fewer
- Must start with a letter
- Can include `$`, `#`, and `_`
- Cannot include apostrophes or start with a special character

Examples:

- `today` - valid
- `last_name` - valid
- `today's_date` - invalid (apostrophe)
- `number_of_days_in_february_this_year` - invalid (too long)
- `isleap$year` - valid
- `$amount` - invalid (cannot start with special character)
- `number#` - valid
- `number1to7` - valid

---

## 2. Valid vs Invalid Declarations (this is where people trip)

Examples:

**A. Valid**

```sql
number_of_copies PLS_INTEGER;
```

**B. Invalid** (constant not initialized)

```sql
printer_name CONSTANT VARCHAR2(10);
```

**C. Invalid** (string missing quotes)

```sql
deliver_to VARCHAR2(10) := Johnson;
```

**D. Valid**

```sql
by_when DATE := CURRENT_DATE + 1;
```

Note: if your activity guide labels this list incorrectly, you are not imagining it.

---

## 3. Anonymous Block Question (the quiz in disguise)

Given the block, the correct answer is:

- **The block executes successfully and prints Fernandez**

The variable is declared and initialized, and `DEFAULT` is perfectly valid for `VARCHAR2`.

---

## 4. Modify the Hello World Script

Open your `lab22solution.sql` script from Practice 2 and update it.

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

- `lab0304solution.sql`

Run it to confirm output.

---

## 5. Add Bind Variables

Add bind variables at the top of the script:

```sql
VARIABLE b_basic_percent NUMBER
VARIABLE bpf_percent NUMBER
```

Inside the block, assign values:

```sql
:b_basic_percent := 45;
:bpf_percent := 12;
```

Terminate with `/` and then print:

```sql
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