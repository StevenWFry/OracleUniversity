# 24 - Enabling Temporary Undo and Undo Operations (Because "Who Changed This?" Is a Daily Event)

This lesson continues undo management with hands-on focus: enabling temp undo,
adjusting retention, reviewing undo in EM Express, and querying historical row
versions after commit.

---

## 1. Enabling Temporary Undo

`TEMP_UNDO_ENABLED` can be turned on at database level:

```sql
ALTER SYSTEM SET temp_undo_enabled = TRUE;
```

It is dynamic, so changes can apply immediately (and persist when configured
accordingly).

Session-level override is also possible:

```sql
ALTER SESSION SET temp_undo_enabled = TRUE;
```

Lecture recommendation:

- prefer database-level enablement in environments that rely on temp-table-driven reporting workflows.

Monitor behavior with:

- `V$TEMPUNDOSTAT` (and related runtime views) for temp undo generation trends.

---

## 2. Undo in EM Express: What to Review

From EM Express storage pages, you can inspect:

- undo mode (`AUTO`)
- current undo tablespace
- current undo retention
- usage statistics and free-space trends
- related redo group context from storage panels

For retention changes via parameter UI:

- inspect SQL before applying (`Show SQL`)
- confirm both memory and spfile scope behavior

Example shown:

- raising `UNDO_RETENTION` from default window to `3600` seconds (1 hour)

Because "just increase it" without checking SQL is how config drift gets promoted to policy.

---

## 3. Undo Retention Tuning Pattern

Practical sequence shown:

1. identify long-running report/job duration
2. set retention above that runtime window
3. validate undo space can actually support retention
4. monitor for `snapshot too old` recurrence

If retention grows but undo size does not, policy is aspirational, not real.

---

## 4. Flashback-Style Querying of Old Values

Demo setup:

- table `EMP` with rows
- perform updates + commits on a target row
- query historical versions using `VERSIONS BETWEEN`

Example pattern:

```sql
SELECT id, sal
FROM   emp
VERSIONS BETWEEN SCN MINVALUE AND MAXVALUE
WHERE  id = 2;
```

Result interpretation:

- newest/top row from current table state
- prior versions from undo history

You can also use:

- timestamp/SCN columns for time-anchored change tracking
- `AS OF` queries for point-in-time value recovery

---

## 5. Row Movement Requirement for User-Level Flashback Use Cases

For table-level flashback-style operations, row movement must be enabled:

```sql
ALTER TABLE emp ENABLE ROW MOVEMENT;
```

If already enabled, Oracle returns corresponding status/error behavior.

Disable when not needed:

```sql
ALTER TABLE emp DISABLE ROW MOVEMENT;
```

Operational approach:

- enable around major change windows where rollback/flashback visibility is needed
- disable afterward if your policy minimizes any extra overhead footprint

---

## 6. Example Change Timeline from Demo

Observed change sequence:

1. original `SAL` value (for `ID=2`) was `100`
2. updated/committed to `1`
3. updated/committed to `2`

Version query returned all states, allowing reconstruction of prior value
without relying on memory, spreadsheets, or panic.

---

## 7. Key Takeaways

- enable temp undo where reporting/temp-table workflows justify it.
- tune undo retention with real workload durations, not round numbers.
- always validate generated SQL in GUI tools before apply.
- use version/as-of queries to retrieve old values after commit.
- row movement is part of the flashback operational toolkit.
