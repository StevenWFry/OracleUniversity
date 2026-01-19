## Lesson 21 - Tuning the PL/SQL Compiler

OK, my friends, welcome to Lesson 21. This is the part where you tune the compiler like a suspiciously precise espresso machine.

After completing this lesson, you should be able to:

- Use PL/SQL compiler initialization parameters
- Use PL/SQL compile-time warnings

---

## 0. Where This Lesson Fits (Student Guide Lesson 21: tuning the compiler)

In the Student Guide, this matches **Lesson 21: Tuning the PL/SQL Compiler**. The agenda covers:

- Initialization parameters for PL/SQL compilation (`PLSQL_CODE_TYPE`, `PLSQL_OPTIMIZE_LEVEL`)
- Viewing and changing compilation settings
- Compile-time warnings: categories, enabling/disabling, elevating to errors
- Using the `PLSQL_WARNINGS` parameter and the `DBMS_WARNING` package

The sections below align with those topics, and the lab implements **Activity Guide Practice 21**, where you flip native/interpreted modes, toggle warnings, compile a procedure that generates warnings, and then classify warnings using `DBMS_WARNING`.

---

## Compiler Initialization Parameters

The compiler rearranges code for performance. You control it with initialization parameters. The two most important for this course:

- `PLSQL_CODE_TYPE`
- `PLSQL_OPTIMIZE_LEVEL`

Optimization levels:

- 0: off
- 1-3: increasing optimization
- Default in SQL Developer: 2

Code type:

- `INTERPRETED` (default) for debugging
- `NATIVE` for performance (often faster)

---

## Where to Set Them

In SQL Developer:

`Tools` → `Preferences` → `Database` → `PL/SQL Compiler`

At the session level:

```sql
ALTER SESSION SET plsql_code_type = native;
ALTER SESSION SET plsql_optimize_level = 2;
```

Then recompile the object.

---

## Reuse Settings

When recompiling, use `REUSE SETTINGS` so you do not overwrite object-level compiler settings:

```sql
ALTER PACKAGE demopkg COMPILE REUSE SETTINGS;
```

Because nothing says "professional" like not accidentally undoing your own performance tuning.

---

## Compiler Warnings

Three categories:

- Severe (PLW-05000 range)
- Informational (PLW-06000 range)
- Performance (PLW-07000 range)

Enable via SQL Developer or:

```sql
ALTER SESSION SET plsql_warnings = 'enable:all';
```

Or toggle categories:

```sql
ALTER SESSION SET plsql_warnings = 'enable:severe,disable:performance';
```

You can also mark a specific warning as an error:

```sql
ALTER SESSION SET plsql_warnings = 'error:05003';
```

---

## DBMS_WARNING Package

You can query and manage warning settings programmatically:

```sql
DBMS_WARNING.get_warning_setting_string;
DBMS_WARNING.set_warning_setting_string('disable:performance');
DBMS_WARNING.get_category(7203);
```

If you forget what 7203 means, the package will remind you. Politely.

---

## Viewing Warnings

Ways to see warnings:

- `SHOW ERRORS`
- SQL Developer compiler log
- Data dictionary views

The warning prefix is always `PLW-`.

---

## Wrap-Up

You should now be able to:

- Control `PLSQL_CODE_TYPE` and optimization levels
- Enable and tune compile-time warnings
- Use DBMS_WARNING for fine-grained control

Next up: Practice 21, where you flip warnings on and off and pretend not to enjoy it.
