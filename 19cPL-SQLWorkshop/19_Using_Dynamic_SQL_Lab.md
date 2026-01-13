## Lesson 17 Lab - Using Dynamic SQL

This practice builds a package that uses native dynamic SQL (NDS) to create, populate, update, delete, and drop tables, plus a package that recompiles objects in your schema.

Before starting:

- Run `cleanup_17.sql`

---

## 1. Create TABLE_PKG Specification

```sql
CREATE OR REPLACE PACKAGE table_pkg IS
  PROCEDURE make (
    p_table_name IN VARCHAR2,
    p_columns    IN VARCHAR2
  );

  PROCEDURE add_row (
    p_table_name IN VARCHAR2,
    p_id         IN NUMBER,
    p_name       IN VARCHAR2
  );

  PROCEDURE upd_row (
    p_table_name IN VARCHAR2,
    p_set        IN VARCHAR2,
    p_where      IN VARCHAR2 DEFAULT NULL
  );

  PROCEDURE del_row (
    p_table_name IN VARCHAR2,
    p_where      IN VARCHAR2 DEFAULT NULL
  );

  PROCEDURE remove (
    p_table_name IN VARCHAR2
  );
END table_pkg;
/
```

---

## 2. Create TABLE_PKG Body

```sql
CREATE OR REPLACE PACKAGE BODY table_pkg IS

  PROCEDURE execute_stmt (p_stmt IN VARCHAR2) IS
  BEGIN
    EXECUTE IMMEDIATE p_stmt;
  END execute_stmt;

  PROCEDURE make (p_table_name IN VARCHAR2, p_columns IN VARCHAR2) IS
  BEGIN
    EXECUTE IMMEDIATE 'CREATE TABLE ' || p_table_name || ' (' || p_columns || ')';
  END make;

  PROCEDURE add_row (p_table_name IN VARCHAR2, p_id IN NUMBER, p_name IN VARCHAR2) IS
  BEGIN
    EXECUTE IMMEDIATE
      'INSERT INTO ' || p_table_name || ' VALUES (:1, :2)'
      USING p_id, p_name;
  END add_row;

  PROCEDURE upd_row (p_table_name IN VARCHAR2, p_set IN VARCHAR2, p_where IN VARCHAR2 DEFAULT NULL) IS
    v_stmt VARCHAR2(1000);
  BEGIN
    v_stmt := 'UPDATE ' || p_table_name || ' SET ' || p_set;
    IF p_where IS NOT NULL THEN
      v_stmt := v_stmt || ' WHERE ' || p_where;
    END IF;
    EXECUTE IMMEDIATE v_stmt;
  END upd_row;

  PROCEDURE del_row (p_table_name IN VARCHAR2, p_where IN VARCHAR2 DEFAULT NULL) IS
    v_stmt VARCHAR2(1000);
  BEGIN
    v_stmt := 'DELETE FROM ' || p_table_name;
    IF p_where IS NOT NULL THEN
      v_stmt := v_stmt || ' WHERE ' || p_where;
    END IF;
    EXECUTE IMMEDIATE v_stmt;
  END del_row;

  PROCEDURE remove (p_table_name IN VARCHAR2) IS
    v_cur_id INTEGER;
  BEGIN
    v_cur_id := DBMS_SQL.OPEN_CURSOR;
    DBMS_SQL.PARSE(v_cur_id, 'DROP TABLE ' || p_table_name, DBMS_SQL.NATIVE);
    DBMS_SQL.CLOSE_CURSOR(v_cur_id);
  END remove;

END table_pkg;
/
```

---

## 3. Use TABLE_PKG

Create table:

```sql
EXECUTE table_pkg.make('my_contacts', 'id NUMBER, name VARCHAR2(25)')
```

Insert rows:

```sql
EXECUTE table_pkg.add_row('my_contacts', 1, 'Lauren')
EXECUTE table_pkg.add_row('my_contacts', 2, 'Nancy')
EXECUTE table_pkg.add_row('my_contacts', 3, 'Sunitha')
EXECUTE table_pkg.add_row('my_contacts', 4, 'Valli')
```

Update:

```sql
EXECUTE table_pkg.upd_row('my_contacts', "name='Nancy Greenberg'", 'id = 2')
```

Delete:

```sql
EXECUTE table_pkg.del_row('my_contacts', 'id = 3')
```

Drop:

```sql
EXECUTE table_pkg.remove('my_contacts')
```

---

## 4. Create COMPILE_PKG

Specification:

```sql
CREATE OR REPLACE PACKAGE compile_pkg IS
  PROCEDURE make (p_name IN VARCHAR2);
END compile_pkg;
/
```

Body:

```sql
CREATE OR REPLACE PACKAGE BODY compile_pkg IS

  PROCEDURE execute_stmt (p_stmt IN VARCHAR2) IS
  BEGIN
    DBMS_OUTPUT.PUT_LINE(p_stmt);
    EXECUTE IMMEDIATE p_stmt;
  END execute_stmt;

  FUNCTION get_type (p_name IN VARCHAR2) RETURN VARCHAR2 IS
    v_proc_type user_objects.object_type%TYPE;
  BEGIN
    SELECT object_type
    INTO v_proc_type
    FROM user_objects
    WHERE object_name = UPPER(p_name)
      AND ROWNUM = 1;

    RETURN v_proc_type;
  EXCEPTION
    WHEN NO_DATA_FOUND THEN
      RETURN NULL;
  END get_type;

  PROCEDURE make (p_name IN VARCHAR2) IS
    v_type VARCHAR2(30);
  BEGIN
    v_type := get_type(p_name);

    IF v_type IS NOT NULL THEN
      execute_stmt('ALTER ' || v_type || ' ' || p_name || ' COMPILE');
    ELSE
      RAISE_APPLICATION_ERROR(-20001, 'Object does not exist: ' || p_name);
    END IF;
  END make;

END compile_pkg;
/
```

---

## 5. Test COMPILE_PKG

```sql
EXECUTE compile_pkg.make('EMPLOYEE_REPORT')
EXECUTE compile_pkg.make('EMP_PKG')
EXECUTE compile_pkg.make('EMP_DATA')
```

Expected: first two compile, last one errors with "Object does not exist".

---

## Wrap-Up

You built two packages: one for dynamic DDL/DML and one for recompiling PL/SQL objects dynamically.