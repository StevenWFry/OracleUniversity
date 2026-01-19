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

Example: a standardized error package with named exceptions bound to error codes via `PRAGMA EXCEPTION_INIT`. You define the exception once and reuse it everywhere.

---

## Local Subprograms

You can define a function or procedure inside another subprogram. That local subprogram is only visible inside the parent block.

Example: `employee_sal` declares a local `tax` function and uses it in the executable section. Local subprograms keep helper logic close and private, like a tiny, well-behaved gremlin.

---

## Definer’s Rights vs Invoker’s Rights

By default, subprograms run with **definer’s rights**.

If you add:

```
AUTHID CURRENT_USER
```

you get **invoker’s rights**. That means:

- Object names resolve in the invoker’s schema
- Privileges come from the user calling the program

Use this when you want reusable code across multiple schemas without forcing everything into one owner’s privileges.

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

If the main transaction fails, the audit log still survives. It is basically your database version of "pics or it didn’t happen."

---

## Performance Directives

### NOCOPY

Pass `OUT` and `IN OUT` parameters by reference instead of by value.

Benefits:

- Less memory overhead
- Faster parameter passing

Downside: if the subprogram fails, you cannot rely on partial values.

### PARALLEL_ENABLE

Lets functions run in parallel queries and DML. Use when the function is safe for parallel execution.

### RESULT_CACHE

Caches function results by parameter values in the SGA.

Example:

- Cached results reused across sessions
- Use `RELIES_ON` for cache invalidation when data changes

### DETERMINISTIC

Tells Oracle that the same inputs always produce the same outputs. Useful for optimization, but do not use if the function depends on session or schema state.

---

## RETURNING Clause

Use `RETURNING` in `INSERT`, `UPDATE`, or `DELETE` to avoid a second `SELECT`. One trip to the database is enough; your app does not need to commute twice.

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
