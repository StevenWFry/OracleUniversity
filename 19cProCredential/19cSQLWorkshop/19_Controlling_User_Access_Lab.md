# Lesson 18/19 Lab – Controlling User Access (Or: How Not To Give Everyone `DBA`)

And look, nothing ruins a database faster than handing out `GRANT ALL ON EVERYTHING TO EVERYONE` like it’s Halloween candy. This lab is where you practice doing it properly: precise system and object privileges, roles, synonyms, and the joy of discovering you can’t revoke what you didn’t grant.

You’ll:

- Grant and revoke object privileges (with and without `WITH GRANT OPTION`)  
- Let one user modify another user’s tables (on purpose!)  
- Use synonyms to hide schema names and pretend everything is local  

Environment: two SQL Developer sessions – one as `ora21`, one as `ora22` (and briefly a third user `ora23`).

---

## Task 0 – Concept warm‑up

From the practice questions:

- To **log on** to the database, a user needs the `CREATE SESSION` system privilege.  
- To **create tables**, they need the `CREATE TABLE` system privilege.  
- If *you* create a table:
  - You can grant object privileges on that table.  
  - Anyone you granted privileges to **with `WITH GRANT OPTION`** can also pass them on.  
- If you’ve got many users needing the same system privileges, create a **role**, grant privileges to the role, then grant the role to users.  
- To change your own password: `ALTER USER <username> IDENTIFIED BY <new_password>;`

Later questions drive home:

- Revoke from the user who had `WITH GRANT OPTION`, and the privileges vanish from downstream users too.  
- `GRANT UPDATE ON departments TO scott WITH GRANT OPTION;` lets Scott update and also grant that privilege on.

With that in your head, on to the hands‑on bits.

---

## Task 1 – Grant `SELECT` on `REGIONS` with `WITH GRANT OPTION`

Session 1: connect as `ora21`.  
Session 2: connect as `ora22`.

**Step A – `ora21` grants `SELECT` on `REGIONS` to `ora22` with grant option**

```sql
GRANT SELECT ON regions TO ora22 WITH GRANT OPTION;
```

**Step B – `ora22` queries `ora21`’s `REGIONS`**

In the `ora22` session:

```sql
SELECT * 
FROM   ora21.regions;
```

If that works, the privilege is active.

**Step C – `ora22` passes the privilege on to `ora23`**

Assuming user `ora23` exists:

```sql
GRANT SELECT ON ora21.regions TO ora23;
```

Thanks to `WITH GRANT OPTION`, `ora22` is now a tiny privilege distributor.

---

## Task 2 – See what happens when you revoke from the grantor

Back as `ora21`, revoke the privilege from `ora22`:

```sql
REVOKE SELECT ON regions FROM ora22;
```

Effects:

- `ora22` loses `SELECT` on `ora21.regions`.  
- **All privileges `ora22` granted on that object (to `ora23`) are revoked as well.**

End result: only `ora21` retains full control, which is exactly how revocation should behave.

---

## Task 3 – Grant query and DML privileges (no grant option)

Now give `ora22` meaningful access without the ability to pass it on.

From `ora21`:

```sql
GRANT SELECT, INSERT, UPDATE
ON    countries
TO    ora22;
```

Check that `ora22` can work with `ora21.countries` but **cannot** grant those privileges further:

```sql
-- Works
SELECT * 
FROM   ora21.countries;

-- Fails
GRANT SELECT ON ora21.countries TO ora23;
```

Revoke them when done:

```sql
REVOKE SELECT, INSERT, UPDATE
ON    countries
FROM  ora22;
```

---

## Task 4 – Share `DEPARTMENTS` both ways

This is the “data sharing, but with boundaries” part.

**Step A – `ora22` grants `SELECT` on its `DEPARTMENTS` to `ora21`**

From `ora22`:

```sql
GRANT SELECT ON departments TO ora21;
```

**Step B – `ora21` grants `SELECT` on its `DEPARTMENTS` to `ora22` (no grant option)**

From `ora21`:

```sql
GRANT SELECT ON departments TO ora22;
```

**Step C – Check cross‑schema access**

From `ora21`:

```sql
SELECT * 
FROM   ora22.departments;
```

From `ora22`:

```sql
SELECT * 
FROM   ora21.departments;
```

You should see each other’s department data, including the last existing row (note the highest `DEPARTMENT_ID` – you’ll add to it next).

---

## Task 5 – Each user inserts a department in their own table

**`ora21`** inserts department 500 (Education):

```sql
INSERT INTO departments (department_id, department_name, manager_id, location_id)
VALUES (500, 'Education', NULL, 1700);

COMMIT;
```

**`ora22`** inserts department 510 (Human Resources):

```sql
INSERT INTO departments (department_id, department_name, manager_id, location_id)
VALUES (510, 'Human Resources', NULL, 1700);

COMMIT;
```

Verify cross‑schema:

From `ora21`:

```sql
SELECT * 
FROM   ora22.departments
WHERE  department_id = 510;
```

From `ora22`:

```sql
SELECT * 
FROM   ora21.departments
WHERE  department_id = 500;
```

Everyone can see everyone else’s new rows. The HR data mesh is alive.

---

## Task 6 – Hide schema names with synonyms

Because typing `ORA22.DEPARTMENTS` all day is how wrists die.

**Step A – `ora21` creates a synonym for `ora22`’s `DEPARTMENTS`**

```sql
CREATE SYNONYM user2 FOR ora22.departments;
```

**Step B – `ora22` creates a synonym for `ora21`’s `DEPARTMENTS`**

```sql
CREATE SYNONYM user1 FOR ora21.departments;
```

Now query via synonyms:

From `ora21`:

```sql
SELECT * 
FROM   user2;
```

From `ora22`:

```sql
SELECT * 
FROM   user1;
```

Same data, fewer keystrokes, slightly more indirection.

---

## Task 7 – Revoke `SELECT` and clean up data and synonyms

**Step A – `ora22` revokes `SELECT` from `ora21`**

From `ora22`:

```sql
REVOKE SELECT ON departments FROM ora21;
```

**Step B – `ora21` revokes `SELECT` from `ora22`**

From `ora21`:

```sql
REVOKE SELECT ON departments FROM ora22;
```

At this point, cross‑schema direct access should fail.

**Step C – Remove the rows you added**

From `ora21`:

```sql
DELETE FROM departments
WHERE  department_id = 500;

COMMIT;
```

From `ora22`:

```sql
DELETE FROM departments
WHERE  department_id = 510;

COMMIT;
```

**Step D – Drop the synonyms**

From `ora21`:

```sql
DROP SYNONYM user2;
```

From `ora22`:

```sql
DROP SYNONYM user1;
```

And just like that, all the sharing and the shortcuts are gone, and your schemas are back to minding their own business.

---

## What You’ve Practised

In this lab you’ve:

- Granted object privileges with and without `WITH GRANT OPTION` and seen how revokes cascade  
- Allowed one user to query and modify another user’s tables (intentionally)  
- Used synonyms to hide schema prefixes and simplify cross‑schema queries  

Which means you’re now capable of controlling who can see and change what—without either:

- Locking everyone out, or  
- Accidentally giving the entire office full access to production.

In security terms, that’s called “doing your job”, but in database terms it’s basically wizardry.  

