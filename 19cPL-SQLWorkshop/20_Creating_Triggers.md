## Lesson 20 - Creating Triggers (in which the database gets reflexes)

Welcome to Unit 5. This lesson covers triggers: what they are, why you use them, and how not to accidentally block your own updates at 2 AM.

By the end of this lesson, you should be able to:

- Describe triggers and their uses
- Identify trigger types
- Create triggers and understand firing rules
- Enable/disable and drop triggers
- View trigger metadata

---

## 0. Where This Lesson Fits (Student Guide Lesson 18: Creating Triggers)

In the Student Guide, this lines up with **Lesson 18: Creating Triggers**. The agenda there covers:

- What triggers are and why you use them
- Trigger event types and available trigger types
- Statement vs row-level triggers, OLD/NEW and pseudorecords
- Creating DML triggers (both with `CREATE TRIGGER` and SQL Developer)
- Managing trigger status and viewing trigger information

The material below follows that same progression, and your lab is the hands-on version of **Activity Guide Practice 18**, where you build salary validation and business-hour triggers (`CHECK_SALARY_TRIG` and `DELETE_M_TRIG`) wired to reusable procedures.

---

## 1. What Is a Trigger?

A trigger is a **PL/SQL block stored in the database** that fires automatically when a specific event occurs.

Triggers can fire on:

- **DML**: INSERT, UPDATE, DELETE
- **DDL**: CREATE, ALTER, DROP
- **Database events**: LOGON, LOGOFF, STARTUP, SHUTDOWN, SERVERERROR

---

## 2. Why Use Triggers?

Common use cases:

- Audit logging
- Enforce business rules
- Prevent invalid transactions
- Maintain summary data
- Handle INSTEAD OF logic on views

If Jack wants salary changes logged automatically, triggers are the answer.

---

## 3. Trigger Types

- **Statement-level**: fires once per statement
- **Row-level**: fires once per row (`FOR EACH ROW`)
- **BEFORE / AFTER** timing
- **INSTEAD OF** (for views)

---

## 4. Example: Salary Log Trigger

```sql
CREATE TABLE salary_log (
  who_did_it  VARCHAR2(30),
  when_did_it TIMESTAMP,
  old_salary  NUMBER,
  new_salary  NUMBER,
  emp_id      NUMBER
);

CREATE OR REPLACE TRIGGER saltrig
AFTER INSERT OR UPDATE OF salary ON employees
FOR EACH ROW
BEGIN
  INSERT INTO salary_log
  VALUES (
    USER,
    SYSTIMESTAMP,
    :OLD.salary,
    :NEW.salary,
    :NEW.employee_id
  );
END;
/
```

No COMMIT inside triggers. The trigger obeys the calling transaction.

---

## 5. OLD and NEW

Row-level triggers can reference:

- `:OLD` (before change)
- `:NEW` (after change)

Only valid for row-level triggers. INSERT has no OLD, DELETE has no NEW.

---

## 6. Trigger Firing Order

- All **BEFORE STATEMENT** triggers
- For each row:
  - BEFORE ROW triggers
  - DML execution
  - constraint checking
  - AFTER ROW triggers
- All **AFTER STATEMENT** triggers

If you have multiple triggers on the same event, use `FOLLOWS` or `PRECEDES` to control order.

---

## 7. INSTEAD OF Triggers (for views)

Used for non-updatable views. The trigger performs DML on base tables instead.

---

## 8. Managing Triggers

Enable/disable:

```sql
ALTER TRIGGER trigger_name DISABLE;
ALTER TRIGGER trigger_name ENABLE;
```

Drop:

```sql
DROP TRIGGER trigger_name;
```

View metadata:

```sql
SELECT * FROM user_triggers;
```

Errors:

```sql
SELECT * FROM user_errors;
```

---

## 9. Wrap-Up

You now know:

- How triggers work and when they fire
- How to create row- and statement-level triggers
- How to manage and test triggers

Next up: **Practice 18**, where you build row and statement triggers and call procedures from triggers.
