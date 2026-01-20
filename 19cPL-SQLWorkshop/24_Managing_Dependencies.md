## Lesson 22 - Managing Dependencies

OK, my friends, welcome to the final lesson. We are managing dependencies, which is a fancy way of saying, "What breaks when I touch this?"

After completing this lesson, you should be able to:

- Track procedural dependencies
- Predict the effect of changing a database object on procedures and functions
- Manage procedural dependencies
- Manage local and remote dependencies

---

## 0. Where This Lesson Fits (Student Guide Lesson 22: managing dependencies)

In the Student Guide, this is **Lesson 22: Managing Dependencies in Your Schema**. The agenda covers:

- Direct vs indirect dependencies and how to display them
- Fine-grained dependency management (less unnecessary invalidation)
- Guidelines for reducing invalidations and object revalidation
- Managing remote dependencies (`REMOTE_DEPENDENCIES_MODE`, timestamp vs signature)
- Recompiling PL/SQL units and understanding how invalid/valid states change

The sections that follow mirror that structure, and your lab is the implementation of **Activity Guide Practice 22**, where you use `DEPTREE_FILL`/`IDEPTREE` to explore dependencies and then extend `COMPILE_PKG` so you can recompile invalid objects with dynamic SQL.

---

## Schema Object Dependencies

A dependency exists when one object relies on another for its definition. Examples:

- Views depend on tables
- Procedures depend on tables or views

Direct dependencies:

- Object A directly references Object B

Indirect dependencies:

- Object A depends on Object B, which depends on Object C

Changing Object C can invalidate Object A, even if A never mentioned C directly.

Tiny demo:

```sql
CREATE OR REPLACE VIEW emp_names_v AS
  SELECT employee_id, last_name
  FROM employees;

-- EMP_NAMES_V now depends on EMPLOYEES
```

---

## Tracking Dependencies

Use:

- SQL Developer `Dependencies` tab
- `USER_DEPENDENCIES`

Direct dependencies are level 1. Indirect dependencies are level 2+.

For a full tree (including indirect):

- Run `utldtree.sql`
- Use `DEPTREE_FILL`
- Query `DEPTREE` or `IDEPTREE`

If you like visual trees, `IDEPTREE` is your friend.

Quick example of querying dependencies for a view:

```sql
SELECT referenced_owner,
       referenced_name,
       referenced_type
FROM   user_dependencies
WHERE  name = 'EMP_NAMES_V';
```

---

## Object Status Values

Objects can be:

- `VALID`
- `INVALID`
- `COMPILED WITH ERRORS`
- `UNAUTHORIZED`

Only dependent objects become INVALID or UNAUTHORIZED. The referenced object is innocent. It just changed its hair.

You can check status in `USER_OBJECTS`:

```sql
SELECT object_name, object_type, status
FROM   user_objects
WHERE  object_name IN ('EMPLOYEES', 'EMP_NAMES_V');
```

---

## Fine-Grained Dependency Management

Since 11g, Oracle tracks dependencies at the element level.

Result:

- Adding a new column does not invalidate a view that never referenced it
- Modifying a referenced column does invalidate dependent objects

Less invalidation. More sanity.

---

## Local vs Remote Dependencies

Local:

- Managed automatically by Oracle

Remote:

- Managed with `REMOTE_DEPENDENCIES_MODE`

Modes:

- `TIMESTAMP` (default)
- `SIGNATURE`

Timestamp checks invalidation if the remote object was recompiled.
Signature checks only invalidate if the interface changed.

Example of changing the mode at the session level before calling remote code:

```sql
ALTER SESSION SET remote_dependencies_mode = signature;

-- call a remote procedure over a database link
BEGIN
  remote_pkg.do_something@hr_link;
END;
/ 
```

---

## Recompiling Objects

Objects revalidate automatically when referenced, unless:

- They have compilation errors
- They are unauthorized
- Remote dependencies prevent validation

You can also force it:

```sql
ALTER PROCEDURE my_proc COMPILE;
ALTER PACKAGE my_pkg COMPILE;
```

To recompile everything invalid in your schema:

```sql
BEGIN
  FOR r IN (
    SELECT object_name, object_type
    FROM   user_objects
    WHERE  status = 'INVALID'
  ) LOOP
    EXECUTE IMMEDIATE
      'ALTER ' || r.object_type || ' ' || r.object_name || ' COMPILE';
  END LOOP;
END;
/ 
```

---

## Reducing Invalidation

Best practices:

- Use `%TYPE` and `%ROWTYPE`
- Reference tables through views
- Add new package elements at the end
- Avoid changing signatures unless necessary

Think of dependencies as a delicate house of cards. Touch gently.

---

## Wrap-Up

You should now be able to:

- Track direct and indirect dependencies
- Predict invalidation after object changes
- Use dependency tools (`USER_DEPENDENCIES`, `DEPTREE`, `IDEPTREE`)
- Manage local and remote dependency behavior

This is the final lesson. Congratulations. Go celebrate. Then back up your schema.
