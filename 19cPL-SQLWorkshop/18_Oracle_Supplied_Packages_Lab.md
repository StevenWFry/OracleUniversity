## Lesson 16 Lab - Oracle-Supplied Packages (UTL_FILE Report)

This practice uses UTL_FILE to generate a report of employees whose salaries exceed the average for their department.

Before starting:

- Run `cleanup_16.sql`

---

## 1. Create the Directory Object

```sql
CREATE OR REPLACE DIRECTORY reports_dir AS '/home/oracle/labs/plpu/reports';
```

Refresh the **Directories** node and confirm `REPORTS_DIR` appears.

---

## 2. Create EMPLOYEE_REPORT Procedure

```sql
CREATE OR REPLACE PROCEDURE employee_report (
  p_dir      IN VARCHAR2,
  p_filename IN VARCHAR2
) IS
  f      UTL_FILE.FILE_TYPE;
  CURSOR cur_avg IS
    SELECT last_name, department_id, salary
    FROM employees outer
    WHERE salary > (
      SELECT AVG(salary)
      FROM employees
      WHERE department_id = outer.department_id
    )
    ORDER BY department_id;
BEGIN
  f := UTL_FILE.FOPEN(p_dir, p_filename, 'W');

  UTL_FILE.PUT_LINE(f, 'Report generated on ' || TO_CHAR(SYSDATE, 'DD-MON-YY'));
  UTL_FILE.NEW_LINE(f);

  FOR r IN cur_avg LOOP
    UTL_FILE.PUT_LINE(
      f,
      RPAD(r.last_name, 30) || ' ' ||
      LPAD(NVL(r.department_id, 9999), 5, ' ') || ' - ' ||
      LPAD(r.salary, 8, ' ')
    );
  END LOOP;

  UTL_FILE.NEW_LINE(f);
  UTL_FILE.PUT_LINE(f, 'End of report');

  UTL_FILE.FCLOSE(f);
EXCEPTION
  WHEN UTL_FILE.INVALID_FILEHANDLE THEN
    DBMS_OUTPUT.PUT_LINE('Invalid file handle');
  WHEN UTL_FILE.INVALID_PATH THEN
    DBMS_OUTPUT.PUT_LINE('Invalid path');
  WHEN UTL_FILE.WRITE_ERROR THEN
    DBMS_OUTPUT.PUT_LINE('Write error');
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('Unexpected error');
END employee_report;
/
```

---

## 3. Execute the Procedure

```sql
EXECUTE employee_report('REPORTS_DIR', 'sal_rpt71.txt')
```

---

## 4. Verify Output

Check the OS directory:

- `/home/oracle/labs/plpu/reports/sal_rpt71.txt`

The report should list employees whose salary exceeds their department average.

---

## Wrap-Up

You generated a text report using UTL_FILE and confirmed it in the filesystem.