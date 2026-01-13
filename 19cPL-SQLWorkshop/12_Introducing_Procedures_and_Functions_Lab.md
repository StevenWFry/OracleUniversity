## Lesson 10 Lab - Introducing Stored Procedures and Functions

This practice converts an anonymous block into a stored procedure, then modifies it to accept a parameter.

You will:

- Convert a block into `greet`
- Invoke the procedure from an anonymous block
- Add a parameter to `greet`

---

## 1. Convert the Anonymous Block to a Procedure

Open `sol_3.sql` from `/home/oracle/labs/plsf/soln` and copy **Task 4** into a new worksheet.

Modify it into a procedure named `greet`:

```sql
CREATE OR REPLACE PROCEDURE greet IS
BEGIN
  DBMS_OUTPUT.PUT_LINE('Hello World');
  DBMS_OUTPUT.PUT_LINE('Today is ' || TO_CHAR(SYSDATE, 'DD-MON-YY'));
  DBMS_OUTPUT.PUT_LINE('Tomorrow is ' || TO_CHAR(SYSDATE + 1, 'DD-MON-YY'));
END greet;
/
```

Note: remove `SET SERVEROUTPUT ON` from the procedure script.

Run it and confirm `greet` appears under Procedures.

Save as:

- `lab_10_01_soln.sql`

---

## 2. Invoke the Procedure

Clear the worksheet and run:

```sql
SET SERVEROUTPUT ON

BEGIN
  greet;
END;
/
```

Expected output:

- Hello World
- Today is ...
- Tomorrow is ...

---

## 3. Add a Parameter

Drop and recreate the procedure with a parameter:

```sql
DROP PROCEDURE greet;

CREATE OR REPLACE PROCEDURE greet (p_name VARCHAR2) IS
BEGIN
  DBMS_OUTPUT.PUT_LINE('Hello ' || p_name);
  DBMS_OUTPUT.PUT_LINE('Today is ' || TO_CHAR(SYSDATE, 'DD-MON-YY'));
  DBMS_OUTPUT.PUT_LINE('Tomorrow is ' || TO_CHAR(SYSDATE + 1, 'DD-MON-YY'));
END greet;
/
```

Save as:

- `lab_10_02_soln.sql`

---

## 4. Invoke with a Name

```sql
SET SERVEROUTPUT ON

BEGIN
  greet('Brent');
END;
/
```

Expected output:

- Hello Brent
- Today is ...
- Tomorrow is ...

---

## 5. Wrap-Up

You have now:

- Converted an anonymous block into a procedure
- Compiled and invoked it
- Added a parameter and called it with a value

Practice 10 complete.