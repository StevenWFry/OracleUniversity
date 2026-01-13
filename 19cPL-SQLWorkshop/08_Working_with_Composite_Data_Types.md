## Lesson 8 - Working with Composite Data Types (in which PL/SQL learns to multitask)

Welcome to Lesson 8, where we stop stuffing single values into tiny variables and start using records and collections like grownups. This is the part where PL/SQL gets a little more sophisticated and also slightly more confusing, but in a fun way.

By the end of this lesson, you should be able to:

- Describe PL/SQL collections and records
- Create user-defined PL/SQL records
- Create a `%ROWTYPE` record
- Create associative arrays (INDEX BY tables)
- Create associative arrays of records

---

## 1. Composite Data Types (more than one value, finally)

Composite types store multiple values in a single structure:

- **Records**: store a row of related data (different data types)
- **Collections**: store a list of values (usually the same data type)

If scalar variables are single Legos, composite types are the entire spaceship.

---

## 2. Records (rows in memory)

### 2.1 Custom Record Type

Define your own record type, then declare a variable of that type:

```sql
DECLARE
  TYPE t_emp IS RECORD (
    fname employees.first_name%TYPE,
    lname employees.last_name%TYPE
  );

  v_emp t_emp;
BEGIN
  SELECT first_name, last_name
  INTO v_emp
  FROM employees
  WHERE employee_id = 100;

  DBMS_OUTPUT.PUT_LINE(v_emp.fname || ' ' || v_emp.lname);
END;
/
```

The order in your SELECT must match the order in the record definition. This is the database equivalent of "put the right shoe on the right foot."

---

## 3. %ROWTYPE (the record shortcut)

If your record matches a table or view, use `%ROWTYPE` and skip the manual definition:

```sql
DECLARE
  v_emp employees%ROWTYPE;
BEGIN
  SELECT *
  INTO v_emp
  FROM employees
  WHERE employee_id = 100;

  DBMS_OUTPUT.PUT_LINE(v_emp.first_name || ' ' || v_emp.last_name);
END;
/
```

Benefits:

- You do not need to know all column types
- It stays aligned if the table changes
- Perfect for row-level INSERTs and UPDATEs

---

## 4. Records Inside Records (yes, it is possible)

You can nest records like Russian dolls:

```sql
DECLARE
  TYPE t_rec IS RECORD (
    v_sal      NUMBER,
    v_minsal   NUMBER DEFAULT 1000,
    v_hiredate employees.hire_date%TYPE,
    v_rec1     employees%ROWTYPE
  );

  v_myrec t_rec;
BEGIN
  v_myrec.v_sal := v_myrec.v_minsal + 500;
  v_myrec.v_hiredate := SYSDATE;

  SELECT *
  INTO v_myrec.v_rec1
  FROM employees
  WHERE employee_id = 100;

  DBMS_OUTPUT.PUT_LINE(v_myrec.v_rec1.first_name);
END;
/
```

Yes, the field access is now `record.field.field`. No, it does not get prettier.

---

## 5. Row-Level INSERT and UPDATE

You can insert or update entire rows using a `%ROWTYPE` record:

```sql
UPDATE employees
SET ROW = v_emp
WHERE employee_id = v_emp.employee_id;
```

```sql
INSERT INTO copy_emp
VALUES v_emp;
```

This is powerful and also dangerous, like giving a toddler a chainsaw.

---

## 6. Collections (associative arrays / index-by tables)

Associative arrays are collections with two parts:

- **Key**: integer or string
- **Value**: scalar or record

Define one:

```sql
DECLARE
  TYPE t_names IS TABLE OF employees.last_name%TYPE
  INDEX BY PLS_INTEGER;

  v_names t_names;
BEGIN
  FOR i IN 100..102 LOOP
    SELECT last_name
    INTO v_names(i)
    FROM employees
    WHERE employee_id = i;
  END LOOP;

  DBMS_OUTPUT.PUT_LINE(v_names(100));
END;
/
```

Now index 100 holds `King`, 101 holds `Kochhar`, 102 holds `De Haan`.

---

## 7. Collection Methods (FIRST, LAST, NEXT, PRIOR, EXISTS)

Associative arrays are not always sequential, so use methods:

- `FIRST` and `LAST` to find boundaries
- `NEXT` and `PRIOR` to walk indexes
- `EXISTS` to avoid missing-key errors

Example:

```sql
FOR j IN v_names.FIRST .. v_names.LAST LOOP
  IF v_names.EXISTS(j) THEN
    DBMS_OUTPUT.PUT_LINE(j || ': ' || v_names(j));
  END IF;
END LOOP;
```

If you delete an index, you leave a hole. `EXISTS` saves you from faceplanting into `NO_DATA_FOUND`.

---

## 8. Wrap-Up

You now know how to:

- Build records and rowtypes
- Store whole rows in memory
- Use associative arrays with keys
- Navigate collections safely

Next up: the practice for Lesson 7, where you turn all of this into actual working code.