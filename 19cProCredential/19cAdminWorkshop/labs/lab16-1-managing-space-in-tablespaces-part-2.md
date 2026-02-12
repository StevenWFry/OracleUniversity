# Lab 16-1 - Managing Space in Tablespaces (Part 2)

Part 2 continues after warning-level alert verification and pushes `TBSALERT`
toward critical threshold behavior.

---

## 1. Starting State

You should already have:

- `TBSALERT` warning alert visible in `DBA_OUTSTANDING_ALERTS`
- reason text indicating about `60%` full

---

## 2. Add More Data to Increase Tablespace Usage

In SQL*Plus (`system@orclpdb1`), run additional inserts and commits per guide.

Transcript flow used:

```sql
INSERT INTO hr.employees4
SELECT *
FROM   hr.employees4;
COMMIT;
```

Then similarly for the next table:

```sql
INSERT INTO hr.employees5
SELECT *
FROM   hr.employees5;
COMMIT;
```

Purpose:

- push `TBSALERT` from warning-level usage (~60%) toward critical-level usage.

---

## 3. Recheck Fullness of `TBSALERT`

Rerun the same size/fullness query/script used in Part 1.

Expected target in this section:

- about `75%` full

---

## 4. Recheck Outstanding Alerts

Query outstanding alerts again:

```sql
SELECT reason
FROM   dba_outstanding_alerts
WHERE  object_name = 'TBSALERT';
```

Expected update:

- reason reflects tablespace around `75%` full

Timing note:

- alert updates can be delayed
- retry for up to ~10 minutes if needed

---

## 5. Part 2 Stopping Point

Part 2 ends at the second wait/retry stage while alert output catches up to the
new `75%` fullness level.
