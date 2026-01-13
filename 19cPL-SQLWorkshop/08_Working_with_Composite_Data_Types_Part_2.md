## Lesson 8 - Working with Composite Data Types (Part 2: Collections, Arrays, and the Chaos in Between)

Welcome back. This part focuses on collections, especially associative arrays (index-by tables), and then briefly nods at nested tables and varrays. We do not fully cover nested tables or varrays in this class, but we will politely acknowledge their existence.

---

## 1. Associative Arrays of Records (full rows in memory)

If you want to store an entire row inside an index-by table, define the collection as a table of `%ROWTYPE`:

```sql
DECLARE
  TYPE t_emp_tab IS TABLE OF employees%ROWTYPE
  INDEX BY PLS_INTEGER;

  v_emp_tab t_emp_tab;
BEGIN
  SELECT *
  INTO v_emp_tab(100)
  FROM employees
  WHERE employee_id = 100;

  DBMS_OUTPUT.PUT_LINE(v_emp_tab(100).last_name);
  DBMS_OUTPUT.PUT_LINE(v_emp_tab(100).salary);
END;
/
```

You now have all columns available, not just a single scalar field. This is called an **index-by table of records**.

---

## 2. Associative Arrays (key/value, not necessarily sequential)

Key points:

- Keys can be integers or strings
- Indexes do **not** need to be sequential
- Use `FIRST`, `LAST`, `NEXT`, `PRIOR`, `EXISTS` to navigate

Example:

```sql
DECLARE
  TYPE email_table IS TABLE OF employees.email%TYPE
  INDEX BY PLS_INTEGER;

  email_list email_table;
BEGIN
  email_list(100) := 'SKING';
  email_list(105) := 'DAUSTIN';
  email_list(110) := 'JCHEN';

  DBMS_OUTPUT.PUT_LINE(email_list(100));
END;
/
```

If you delete a key, you create a hole. Use `EXISTS` to avoid `NO_DATA_FOUND`.

---

## 3. Collection Methods (navigation tools)

- `FIRST` / `LAST`: boundary keys
- `NEXT` / `PRIOR`: step through keys
- `COUNT`: number of elements

Example loop:

```sql
FOR i IN email_list.FIRST .. email_list.LAST LOOP
  IF email_list.EXISTS(i) THEN
    DBMS_OUTPUT.PUT_LINE(email_list(i));
  END IF;
END LOOP;
```

---

## 4. Nested Tables (mentioned, not mastered)

Nested tables:

- Store a set of values
- Can be persisted as a column type in a table
- Indexes are sequential (1, 2, 3...)
- You can delete elements, which may create gaps

Example (conceptual):

```sql
TYPE dept_mail IS TABLE OF VARCHAR2(20);
```

---

## 5. VARRAYs (nested tables with a bouncer)

VARRAYs are like nested tables but with a maximum size:

```sql
TYPE email IS VARRAY(5) OF VARCHAR2(20);
```

- Sequential subscripts
- Upper limit enforced
- Extend/trim allowed

Think of it as a guest list with a strict capacity.

---

## 6. Summary of Collection Types

- **Associative Arrays**: key/value, PL/SQL only
- **Nested Tables**: sequential, can be stored in tables
- **VARRAYs**: sequential with max size, can be stored in tables

---

## 7. Quiz Recap

Use `%ROWTYPE` when:

- You are not sure about the structure of a table
- You want to select an entire row

Use `%TYPE` when:

- You need a variable based on a single column or another variable

---

## 8. Wrap-Up

You now understand:

- Index-by tables of scalars and records
- Collection navigation methods
- The difference between associative arrays, nested tables, and varrays

Next up: **Practice 7**, where you use associative arrays and PL/SQL records in real code.