## Lesson 7 Lab - Working with Composite Data Types (in which records and arrays do the heavy lifting)

Welcome to Practice 7. This lab has you using `%ROWTYPE` records and associative arrays to retrieve and print country and department data without manually juggling a dozen variables.

You will:

- Use a `%ROWTYPE` record to print country details
- Load department names into an associative array
- Convert that array into an index-by table of records

---

## 1. Country Record with %ROWTYPE

Create a block that prints details for a country ID.

```sql
SET SERVEROUTPUT ON
SET VERIFY OFF

DECLARE
  v_countryid     VARCHAR2(20) := 'CA';
  v_country_record countries%ROWTYPE;
BEGIN
  SELECT *
  INTO v_country_record
  FROM countries
  WHERE country_id = UPPER(v_countryid);

  DBMS_OUTPUT.PUT_LINE('Country ID: ' || v_country_record.country_id ||
                       ' Country Name: ' || v_country_record.country_name ||
                       ' Region: ' || v_country_record.region_id);
END;
/
```

Test with `DE`, `UK`, and `US` by changing `v_countryid`.

---

## 2. Department Names in an Associative Array

Create and run a block to load 10 department names (IDs 10..100) into an index-by table.

```sql
SET SERVEROUTPUT ON

DECLARE
  TYPE dept_table_type IS TABLE OF departments.department_name%TYPE
  INDEX BY PLS_INTEGER;

  my_dept_table dept_table_type;
  f_loop_count  NUMBER(2) := 10;
  v_deptno      NUMBER(4) := 0;
BEGIN
  FOR i IN 1..f_loop_count LOOP
    v_deptno := v_deptno + 10;
    SELECT department_name
    INTO my_dept_table(i)
    FROM departments
    WHERE department_id = v_deptno;
  END LOOP;

  FOR i IN 1..f_loop_count LOOP
    DBMS_OUTPUT.PUT_LINE(my_dept_table(i));
  END LOOP;
END;
/
```

Save as:

- `lab_07_02_soln.sql`

---

## 3. Convert to an Index-By Table of Records

Modify the type to store the full department row:

```sql
TYPE dept_table_type IS TABLE OF departments%ROWTYPE
INDEX BY PLS_INTEGER;
```

Update the SELECT and output:

```sql
SELECT *
INTO my_dept_table(i)
FROM departments
WHERE department_id = v_deptno;

DBMS_OUTPUT.PUT_LINE('Department ID: ' || my_dept_table(i).department_id);
DBMS_OUTPUT.PUT_LINE('Department Name: ' || my_dept_table(i).department_name);
DBMS_OUTPUT.PUT_LINE('Manager ID: ' || my_dept_table(i).manager_id);
DBMS_OUTPUT.PUT_LINE('Location ID: ' || my_dept_table(i).location_id);
```

Optional save as:

- `lab_07_03_soln.sql`

---

## 4. Wrap-Up

You have now:

- Printed a full country record with `%ROWTYPE`
- Built an associative array of department names
- Expanded that array to store full department records

Composite types are now officially in your toolbox. Use them wisely, or at least dramatically.