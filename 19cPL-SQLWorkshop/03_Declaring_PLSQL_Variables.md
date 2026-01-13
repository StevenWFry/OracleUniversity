## Lesson 3 - Declaring PL/SQL Variables (in which names matter and NULL is a menace)

Welcome to Lesson 3. This is where we give PL/SQL a memory, then immediately argue about what to call it, then accidentally forget to initialize it and watch it stare back at us like a disappointed cat.

By the end of this lesson, you should be able to:

- Recognize valid and invalid identifiers
- List the uses of variables
- Declare and initialize variables
- Describe common data types
- Identify the benefits of `%TYPE`
- Declare, use, and print bind variables

---

## 1. What Variables Are (the labeled jars in your database pantry)

Variables are labeled storage locations used to hold and process data inside a PL/SQL block. You must declare them before you use them, which feels obvious until you do not, and then Oracle judges you.

Example use case:

```sql
DECLARE
  v_sal employees.salary%TYPE;
BEGIN
  SELECT salary
  INTO v_sal
  FROM employees
  WHERE employee_id = 100;

  UPDATE employees
  SET salary = v_sal + 100
  WHERE employee_id = 100;
END;
/
```

The variable holds one value (salary) so you can reuse it without re-querying like a maniac.

---

## 2. Naming Rules (the law, but with more semicolons)

A variable name:

- Must start with a letter
- Can include letters, numbers, and special characters (`$`, `_`, `#`)
- Must be 30 characters or fewer
- Cannot be a reserved word (no, you cannot name a variable `VARCHAR2` and call it art)

---

## 3. Declaring and Initializing Variables (aka, please set a value)

You declare variables in the **declaration** section. You can optionally initialize them there too.

```sql
DECLARE
  v_hiredate DATE;                -- NULL by default
  v_location VARCHAR2(13) := 'Atlanta';
  v_deptno   NUMBER(2) NOT NULL := 10;
  c_comm     CONSTANT NUMBER := 1400;
BEGIN
  NULL;
END;
/
```

Rules to remember:

- `NOT NULL` variables must be initialized
- `CONSTANT` variables must be initialized and cannot be reassigned
- Use meaningful names (do not name your variable `employee_id` if it stores a column called `employee_id`, unless you enjoy confusion as a lifestyle)

---

## 4. Naming Conventions (so your future self does not haunt you)

Recommended prefixes:

- `v_` for variables
- `c_` for constants
- `p_` for parameters
- `b_` for bind variables
- `r_` for records
- `e_` for exceptions

Consistency is more important than any specific prefix, but choose one and stick with it like it is the last donut in the break room.

---

## 5. Common Data Types (the cast of characters)

**Character types**

- `CHAR`, `VARCHAR2`, `NCHAR`, `NVARCHAR2`, `CLOB`, `NCLOB`

**Numeric types**

- `NUMBER`, `PLS_INTEGER`, `SIMPLE_INTEGER`, `BINARY_INTEGER`, `BINARY_FLOAT`, `BINARY_DOUBLE`

**Boolean**

- `BOOLEAN` with values `TRUE`, `FALSE`, or `NULL`

**Date and time**

- `DATE`, `TIMESTAMP`, `TIMESTAMP WITH TIME ZONE`, `INTERVAL YEAR TO MONTH`, `INTERVAL DAY TO SECOND`

---

## 6. Initializing with Assignment or DEFAULT (two flavors, same dessert)

You can initialize with either `:=` or `DEFAULT`:

```sql
DECLARE
  v_rate NUMBER := 0.07;
  v_tax  NUMBER DEFAULT 0.08;
BEGIN
  NULL;
END;
/
```

Use one style consistently. Most people use `:=` because it feels like the inside of a database gave a tiny nod.

---

## 7. Strings and the Alternative Quote Operator (apostrophes are tiny landmines)

Strings use single quotes. Apostrophes require escaping, or you can use the alternative quote operator:

```sql
DECLARE
  v_event VARCHAR2(30);
BEGIN
  v_event := q'[Father''s Day]';
  DBMS_OUTPUT.PUT_LINE(v_event);
END;
/
```

You can use different delimiters with `q` as long as they match, like database-themed bookends.

---

## 8. Implicit vs Explicit Conversions (do not trust magic)

Oracle can do implicit conversions, but it is better to be explicit:

```sql
DECLARE
  v_date DATE;
BEGIN
  v_date := TO_DATE('February 02, 2000', 'Month DD, YYYY');
END;
/
```

Explicit conversions are clearer, safer, and less likely to break your index usage when you least expect it.

---

## 9. The %TYPE Attribute (because hard-coding types is a trap)

`%TYPE` lets a variable match the data type of a table column:

```sql
DECLARE
  v_sal employees.salary%TYPE;
BEGIN
  NULL;
END;
/
```

Benefits:

- Automatically stays in sync if the column changes
- Reduces maintenance and errors
- Saves you from quietly drifting into a type mismatch crisis

---

## 10. NOT NULL and CONSTANT in Practice (the strict parents)

```sql
DECLARE
  v_name VARCHAR2(25) NOT NULL := 'Steven';
  c_tax  CONSTANT NUMBER := 0.08;
BEGIN
  NULL;
END;
/
```

If you skip initialization for `NOT NULL` or `CONSTANT`, you get a compile error, which is Oracle's way of saying, "No. Absolutely not."

---

## 11. Wrap-Up

You now know how to declare, name, and initialize PL/SQL variables, and why `%TYPE` is your new best friend who occasionally saves you from yourself. Next up, we use them in actual blocks and start breaking things on purpose.