## Lesson 8 Lab - Using Explicit Cursors (in which you herd rows like cats)

Welcome to Practice 8. You will use explicit cursors and cursor FOR loops to process employees by department, then write a nested cursor block using parameters and cursor attributes.

You will:

- Use a cursor FOR loop to flag raise eligibility
- Use two explicit cursors (one parameterized) to print department and employee info

---

## 1. Cursor FOR Loop: Raise Eligibility

Create a block that checks employees in a department and prints whether they are due for a raise.

Rules:

- If salary < 5000 AND manager_id is 101 or 124: **due for a raise**
- Otherwise: **not due for a raise**

```sql
DECLARE
  v_deptno NUMBER := 10;

  CURSOR c_emp_cursor IS
    SELECT last_name, salary, manager_id
    FROM employees
    WHERE department_id = v_deptno;
BEGIN
  FOR emp_record IN c_emp_cursor LOOP
    IF emp_record.salary < 5000 AND
       (emp_record.manager_id = 101 OR emp_record.manager_id = 124) THEN
      DBMS_OUTPUT.PUT_LINE(emp_record.last_name || ' due for a raise');
    ELSE
      DBMS_OUTPUT.PUT_LINE(emp_record.last_name || ' not due for a raise');
    END IF;
  END LOOP;
END;
/
```

Test with departments:

- 10
- 20
- 50
- 80

Expected examples:

- Dept 10: Whalen due for a raise
- Dept 20: Hartstein not due for a raise, Fay not due for a raise
- Dept 50: Weiss not due for a raise ... Grant due for a raise

---

## 2. Two Explicit Cursors (one with a parameter)

Cursor 1: departments with ID < 100

Cursor 2: employees in each department with employee_id < 120

```sql
DECLARE
  CURSOR c_dept_cursor IS
    SELECT department_id, department_name
    FROM departments
    WHERE department_id < 100
    ORDER BY department_id;

  CURSOR c_emp_cursor (v_deptno NUMBER) IS
    SELECT last_name, job_id, hire_date, salary
    FROM employees
    WHERE department_id = v_deptno
      AND employee_id < 120;

  v_current_deptno departments.department_id%TYPE;
  v_current_dname departments.department_name%TYPE;
  v_ename         employees.last_name%TYPE;
  v_job           employees.job_id%TYPE;
  v_hiredate      employees.hire_date%TYPE;
  v_sal           employees.salary%TYPE;
BEGIN
  OPEN c_dept_cursor;
  LOOP
    FETCH c_dept_cursor INTO v_current_deptno, v_current_dname;
    EXIT WHEN c_dept_cursor%NOTFOUND;

    DBMS_OUTPUT.PUT_LINE('Department number: ' || v_current_deptno);
    DBMS_OUTPUT.PUT_LINE('Department name: ' || v_current_dname);

    IF c_emp_cursor%ISOPEN THEN
      CLOSE c_emp_cursor;
    END IF;

    OPEN c_emp_cursor(v_current_deptno);
    LOOP
      FETCH c_emp_cursor INTO v_ename, v_job, v_hiredate, v_sal;
      EXIT WHEN c_emp_cursor%NOTFOUND;

      DBMS_OUTPUT.PUT_LINE(v_ename || ' ' || v_job || ' ' ||
                           v_hiredate || ' ' || v_sal);
    END LOOP;

    DBMS_OUTPUT.PUT_LINE('-----------------------------');
    CLOSE c_emp_cursor;
  END LOOP;

  CLOSE c_dept_cursor;
END;
/
```

Output should list each department (<100), followed by employees in that department, then a separator line.

---

## 3. Wrap-Up

You have now:

- Used a cursor FOR loop with conditional logic
- Managed two explicit cursors, one parameterized
- Used cursor attributes for exit and safety checks

That concludes Practice 8.