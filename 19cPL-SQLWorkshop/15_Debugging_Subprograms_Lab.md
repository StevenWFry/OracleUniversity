## Lesson 13 Lab - Debugging Subprograms

This practice walks through enabling the debugger, compiling for debug, stepping through code, and inspecting variables.

You will:

- Compile the `emp_list` procedure and `get_location` function
- Set breakpoints and debug with F7
- Inspect variables in Data/Smart Data
- Modify values during execution

---

## 1. Enable Server Output

```sql
SET SERVEROUTPUT ON
```

---

## 2. Create EMP_LIST and GET_LOCATION

From `sol_04.sql`, copy Task 2 (lines 23–75) into a worksheet.

Compile and expect an error:

- `get_location must be declared`

Then run Task 3 in `sol_04.sql` to create `get_location`, and recompile `emp_list`.

Confirm both exist under Procedures and Functions.

---

## 3. Grant Debug Privileges (if needed)

If debugging fails, grant these as PDB1 SYS:

```sql
GRANT DEBUG CONNECT SESSION TO ora61;
GRANT DEBUG ANY PROCEDURE TO ora61;
GRANT EXECUTE ON DBMS_DEBUG_JDWP TO ora61;
```

If you hit the ACL error, apply the ACL workaround from the lab instructions.

---

## 4. Add Breakpoints in EMP_LIST

Set breakpoints at:

- `OPEN cur_emp;`
- `WHILE cur_emp%FOUND LOOP`
- `v_city := get_location(...);`
- `CLOSE cur_emp;`

Compile **for Debug**.

---

## 5. Start Debugging

Click **Debug**, set `P_MAXROWS` to 100, and click OK.

If Data/Smart Data/Watches are missing:

- View > Debugger > Data / Smart Data / Watches

---

## 6. Step Through (F7)

- Step into the loop once
- Inspect `rec_emp` and `emp_tab`
- Observe values after the FETCH

Modify `i`:

- Right-click `i` > Modify Value > set to `98`

Continue stepping until the log displays employees. You should see 4 employees after that adjustment.

---

## 7. Step Over vs Step Into

- **Step Over** executes `get_location` but does not enter the function
- **Step Into** enters `get_location` line-by-line

---

## 8. Wrap-Up

You now know how to:

- Compile subprograms for debug
- Step through execution
- Inspect and modify variables
- Interpret debugger output

Practice 13 complete.