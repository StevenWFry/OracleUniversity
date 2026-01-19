## Lesson 7 - Writing Control Structures (in which PL/SQL learns manners)

Welcome to Unit 2, where PL/SQL grows up, learns how to make decisions, and then immediately starts looping through those decisions for no clear reason. We are talking about IFs, CASEs, and loops, which are the three ingredients in every control-structure casserole.

By the end of this lesson, you should be able to:

- Identify the uses and types of control structures
- Construct IF statements
- Use CASE statements and CASE expressions
- Construct and identify loops
- Apply guidelines for readable control logic

---

## 0. Where This Lesson Fits (Student Guide Lesson 6, but with jokes)

In the Student Guide, this is **Lesson 6: Writing Control Structures**. The agenda there walks through:

- PL/SQL control structures in general
- IF / ELSIF / ELSE and how NULL behaves in conditions
- CASE expressions vs CASE statements
- LOOP, WHILE, and FOR loops (including rules and examples)
- Nested loops, labels, and the `CONTINUE` statement

Everything you do in this chapter lines up with that structure and feeds directly into **Activity Guide Practice 6**, where you build real IFs and loops instead of just admiring them on slides.

---

## 1. IF Statements (the polite bouncer)

If the condition is true, the code runs. If it is not, it does not. That is the deal.

Simple IF:

```sql
IF v_myage < 11 THEN
  DBMS_OUTPUT.PUT_LINE('I am a child');
END IF;
```

IF with ELSE:

```sql
IF v_myage < 11 THEN
  DBMS_OUTPUT.PUT_LINE('I am a child');
ELSE
  DBMS_OUTPUT.PUT_LINE('I am not a child');
END IF;
```

IF with ELSIF (note: **ELSIF**, not ELSEIF):

```sql
IF v_myage < 11 THEN
  DBMS_OUTPUT.PUT_LINE('I am a child');
ELSIF v_myage < 20 THEN
  DBMS_OUTPUT.PUT_LINE('I am young');
ELSIF v_myage < 30 THEN
  DBMS_OUTPUT.PUT_LINE('I am in my twenties');
ELSIF v_myage < 40 THEN
  DBMS_OUTPUT.PUT_LINE('I am in my thirties');
ELSE
  DBMS_OUTPUT.PUT_LINE('I am always young');
END IF;
```

Tip: uninitialized variables default to `NULL`, which means your IF conditions quietly evaluate to **NULL** and nothing runs. This is why we initialize variables like adults.

---

## 2. CASE Expressions (the SQL-flavored decision tree)

There are two kinds:

- **Simple CASE** (selector after `CASE`)
- **Searched CASE** (selector in each `WHEN`)

Simple CASE:

```sql
CASE job_id
  WHEN 'AD_PRES' THEN salary * 2
  WHEN 'AD_VP'   THEN salary * 1.5
  ELSE salary * 0.5
END
```

Searched CASE:

```sql
CASE
  WHEN job_id = 'AD_PRES' THEN salary * 2
  WHEN job_id = 'AD_VP' AND last_name = 'Kochhar' THEN salary * 1.5
  ELSE salary * 0.5
END
```

Searched CASE lets you stack conditions like a legal argument.

---

## 3. CASE Statements (when you want actions, not values)

CASE statements are PL/SQL control structures and end with **END CASE**.

```sql
CASE v_manager_id
  WHEN 108 THEN
    SELECT department_id, department_name
    INTO v_dept_id, v_dept_name
    FROM departments
    WHERE manager_id = v_manager_id;

    SELECT COUNT(*)
    INTO v_count
    FROM employees
    WHERE department_id = v_dept_id;

    DBMS_OUTPUT.PUT_LINE('You are working in the ' || v_dept_name || ' department');
  WHEN 200 THEN
    -- other action
    NULL;
END CASE;
```

Each `WHEN` does real work, so it must end in a semicolon.

---

## 4. NULL Logic (the chaos in your IF statement)

- `NULL` in a comparison returns `NULL`
- `NOT NULL` is still `NULL`
- If a condition evaluates to `NULL`, the block does not run

Truth tables:

- `TRUE AND TRUE` = TRUE
- `TRUE AND FALSE` = FALSE
- `TRUE AND NULL` = NULL
- `FALSE AND NULL` = FALSE

- `TRUE OR NULL` = TRUE
- `FALSE OR NULL` = NULL
- `NULL OR NULL` = NULL

Yes, NULL is the ghost that haunts every boolean expression.

---

## 5. Loops (the art of doing it again)

Three types:

### 5.1 Basic LOOP

Always runs at least once. You must provide an exit condition:

```sql
DECLARE
  v_val NUMBER := 0;
BEGIN
  LOOP
    DBMS_OUTPUT.PUT_LINE(v_val);
    EXIT WHEN v_val > 10;
    v_val := v_val + 1;
  END LOOP;
END;
/
```

### 5.2 WHILE LOOP

Checks before it runs:

```sql
DECLARE
  v_val NUMBER := 0;
BEGIN
  WHILE v_val < 11 LOOP
    DBMS_OUTPUT.PUT_LINE(v_val);
    v_val := v_val + 1;
  END LOOP;
END;
/
```

### 5.3 FOR LOOP

Runs a fixed number of times. The counter is implied and cannot be modified.

```sql
BEGIN
  FOR i IN 1..10 LOOP
    DBMS_OUTPUT.PUT_LINE(i);
  END LOOP;
END;
/
```

Reverse order:

```sql
FOR i IN REVERSE 1..10 LOOP
  DBMS_OUTPUT.PUT_LINE(i);
END LOOP;
```

You cannot reference `i` outside the loop. It is a loaner, not a roommate.

---

## 6. Example: INSERT with a Loop

Given:

- `v_countryid := 'CA'`
- max `location_id` for Canada is 1900

Loop inserts three new locations: 1901, 1902, 1903, then reports 3 rows added. This is a loop doing exactly what you expect, which is rare and should be celebrated.

---

## 7. Wrap-Up

You now know how to:

- Write IF and CASE logic without spelling ELSIF wrong
- Handle NULL logic like a seasoned adult
- Build loops that exit before the heat death of the universe

Next up: **Practice 6**, where you actually write these control structures and test your patience.
