## Lesson 16 - Creating Packages (in which PL/SQL gets organized)

Welcome to Lesson 14, where we take the chaos of standalone procedures and functions and put them into neat boxes called packages. Think of packages as the filing cabinet of PL/SQL: tidy on the outside, terrifying inside.

By the end of this lesson, you should be able to:

- Describe packages and list their components
- Create a package with public and private constructs
- Invoke package subprograms
- Explain bodiless packages
- View and drop packages

---

## 1. What Is a Package?

A package is a **schema object** that groups related PL/SQL types, variables, constants, cursors, and subprograms. You already use one: `DBMS_OUTPUT`.

Packages offer:

- Easier maintenance (related things live together)
- Better modularity
- Improved performance (loaded once per session)
- Encapsulation (private logic hidden in body)

---

## 2. Package Components

A package has two parts:

- **Specification**: public interface
- **Body**: implementation and private items

You must compile the **spec** first, then the **body**.

---

## 3. Public vs Private

Anything in the **spec** is public. Anything only in the **body** is private.

Example:

```sql
CREATE OR REPLACE PACKAGE demo_pkg IS
  PROCEDURE update_sal(p_id NUMBER, p_salary NUMBER);
  PROCEDURE get_emp(p_id NUMBER, p_name OUT VARCHAR2);
END demo_pkg;
/

CREATE OR REPLACE PACKAGE BODY demo_pkg IS
  PROCEDURE update_sal(p_id NUMBER, p_salary NUMBER) IS
  BEGIN
    UPDATE employees
    SET salary = p_salary
    WHERE employee_id = p_id;
  END update_sal;

  PROCEDURE get_emp(p_id NUMBER, p_name OUT VARCHAR2) IS
  BEGIN
    SELECT last_name
    INTO p_name
    FROM employees
    WHERE employee_id = p_id;
  END get_emp;
END demo_pkg;
/
```

Private subprograms go only in the body.

---

## 4. Creating the Body (the lazy way)

In SQL Developer:

- Right-click the package spec
- Choose **Create Body**

It copies the signature and inserts TODO markers so you fill in the logic without forgetting parameter names.

---

## 5. Invoking Package Subprograms

Use **package_name.procedure_name**:

```sql
BEGIN
  demo_pkg.update_sal(100, 1);
END;
/
```

You can also EXECUTE it from the command line:

```sql
EXECUTE demo_pkg.update_sal(100, 1)
```

---

## 6. Bodiless Packages

Some packages only need constants or declarations, no body.

Example:

```sql
CREATE OR REPLACE PACKAGE global_consts IS
  c_mile_2_kilo CONSTANT NUMBER := 1.60934;
  c_kilo_2_mile CONSTANT NUMBER := 0.62137;
END global_consts;
/
```

No body required. It is just a container of constants, like a cabinet of measuring tape.

---

## 7. Viewing and Dropping Packages

View source:

```sql
SELECT text
FROM user_source
WHERE name = 'COMM_PKG'
  AND type = 'PACKAGE'
ORDER BY line;
```

For the body, use `type = 'PACKAGE BODY'`.

Drop:

```sql
DROP PACKAGE comm_pkg;
```

Dropping the spec also drops the body. You can drop the body alone with:

```sql
DROP PACKAGE BODY comm_pkg;
```

---

## 8. Guidelines

- Keep the spec minimal (public only)
- Group related subprograms in one package
- Define the spec before the body
- Use private helpers in the body

Packages are only nice if you do not dump everything into one giant drawer.

---

## 9. Quiz Recap

Statement: *The package specification declares public types, variables, constants, cursors, and subprograms.*

Answer: **True**.

---

## 10. Wrap-Up

You now know how to:

- Create package specs and bodies
- Separate public and private logic
- Use bodiless packages
- View and drop packages

Next up: **Lesson 15**, where you actually work with packages and invoke package program units.