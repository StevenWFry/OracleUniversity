## Lesson 13 - Creating Procedures (in which modularity saves your sanity)

Welcome to Unit 4, where subprograms become the main character. This lesson focuses on procedures: why they exist, how to build them, and how to call them without making a mess.

By the end of this lesson, you should be able to:

- Explain the benefits of modularized, layered design
- Create and call procedures
- Use formal and actual parameters
- Use positional, named, and mixed notation
- Identify parameter modes (IN, OUT, IN OUT)
- Handle exceptions in procedures
- Drop a procedure and view its source

---

## 1. Why Modularize?

Modularization means breaking large tasks into smaller, reusable modules. Benefits include:

- Easy maintenance
- Better security and integrity
- Improved performance
- Reusability

If Jack needs salary changes and promotions, Alice does not write one 500-line block. She writes procedures like `raise_sal` and `promote_emp`.

---

## 2. Creating a Procedure

Basic structure:

```sql
CREATE OR REPLACE PROCEDURE hello_proc (p_name IN VARCHAR2) IS
BEGIN
  DBMS_OUTPUT.PUT_LINE('Hello ' || p_name);
END;
/
```

Notes:

- `OR REPLACE` avoids re-granting privileges
- `IS` and `AS` are interchangeable
- You can compile in SQL Developer or via script

---

## 3. Calling a Procedure

From an anonymous block:

```sql
BEGIN
  hello_proc('Brent');
END;
/
```

From the SQL prompt:

```sql
EXECUTE hello_proc('Brent')
```

Or from SQL Developer’s object tree with **Run**.

---

## 4. Parameters (formal vs actual)

- **Formal parameters** are defined in the procedure signature.
- **Actual parameters** are the values you pass when you call it.

Example:

```sql
PROCEDURE raise_sal (p_id IN NUMBER, p_percent IN NUMBER)
```

`p_id` and `p_percent` are formal parameters. `raise_sal(176, 10)` contains the actual values.

---

## 5. Parameter Modes

- **IN** (default): value passed in, treated as constant
- **OUT**: value returned to caller, must be a variable
- **IN OUT**: value passed in and returned, must be initialized variable

Quick comparison:

- IN can have defaults
- OUT and IN OUT cannot have defaults
- OUT and IN OUT must be variables

---

## 6. Positional, Named, and Mixed Notation

Positional:

```sql
raise_sal(176, 10)
```

Named:

```sql
raise_sal(p_percent => 10, p_id => 176)
```

Mixed (positional first, then named):

```sql
raise_sal(176, p_percent => 10)
```

Once you switch to named notation, you cannot go back to positional.

---

## 7. Defaults for IN Parameters

```sql
PROCEDURE raise_sal (p_id IN NUMBER, p_percent IN NUMBER DEFAULT 10)
```

Calls:

- `raise_sal(176)` uses 10%
- `raise_sal(176, 5)` overrides default

---

## 8. Exceptions Inside Procedures

Exceptions should live inside procedures so valid work is preserved:

```sql
EXCEPTION
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('Err adding dept');
```

If you do not handle exceptions, one failure can roll back all inserts.

---

## 9. Dropping and Viewing Procedures

Drop:

```sql
DROP PROCEDURE procedure_name;
```

View source (command line):

```sql
SELECT text
FROM user_source
WHERE name = 'HELLO_PROC'
  AND type = 'PROCEDURE'
ORDER BY line;
```

If the source is wrapped, you will see unreadable gibberish. That is intentional.

---

## 10. Quiz Recap

Invalid procedure call example:

- Mixed notation where named parameters come first, then positional: **invalid**

---

## 11. Wrap-Up

You now know how to:

- Build procedures with parameters
- Use IN, OUT, and IN OUT modes
- Call procedures with positional, named, or mixed notation
- Handle exceptions
- Drop and inspect procedures

Next up: **Practice 11**, where you create and invoke procedures for insert/update/delete and handle exceptions like a pro.