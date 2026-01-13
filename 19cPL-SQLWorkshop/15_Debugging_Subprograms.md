## Lesson 15 - Debugging Subprograms (in which the debugger becomes your therapist)

Welcome to Lesson 13. This is where we stop guessing and start watching variables live, step by step. The SQL Developer debugger is the flashlight you use to explore the caves of your procedure.

By the end of this lesson, you should be able to:

- Describe SQL Developer debugger functionality
- Debug subprograms
- Use debugger features to inspect and modify values

---

## 1. Required Privileges (because debugging is gated)

To debug PL/SQL, you need:

- `DEBUG CONNECT SESSION`
- `DEBUG ANY PROCEDURE`
- `EXECUTE` on `DBMS_DEBUG_JDWP`
- `EXECUTE` on the object you want to debug

Check privileges:

```sql
SELECT *
FROM user_sys_privs;
```

Grant (as SYSDBA):

```sql
GRANT DEBUG CONNECT SESSION TO ora61;
GRANT DEBUG ANY PROCEDURE TO ora61;
GRANT EXECUTE ON DBMS_DEBUG_JDWP TO ora61;
```

If you hit a **network access denied by ACL** error, you must grant the ACL access using the provided script (as SYSDBA).

---

## 2. Debugging Workflow

1) Add breakpoints
2) Compile **for Debug**
3) Click **Debug**
4) Supply input values (if prompted)
5) Step through code

If you see an ACL error, fix it first or the debugger cannot connect.

---

## 3. Debugger Toolbar (your control panel)

- **Terminate**: stop the session
- **Find Execution Point**: jump to the current line
- **Step Over**: skip into called subprograms
- **Step Into**: enter called subprograms
- **Step Out**: return to caller
- **Step to End**: jump to end of current method
- **Resume**: continue execution
- **Pause**: halt without exiting
- **Suspend Breakpoints**: disable all breakpoints

---

## 4. Debugger Windows

- **Breakpoints**: shows active breakpoints
- **Data**: shows current variable values
- **Smart Data**: filtered variable view
- **Watches**: custom watch list
- **Log**: debug messages

Use Watches when you only care about a few variables, not all 100.

---

## 5. Step Over vs Step Into

- **Step Over**: executes the subprogram call, then lands on the next line
- **Step Into**: jumps into the called subprogram and steps through it line by line

Use Step Into when you suspect the bug is inside the called function. Otherwise, Step Over keeps you sane.

---

## 6. Modify Values While Debugging

You can right-click a variable in the Data window and **Modify Value**. Useful when you want to test alternate paths without rerunning the entire procedure.

---

## 7. Remote Debugging (advanced mode)

To debug remotely:

- Compile for Debug
- Choose **Remote Debug**
- Enter host IP and port
- Run the procedure in another session

Once a breakpoint hits, control transfers to SQL Developer.

---

## 8. Wrap-Up

You now know how to:

- Enable debug privileges
- Step through PL/SQL
- Inspect and modify variables
- Use Watches for targeted tracking
- Debug locally or remotely

Next up: **Practice 13**, where you build a procedure and function, set breakpoints, and step through execution in the debugger.