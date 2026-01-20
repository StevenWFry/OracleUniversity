## Lesson 20 - Design Considerations for PL/SQL Code

OK, my friends, welcome to Lesson 20. This is the chapter where your PL/SQL stops looking like a garage band and starts sounding like an actual orchestra. Or at least a competent brass section.

After completing this lesson, you should be able to:

- Write standardized PL/SQL code
- Grant and control runtime privileges of subprograms
- Create and use autonomous transactions
- Use `NOCOPY`, `PARALLEL_ENABLE`, and `DETERMINISTIC`
- Use result caching for optimization
- Use bulk binding for optimization

---

## 0. Where This Lesson Fits (Student Guide Lesson 20: design considerations)

In the Student Guide, this is **Lesson 20: Design Considerations for PL/SQL Code**. The agenda there walks through:

- Standardizing constants, exceptions, and exception handling
- Local subprograms
- Definer’s vs invoker’s rights, including `AUTHID CURRENT_USER`
- Autonomous transactions
- Performance hints: `NOCOPY`, `PARALLEL_ENABLE`, `RESULT_CACHE`, `DETERMINISTIC`
- `RETURNING` and bulk binding (`FORALL`, `BULK COLLECT`)

The material that follows mirrors that list, and your lab is the practical piece of **Activity Guide Practice 20**, where you bulk-fetch employees into a package-level collection and log new employees with an autonomous transaction.

---

## Standardizing Code

Standardization:

- Promotes consistency and reuse
- Makes maintenance less painful
- Enforces company-wide conventions

Use bodiless packages for:

- Constants
- Named exceptions

Example: a standardized error package with named exceptions bound to error codes via `PRAGMA EXCEPTION_INIT`. You define the exception once and reuse it everywhere:

```sql
CREATE OR REPLACE PACKAGE app_errors IS
  e_invalid_dept   EXCEPTION;
  e_negative_salary EXCEPTION;

  PRAGMA EXCEPTION_INIT(e_invalid_dept,   -20001);
  PRAGMA EXCEPTION_INIT(e_negative_salary, -20002);
END app_errors;
/ 
```

Later, in your procedures and functions:

```sql
RAISE app_errors.e_invalid_dept;
```

---

## Local Subprograms

You can define a function or procedure inside another subprogram. That local subprogram is only visible inside the parent block.

Example: `employee_sal` declares a local `tax` function and uses it in the executable section. Local subprograms keep helper logic close and private, like a tiny, well-behaved gremlin:

```sql
CREATE OR REPLACE PROCEDURE employee_sal (
  p_emp_id  IN  employees.employee_id%TYPE,
  p_net_sal OUT employees.salary%TYPE
) IS
  -- local helper, only visible inside EMPLOYEE_SAL
  FUNCTION tax (p_gross employees.salary%TYPE) RETURN NUMBER IS
  BEGIN
    RETURN p_gross * 0.2;
  END tax;

  v_gross employees.salary%TYPE;
BEGIN
  SELECT salary INTO v_gross
  FROM employees
  WHERE employee_id = p_emp_id;

  p_net_sal := v_gross - tax(v_gross);
END employee_sal;
/ 
```

---

## Definer's Rights vs Invoker's Rights

By default, subprograms run with **definer’s rights**.

If you add:

```
AUTHID CURRENT_USER
```
you get **invoker's rights**. That means:

- Object names resolve in the invoker's schema
- Privileges come from the user calling the program

Use this when you want reusable code across multiple schemas without forcing everything into one owner's privileges.

Minimal example:

```sql
CREATE OR REPLACE PROCEDURE show_user_objects
AUTHID CURRENT_USER
IS
  v_count PLS_INTEGER;
BEGIN
  SELECT COUNT(*)
  INTO v_count
  FROM user_objects;

  DBMS_OUTPUT.PUT_LINE('You own ' || v_count || ' objects.');
END show_user_objects;
/ 
```

---

## Autonomous Transactions

Autonomous transactions are independent. They:

- Commit or roll back on their own
- Do not affect the calling transaction
- Require `PRAGMA AUTONOMOUS_TRANSACTION` and a `COMMIT`

