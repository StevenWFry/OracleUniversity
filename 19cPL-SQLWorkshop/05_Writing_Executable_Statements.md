## Lesson 5 - Writing Executable Statements (in which PL/SQL finally does things)

Welcome to Lesson 5, where PL/SQL stops setting the table and starts serving the meal. We are going to talk about lexical units, functions, nested blocks, and sequences, which sounds like four separate topics until you realize they are all ways to keep your code from exploding.

By the end of this lesson, you should be able to:

- Identify lexical units in a PL/SQL block
- Use built-in SQL functions in PL/SQL
- Know when to use implicit vs explicit conversions
- Write nested blocks and qualify variables with labels
- Write readable, indented code
- Use sequences in PL/SQL expressions

---

## 0. Where This Lesson Fits (the Student Guide slide in human)

In the Student Guide, this maps to **Lesson 4: Writing Executable Statements**. The official agenda bullets are:

- Lexical units in a PL/SQL block
- PL/SQL block syntax and guidelines
- Commenting code
- SQL functions in PL/SQL
- Using sequences in PL/SQL blocks
- Nested blocks, scope, and qualifiers
- Operators and programming guidelines

Everything below hits those same beats; the only difference is that here, you’re allowed to laugh at them.

---

## 1. Lexical Units (the alphabet soup of PL/SQL)

Lexical units are the building blocks of PL/SQL. They include letters, numerals, spaces, tabs, returns, and symbols. They fall into four types:

- **Identifiers**: `v_fname`, `c_percent`
- **Delimiters**: `;`, `,`, `+`, `-`
- **Literals**: `'John'`, `428`, `TRUE`
- **Comments**: `-- single line` or `/* multi-line */`

Yes, comments are officially part of the language. They are also the only thing preventing future you from crying in the break room.

---

## 1a. Block Syntax and Guidelines (so the compiler doesn’t file a complaint)

The Student Guide calls this “block syntax and guidelines,” which is a fancy way of saying:

- Every statement ends with a **semicolon** (`;`).
- The block ends with `END;` followed by a `/` when you run it as a script.
- Indentation should reflect block structure (nested blocks indented further).
- Put **one statement per line** unless you enjoy reading sideways.

Example of a nicely-behaved mini block:

```sql
DECLARE
  v_weight  NUMBER(3) := 600;
  v_message VARCHAR2(255) := 'Product 10012';
BEGIN
  v_weight  := v_weight + 1;
  v_message := v_message || ' is in stock';
END;
/
```

Your future debugging self will silently thank you.

---

## 2. Literals and Formatting (make it readable)

Character literals use single quotes. Numbers can be normal or scientific notation.

Use SQL Developer formatting:

- Right-click > **Format**
- Or press **Ctrl+F7**

It will indent, wrap, and uppercase keywords, which makes your code look like it knows what it is doing.

---

## 3. Comments (quietly explaining your chaos)

- Single line: `-- like this`
- Multi-line: `/* like this */`

If you comment out a line that includes a statement, that statement does not run. This is obvious, but still worth repeating because someone will inevitably try it and be confused.

---

## 4. Functions in PL/SQL (yes, but not all of them)

You can use most **single-row functions** in PL/SQL expressions:

- String functions (`LENGTH`, `SUBSTR`, etc.)
- Numeric functions
- Date functions
- Conversion functions

But:

- **Group functions** (like `MAX`, `AVG`, `COUNT`) are **not** allowed directly in PL/SQL expressions.
- **DECODE** is also not allowed in PL/SQL expressions.

You *can* use them inside a SQL statement embedded in PL/SQL:

```sql
DECLARE
  v_max_sal NUMBER;
BEGIN
  SELECT MAX(salary)
  INTO v_max_sal
  FROM employees;
END;
/
```

---

## 5. Sequences in PL/SQL (numbers that keep climbing)

Sequences generate unique numbers without you manually guessing the last ID like a panicked intern.

