## Lesson 19 - Using Dynamic SQL (in which SQL becomes improvisational)

Welcome to Lesson 17, the final stop in Unit 4. This is where we build SQL statements at runtime, because sometimes you just do not know the table name until the user types it.

By the end of this lesson, you should be able to:

- Describe the execution flow of SQL
- Build and execute dynamic SQL with Native Dynamic SQL (NDS)
- Know when to use DBMS_SQL instead of NDS

---

## 1. What Is Dynamic SQL?

Dynamic SQL is SQL whose full text is unknown until runtime. It is useful when:

- The SQL text is not known at compile time
- The number or types of variables are unknown
- Object names (tables, columns) are supplied at runtime

If the SQL is dynamic, parsing and binding happen at runtime.

---

## 2. Example: Dynamic DELETE

This fails in static SQL because the table name is unknown at compile time:

```sql
DELETE FROM p_table;
```

Correct version using NDS:

```sql
EXECUTE IMMEDIATE 'DELETE FROM ' || p_table;
```

Now the table name is resolved at runtime.

---

## 3. NDS Basics (EXECUTE IMMEDIATE)

Syntax:

```sql
EXECUTE IMMEDIATE sql_string
  [INTO ...]
  [USING ...];
```

- `INTO` is for single-row SELECTs
- `USING` supplies bind variables

---

## 4. Examples

Create table dynamically:

```sql
EXECUTE IMMEDIATE 'CREATE TABLE ' || p_table || ' (' || p_cols || ')';
```

Insert with bind placeholders:

```sql
EXECUTE IMMEDIATE
  'INSERT INTO ' || p_table || ' VALUES (:1, :2)'
USING p_id, p_name;
```

Single-row SELECT:

```sql
EXECUTE IMMEDIATE
  'SELECT last_name FROM employees WHERE employee_id = :1'
INTO v_lname
USING p_empid;
```

---

## 5. Multi-Row Results

For multiple rows, use:

- `BULK COLLECT INTO`
- `OPEN FOR` + `FETCH`

Example:

```sql
OPEN emp_cv FOR 'SELECT employee_id, last_name FROM employees';
FETCH emp_cv BULK COLLECT INTO v_ids, v_names;
CLOSE emp_cv;
```

---

## 6. Four Methods of Dynamic SQL

1) **Method 1** - Non-query, no binds
   - Use `EXECUTE IMMEDIATE` only

2) **Method 2** - Non-query with binds
   - `EXECUTE IMMEDIATE ... USING`

3) **Method 3** - Query returning known columns
   - `EXECUTE IMMEDIATE ... INTO ... USING`

4) **Method 4** - Query with unknown columns/types
   - Use `DBMS_SQL`

---

## 7. DBMS_SQL (when you do not know the shape)

Use `DBMS_SQL` when:

- You don’t know the select list at compile time
- You don’t know how many columns will be returned
- You don’t know their data types

DBMS_SQL has more steps:

- `OPEN_CURSOR`
- `PARSE`
- `BIND_VARIABLE`
- `EXECUTE`
- `FETCH_ROWS`
- `CLOSE_CURSOR`

It is more verbose but necessary for Method 4.

---

## 8. Wrap-Up

You now know:

- How dynamic SQL works at runtime
- How to use `EXECUTE IMMEDIATE`
- When to switch to `DBMS_SQL`

Next up: **Practice 17**, where you build a package that uses dynamic SQL to create, modify, and drop objects.