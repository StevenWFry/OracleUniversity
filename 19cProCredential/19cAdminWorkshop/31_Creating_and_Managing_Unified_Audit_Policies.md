# 31 - Creating and Managing Unified Audit Policies

This chapter moves from theory to operation: how to create policies, control
scope, enable/disable safely, and avoid accidentally building a very expensive
noise machine.

---

## 1. Do the Setup in Order

Before policy design:

1. verify unified auditing is enabled in binaries
2. enable/relink if needed and bounce database/home
3. configure protected audit destination/storage
4. review predefined mandatory policies first

Why start with mandatory policies?

- they already generate volume
- they already consume space
- they already contribute overhead

You design custom policy on top of that baseline, not in a fantasy vacuum.

---

## 2. Capacity Is a Functional Requirement

If required audit records cannot be written, audited operations can fail.

So audit capacity planning is not just compliance paperwork. It is availability
engineering.

Plan for:

- retention window
- growth rate
- purge schedule
- failure behavior

---

## 3. Policy Definition Basics

Pattern:

```sql
CREATE AUDIT POLICY policy_name
  ACTIONS ...
  [WHEN ...]
  [EVALUATE PER { STATEMENT | SESSION | INSTANCE }];
```

Targets can include combinations of:

- actions
- privileges
- roles
- objects

Keep policy count intentional. Thousands of micro-policies might look precise
and then become unmaintainable very quickly.

---

## 4. Conditions Are Where Quality Lives

Good auditing is conditional.

Examples from lecture:

- by specific user / excluding temporary exceptions
- success-only or failure-only
- contextual gating for sensitive operations

Failure-focused policies are especially useful for detecting brute-force and
abuse patterns.

---

## 5. Enable, Then Actually Verify

Creating a policy does nothing by itself.

Enable it:

```sql
AUDIT POLICY policy_name;
```

You can also enable with user filters and success/failure qualifiers for
controlled windows of observation.

---

## 6. Disable Before Drop

Disable first:

```sql
NOAUDIT POLICY policy_name;
```

Then drop only after confirming it is no longer needed. Trying to treat active
policies like disposable scratch objects is how audit governance turns into
mystery theater.

---

## 7. Altering Policies: Handle with Care

`ALTER AUDIT POLICY` can adjust scope and behavior, but condition changes are
sensitive.

Common mistakes:

- overbroad conditions that flood trail volume
- top-level-only assumptions that miss nested SQL behavior
- thinking "column not in SELECT" means no sensitive inference is possible

It is very possible to leak insight without selecting the sensitive column
explicitly.

---

## 8. CDB vs PDB Scope Reality

In multitenant:

- unified auditing is enabled at CDB binary level
- most business-data audit policies are created in PDBs
- object audit intent does not magically cross container boundaries

Build policy where the target objects live.

---

## 9. Key Takeaways

- mandatory policy baseline comes first.
- custom policy quality depends on conditions, not volume.
- enable/disable lifecycle discipline is non-negotiable.
- top-level-only policy choices need explicit validation.
- audit design is both security and availability design.