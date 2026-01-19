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

In PL/SQL, you almost always use a small set of core types over and over. Knowing when to pick which one saves you from weird truncation, rounding, and “why is this `NULL` again?” moments.

### 5.1 Character types (text and more text)

Use these for names, codes, messages, and anything not meant for arithmetic.

- `VARCHAR2(size)`
  - Variable-length character data up to 32,767 bytes in PL/SQL.
  - Best general-purpose string type for most columns and variables.
  - If the value is shorter than `size`, it only uses the needed space.
  - Example:
    ```sql
    v_first_name VARCHAR2(50) := 'Steven';
    ```

- `CHAR(size)`
  - Fixed-length character data.
  - Pads with spaces up to `size`, which can cause subtle comparison issues.
  - Use only when you truly want fixed-width data (e.g., fixed codes).
  - Example:
    ```sql
    v_region_code CHAR(2) := 'US';
    ```

- `NCHAR(size)` / `NVARCHAR2(size)`
  - Unicode character data (national character set).
  - Use when you must support multilingual data with different character sets.
  - `NCHAR` is fixed-length; `NVARCHAR2` is variable-length.

- `CLOB` / `NCLOB`
  - Character Large Objects: very large text (documents, logs, etc.).
  - Not commonly used in simple PL/SQL variables in this course, but crucial for big text in real systems.
  - Example:
    ```sql
    v_big_text CLOB;
    ```

### 5.2 Numeric types (for counting, calculating, and overthinking)

Pick a numeric type based on how exact you need to be and whether you care about performance in tight loops.

- `NUMBER(p, s)`
  - The “do almost anything” numeric type.
  - `p` = precision (total digits), `s` = scale (digits to the right of decimal).
  - Great for money, quantities, percentages.
  - Example:
    ```sql
    v_salary NUMBER(8, 2) := 4500.50;   -- up to 999,999.99
    ```

- `PLS_INTEGER`
  - Signed integer; stored in machine-native format.
  - Very fast in pure PL/SQL arithmetic and loops.
  - Only for whole numbers (no decimals).
  - Example:
    ```sql
    v_counter PLS_INTEGER := 0;
    ```

- `SIMPLE_INTEGER`
  - Like `PLS_INTEGER` but:
    - Cannot be `NULL`.
    - No overflow checks (wraps around), which can be faster but dangerous.
  - Good for performance-critical, well-controlled loops.

- `BINARY_INTEGER`
  - Older integer type; still works but mostly replaced by `PLS_INTEGER`.
  - You will see it in legacy code.

- `BINARY_FLOAT` / `BINARY_DOUBLE`
  - IEEE floating-point types (approximate values).
  - Good for scientific/engineering calculations where tiny rounding is acceptable.
  - Not ideal for money, because 0.1 + 0.2 may not be exactly 0.3.

### 5.3 Boolean (true, false, and “I refuse to answer”)

- `BOOLEAN`
  - Can be `TRUE`, `FALSE`, or `NULL`.
  - Only exists in PL/SQL (you cannot have a `BOOLEAN` table column).
  - Great for conditions and branching logic.
  - Example:
    ```sql
    v_is_active BOOLEAN := TRUE;
    ```

### 5.4 Date and time types (when did this happen?)

- `DATE`
  - Stores date and time (down to the second).
  - Most common type for general “when” information.
  - Example:
    ```sql
    v_hiredate DATE := SYSDATE;
    ```

- `TIMESTAMP`
  - Like `DATE` but with fractional seconds (up to 9 digits).
  - Use when you care about sub-second precision.
  - Example:
    ```sql
    v_created_at TIMESTAMP := SYSTIMESTAMP;
    ```

- `TIMESTAMP WITH TIME ZONE`
  - `TIMESTAMP` plus stored time zone.
  - Useful for distributed/global applications where “local time” is not enough.

- `TIMESTAMP WITH LOCAL TIME ZONE`
  - Stored in the database in a normalized time zone; displayed in the user’s session time zone.
  - Handy when users are all over the planet but you want sane display behavior.

- `INTERVAL YEAR TO MONTH`
  - A span of years and months (e.g., “3 years, 2 months”).
  - Good for things like contract durations or seniority.
  - Example:
    ```sql
    v_duration INTERVAL YEAR TO MONTH := INTERVAL '2-6' YEAR TO MONTH; -- 2 years 6 months
    ```

- `INTERVAL DAY TO SECOND`
  - A span of days, hours, minutes, seconds, and fractional seconds.
  - Great for measuring elapsed time between events.
  - Example:
    ```sql
    v_elapsed INTERVAL DAY TO SECOND := INTERVAL '5 12:30:00' DAY TO SECOND;
    ```

---

In real PL/SQL programs, you will often avoid hard-coding these types and instead use `%TYPE` or `%ROWTYPE` to match table columns—but understanding the underlying types gives you a fighting chance when things go weird.


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