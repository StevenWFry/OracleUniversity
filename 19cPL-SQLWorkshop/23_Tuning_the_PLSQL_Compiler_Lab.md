## Lesson 21 Lab - Tuning the PL/SQL Compiler

This practice walks through compiler settings, native compilation, and warning management. It is basically a wellness check for your PL/SQL.

Before starting:

- Run `cleanup_21.sql`

---

## 1. Display Compiler Settings

Query `USER_PLSQL_OBJECT_SETTINGS` for `ADD_DEPARTMENT`:

```sql
SELECT object_name,
       object_type,
       plsql_code_type,
       plsql_optimize_level
FROM user_plsql_object_settings
WHERE object_name = 'ADD_DEPARTMENT';
```

Note the current `PLSQL_CODE_TYPE` and optimization level.

---

## 2. Enable Native Compilation (Session)

```sql
ALTER SESSION SET plsql_code_type = native;
```

Recompile `ADD_DEPARTMENT`, then re-run the query above to confirm it is now native.

Switch back to interpreted if the lab requires it:

```sql
ALTER SESSION SET plsql_code_type = interpreted;
```

---

## 3. Disable Warnings in SQL Developer

Go to:

`Tools` → `Preferences` → `Database` → `PL/SQL Compiler`

Disable all warning categories.

---

## 4. Create and Compile UNREACHABLE_CODE

Open `lab_21_04.sql` and run it to create the `unreachable_code` procedure.

Expected:

- No warnings (because you disabled them)

---

## 5. Re-enable Warnings and Recompile

Turn warnings back on in SQL Developer and recompile `unreachable_code`.

Expected warnings:

- `PLW-06002` (unreachable code)
- Possibly an AUTHID warning depending on your SQL Developer version

---

## 6. View Warnings in USER_ERRORS

```sql
SELECT * FROM user_errors;
```

The `TEXT` column should include warning details.

---

## 7. Identify Warning Categories

Enable server output and use `DBMS_WARNING`:

```sql
SET SERVEROUTPUT ON;
BEGIN
  DBMS_OUTPUT.PUT_LINE(DBMS_WARNING.get_category(&warning_number));
END;
/
```

Test:

- 5050 → SEVERE
- 6075 → INFORMATIONAL
- 7100 → PERFORMANCE

Yes, the numbers are weird, and yes, Oracle chose them on purpose.

---

## Wrap-Up

You just:

- Toggled compiler settings
- Switched between interpreted and native
- Managed warning categories
- Used DBMS_WARNING to classify errors

Practice 21 complete.
