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

## 1. Lexical Units (the alphabet soup of PL/SQL)

Lexical units are the building blocks of PL/SQL. They include letters, numerals, spaces, tabs, returns, and symbols. They fall into four types:

- **Identifiers**: `v_fname`, `c_percent`
- **Delimiters**: `;`, `,`, `+`, `-`
- **Literals**: `'John'`, `428`, `TRUE`
- **Comments**: `-- single line` or `/* multi-line */`

Yes, comments are officially part of the language. They are also the only thing preventing future you from crying in the break room.

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

## 9. Wrap-Up

You now know how PL/SQL is built, what functions it allows, how nesting works, and how to keep code readable. That is a lot of responsibility. Use it wisely.

Next up: **Practice 4**, where you review scoping rules and write blocks that actually behave.