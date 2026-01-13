## Lesson 7 - Writing Control Structures (Part 2: Loops, Labels, and CONTINUE)

We are back in the loop section of Lesson 7, because loops are like potato chips: you cannot just have one. This part covers WHILE and FOR loops, loop guidelines, nested loops, labels, and the CONTINUE statement.

---

## 1. WHILE Loops (checks before it leaps)

A WHILE loop evaluates the condition **before** it runs:

```sql
DECLARE
  v_counter NUMBER := 1;
BEGIN
  WHILE v_counter <= 3 LOOP
    -- insert row
    v_counter := v_counter + 1;
  END LOOP;
END;
/
```

Result: rows 1901, 1902, 1903 are inserted, just like the basic loop.

---

## 2. FOR Loops (fixed number of iterations)

A FOR loop runs a defined number of times using an **implicit counter**. You may read it, but you may not change it.

```sql
BEGIN
  FOR i IN 1..3 LOOP
    -- insert using i
  END LOOP;
END;
/
```

Reverse order is also allowed:

```sql
FOR i IN REVERSE 1..3 LOOP
  -- insert using i
END LOOP;
```

Rules:

- The counter exists **only** inside the loop
- You cannot assign to the counter
- Bounds should never be NULL

---

## 3. When to Use Each Loop

- **Basic LOOP**: when it must run at least once
- **WHILE LOOP**: when the condition must be evaluated first
- **FOR LOOP**: when the number of iterations is known

Pick the right one. Your future self will thank you and your code will stop looking like a spaghetti disaster.

---

## 4. Labels and Nested Loops (loops inside loops inside loops)

You can name loops with labels and use `EXIT` to break specific loops:

```sql
<<outer_loop>>
FOR i IN 1..5 LOOP
  <<inner_loop>>
  FOR j IN 1..5 LOOP
    EXIT outer_loop WHEN i = 3;
    EXIT inner_loop WHEN j = 4;
  END LOOP;
END LOOP;
```

Labels are your emergency exits in a maze of looping doom.

---

## 5. CONTINUE (the polite skip)

`CONTINUE` jumps to the next iteration of the loop. It is the "not today" of PL/SQL.

Simple example:

```sql
BEGIN
  FOR i IN 1..10 LOOP
    DBMS_OUTPUT.PUT_LINE('Top part: ' || i);

    CONTINUE WHEN i > 5;

    DBMS_OUTPUT.PUT_LINE('Bottom part: ' || i);
  END LOOP;
END;
/
```

Result:

- For `i` = 1..5, both lines print
- For `i` = 6..10, only the top line prints

Nested loop example:

```sql
BEGIN
  FOR i IN 1..5 LOOP
    FOR j IN 1..5 LOOP
      CONTINUE WHEN i + j > 5;
      DBMS_OUTPUT.PUT_LINE('i=' || i || ', j=' || j);
    END LOOP;
  END LOOP;
END;
/
```

The inner loop stops early based on the condition, then control returns to the outer loop. This is how you prevent the inner loop from eating your entire afternoon.

---

## 6. Quiz Recap

Statement: *There are three types of loops: basic, FOR, and WHILE.*

Answer: **True**.

---

## 7. Wrap-Up

You now know how to:

- Use WHILE and FOR loops correctly
- Avoid messing with implicit counters
- Add labels to escape nested loops
- Use CONTINUE to skip parts of a loop without killing it

Next up: **Practice 6**, where you actually write these control structures and make them behave.