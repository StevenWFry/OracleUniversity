## Lesson 4 Lab - Writing Executable Statements (in which scope gets weird)

Welcome to Practice 4. This lab is about scope, nested blocks, comments, and not letting sequences wander around like unsupervised toddlers.

You will:

- Review scoping rules and variable visibility
- Work with nested blocks
- Use comments to remove code temporarily
- Calculate a provident fund contribution

---

## 0. Before You Start (the pre-lab cleanup)

If you ran the code examples for Lesson 4, drop the sequence:

```sql
DROP SEQUENCE my_seq;
```

If you get `sequence does not exist`, that is fine. It just means you did not run the examples.

---

## 1. Scope Check (the visibility quiz)

Given the block in the practice, determine variable values.

**At Position 1**

- `v_weight` = 2 (local `v_weight` starts at 1, then adds 1)
- `v_new_location` = `Western Europe`

**At Position 2**

- `v_weight` = 601 (outer `v_weight` starts at 600, then adds 1)
- `v_message` = `product 10012 is in stock`
- `v_new_location` = **error** (not visible outside the inner block)

---

## 2. Nested Block Values (who sees what)

In the nested block:

- `v_customer` = 201
- `v_name` = Unisport
- `v_credit_rating` = good

In the main block:

- `v_customer` = Womansport
- `v_name` = **error** (not visible)
- `v_credit_rating` = excellent

Yes, scope is petty and unforgiving.

---

## 3. Resume the Session

If you opened a new session, run `lab_03_05_solution.sql` (or your saved script from Lesson 3) to recreate the bind variables.

---

## 4. Convert Bind Variables to Local Variables

- Comment out the `VARIABLE` definitions (single-line comments)
- Comment out the bind variable assignments (multi-line comments)
- Declare local variables instead:

```sql
DECLARE
  v_basic_percent NUMBER := 45;
  v_pf_percent    NUMBER := 12;
  v_fname         VARCHAR2(15);
  v_emp_sal       NUMBER(10);
BEGIN
  SELECT first_name, salary
  INTO v_fname, v_emp_sal
  FROM employees
  WHERE employee_id = 110;

  DBMS_OUTPUT.PUT_LINE('Hello ' || v_fname);
  DBMS_OUTPUT.PUT_LINE('Your salary is ' || v_emp_sal);
  DBMS_OUTPUT.PUT_LINE(
    'Your contribution towards PF is ' ||
    (v_emp_sal * v_basic_percent / 100) * v_pf_percent / 100
  );
END;
/
```

---

## 5. Save and Run

Save as:

- `lab0403solution.sql`

Run the script and confirm output:

- `Hello John`
- `Your salary is 8200`
- `Your contribution towards PF is 442.80`

---

## 6. Wrap-Up

You have now:

- Navigated variable scope without falling in
- Replaced bind variables with locals
- Calculated a provident fund contribution
- Survived nested blocks, which is more than most do

That completes Practice 4.