Typical use:

- Logging
- Audit trails
- Error reporting

Example logging procedure:

```sql
CREATE OR REPLACE PROCEDURE log_event (
  p_message IN VARCHAR2
) IS
  PRAGMA AUTONOMOUS_TRANSACTION;
BEGIN
  INSERT INTO app_log (log_id, log_ts, message)
  VALUES (app_log_seq.NEXTVAL, SYSTIMESTAMP, p_message);

  COMMIT;  -- required in autonomous transactions
END log_event;
/ 
```

If the main transaction fails, the audit log still survives. It is basically your database version of "pics or it didn't happen."

---

## Performance Directives

### NOCOPY

Pass `OUT` and `IN OUT` parameters by reference instead of by value.

Benefits:

- Less memory overhead
- Faster parameter passing

Downside: if the subprogram fails, you cannot rely on partial values.

```sql
CREATE OR REPLACE PROCEDURE add_bonus (
  p_emp_id  IN  employees.employee_id%TYPE,
  p_bonus   IN OUT NOCOPY NUMBER
) IS
BEGIN
  UPDATE employees
  SET salary = salary + p_bonus
  WHERE employee_id = p_emp_id;

  -- reflect the new salary back to the caller
  SELECT salary INTO p_bonus
  FROM employees
  WHERE employee_id = p_emp_id;
END add_bonus;
/ 
```

### PARALLEL_ENABLE

Lets functions run in parallel queries and DML. Use when the function is safe for parallel execution.

### RESULT_CACHE

Caches function results by parameter values in the SGA.

Example:

Cached results reused across sessions:

```sql
CREATE OR REPLACE FUNCTION get_region_name (p_region_id IN regions.region_id%TYPE)
  RETURN regions.region_name%TYPE
  RESULT_CACHE
IS
  v_name regions.region_name%TYPE;
BEGIN
  SELECT region_name INTO v_name
  FROM regions
  WHERE region_id = p_region_id;

  RETURN v_name;
END get_region_name;
/ 
```

### DETERMINISTIC

Tells Oracle that the same inputs always produce the same outputs. Useful for optimization, but do not use if the function depends on session or schema state.

---

## RETURNING Clause

Use `RETURNING` in `INSERT`, `UPDATE`, or `DELETE` to avoid a second `SELECT`. One trip to the database is enough; your app does not need to commute twice.

```sql
DECLARE
  v_new_id employees.employee_id%TYPE;
BEGIN
  INSERT INTO employees (employee_id, last_name, email, hire_date, job_id)
  VALUES (employees_seq.NEXTVAL, 'Sample', 'SAMPLE', SYSDATE, 'IT_PROG')
  RETURNING employee_id INTO v_new_id;

  DBMS_OUTPUT.PUT_LINE('New employee id = ' || v_new_id);
END;
/ 
```

---

## Bulk Binding

### FORALL

Bulk binds a collection into DML statements. It is not a loop, even though it looks suspiciously like one.

### BULK COLLECT

Fetches multiple rows into collections in a single operation.

Result:

- Fewer context switches
- Faster execution
- Less dramatic waiting

Combined example:

```sql
DECLARE
  TYPE t_emp_ids IS TABLE OF employees.employee_id%TYPE;
  l_emp_ids   t_emp_ids;
BEGIN
  SELECT employee_id
  BULK COLLECT INTO l_emp_ids
  FROM employees
  WHERE department_id = 50;

  FORALL i IN l_emp_ids.FIRST .. l_emp_ids.LAST
    UPDATE employees
    SET salary = salary * 1.05
    WHERE employee_id = l_emp_ids(i);
END;
/ 
```

---

## Wrap-Up

You should now be able to:

- Standardize constants and exception handling
- Use local subprograms for tighter scope control
- Control runtime privileges with definer vs invoker rights
- Apply autonomous transactions safely
- Optimize with `NOCOPY`, `PARALLEL_ENABLE`, `DETERMINISTIC`, and `RESULT_CACHE`
- Use `RETURNING` and bulk binding for performance gains

Next up: Practice 20, where you bulk fetch like a pro and log business operations with autonomous transactions.