Example:

```sql
CREATE SEQUENCE my_seq START WITH 1 INCREMENT BY 1;
```

Use it inside PL/SQL:

```sql
DECLARE
  v_id NUMBER;
BEGIN
  v_id := my_seq.NEXTVAL;
  DBMS_OUTPUT.PUT_LINE('New ID: ' || v_id);
END;
/
```

Every call to `NEXTVAL` advances the sequence. It never forgets. It never forgives.

---

## 6. Nested Blocks (blocks within blocks within blocks)

You can nest PL/SQL blocks inside executable or exception sections. Inner blocks can see outer variables, but outer blocks cannot see inner variables.

Example:

```sql
DECLARE
  v_outer VARCHAR2(20) := 'GLOBAL VARIABLE';
BEGIN
  DECLARE
    v_inner VARCHAR2(20) := 'INNER VARIABLE';
  BEGIN
    DBMS_OUTPUT.PUT_LINE(v_inner);
    DBMS_OUTPUT.PUT_LINE(v_outer);
  END;

  DBMS_OUTPUT.PUT_LINE(v_outer);
END;
/
```

Attempting to reference `v_inner` outside its block results in a compile error.

---

## 6a. Variable Scope and Visibility (a.k.a. the Practice 4 trap)

The Activity Guide’s **Practice 4: Writing Executable Statements** exists almost entirely to make you think about scope.

Quick rules, straight from that exercise:

- Inner blocks **can see** outer variables (until they redeclare them).
- Outer blocks **cannot see** inner variables (ever).
- If you declare the **same name** in an inner block, it hides the outer one inside that inner block.

That is why, in the classic `v_weight` / `v_new_locn` example from the Activity Guide:

- `v_weight` at the inner “position 1” is `2` (local copy).
- `v_new_locn` at “position 1” is `'Western Europe'`.
- `v_weight` at the outer “position 2” is `601`.
- `v_new_locn` at “position 2” is **illegal** – that variable never left the inner block.

If this feels petty and vindictive, congratulations: you understand scope.

---

## 7. Labels and Qualifiers (breaking scope on purpose)

If you use the same variable name in nested blocks, PL/SQL defaults to the **nearest** scope. Labels let you explicitly reference the outer block:

```sql
<<outer>>
DECLARE
  v_date_of_birth DATE := DATE '1972-04-20';
BEGIN
  DECLARE
    v_date_of_birth DATE := DATE '2002-12-12';
  BEGIN
    DBMS_OUTPUT.PUT_LINE('Child: ' || v_date_of_birth);
    DBMS_OUTPUT.PUT_LINE('Parent: ' || outer.v_date_of_birth);
  END;
END;
/
```

Labels do not let outer blocks access inner variables. That door stays locked.

---

## 8. Operators and Readability (make it survivable)

Use:

- Arithmetic, logical, concatenation operators
- Parentheses to control precedence

And please, for the sake of every future maintainer:

- Indent your code
- Comment complex sections
- Use consistent naming conventions

SQL Developer's formatter is your friend. Treat it better than you treat your phone battery.

---

## 8a. Programming Guidelines (the “please don’t do that” slide)

The Student Guide wraps this lesson with a set of programming guidelines that are basically professional peer pressure:

- Prefer **meaningful identifiers** over single letters for anything that lives longer than three lines.
- Keep blocks **short and focused** – if your `BEGIN ... END;` runs for four pages, it wants to be a subprogram.
- Use **comments** for *why*, not for “SELECT means select”.
- Avoid clever side effects in expressions; clarity wins over showing off.

In other words: write code as if someone else will have to maintain it. Because they will. It might even be you.

---

## 9. Wrap-Up

You now know how PL/SQL is built, what functions it allows, how nesting works, and how to keep code readable. That is a lot of responsibility. Use it wisely.

Next up: **Practice 4**, where you review scoping rules and write blocks that actually behave.
