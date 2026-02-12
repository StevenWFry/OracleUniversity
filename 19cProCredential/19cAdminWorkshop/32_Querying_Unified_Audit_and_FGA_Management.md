# 32 - Querying Unified Audit, FGA, and Purge Management

This chapter covers day-2 audit operations: reading policy state, using
fine-grained auditing (FGA), and managing retention/purge without setting your
system on fire.

---

## 1. Reading Policy State

Core metadata views:

- `AUDIT_UNIFIED_POLICIES`
- enabled-policy subset views
- unified audit trail views

Yes, there is overlap. Oracle gives you multiple windows into similar data so
you can answer "what exists" and "what is active" without writing heroic SQL.

---

## 2. Top-Level Reminder (Again, Because It Matters)

If you rely only on top-level evaluation, nested/subquery activity may bypass
expected audit capture.

Translation: test real application SQL, not textbook SQL.

---

## 3. Value-Based Auditing and FGA

Value-sensitive changes often require:

- before value
- after value
- who changed it
- execution context

Historically, this meant hand-rolled trigger/procedure pain. `DBMS_FGA` gives
you structured policy control instead.

---

## 4. `DBMS_FGA.ADD_POLICY` Design Points

Typical parameters:

- target schema/object
- unique policy name
- audited columns
- condition predicate
- statement types (`SELECT`, `INSERT`, `UPDATE`, `DELETE`)
- optional handler procedure
- enabled state

Operational notes from lecture:

- policy names are globally unique
- sensitive columns used in `WHERE` still count
- `MERGE` implications are covered via `INSERT`/`UPDATE` semantics

---

## 5. Null Semantics (Powerful and Dangerous)

- condition `NULL` means broad applicability
- column list `NULL` means table-wide policy scope

Great for broad capture. Also great for generating enough audit to melt your
review process.

---

## 6. Common FGA Pitfalls

### Name drift

Policies are name-based. Rename objects/columns carelessly and policy intent can
quietly stop matching reality.

### View-only strategy mistakes

Prefer policy on base tables where possible. Views are derived queries; base
coverage is usually more reliable.

### Handler assumptions

Policy handler execution issues may not fail loudly in the way people expect.
Validate behavior end-to-end.

---

## 7. Volume Discipline

You need enough audit to detect risk, not enough audit to bury your team.

Always validate:

- storage growth
- write performance
- review feasibility

If required writes fail, audited operations may fail. This is not hypothetical.

---

## 8. Purge and Retention (`DBMS_AUDIT_MGMT`)

Use managed lifecycle controls to:

- set retention boundaries
- schedule purge jobs
- run manual cleanup when necessary

Purge actions should remain auditable themselves. Otherwise you have a
beautifully managed blind spot.

---

## 9. Key Takeaways

- know which policies exist and which are enabled.
- FGA is the right tool for value-sensitive, context-rich auditing.
- predicate usage of sensitive columns still triggers policy relevance.
- naming drift can silently break audit intent.
- retention/purge is part of security operations, not janitorial work.