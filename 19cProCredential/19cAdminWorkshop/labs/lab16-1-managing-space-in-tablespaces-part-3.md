# Lab 16-1 - Managing Space in Tablespaces (Part 3)

Part 3 completes Practice 3-1 by validating critical-level alerts, running
Segment Advisor, shrinking recommended segments, and confirming reclaimed space.

---

## 1. Confirm Alert at ~75% Full

Recheck outstanding alert:

```sql
SELECT reason
FROM   dba_outstanding_alerts
WHERE  object_name = 'TBSALERT';
```

Expected:

- reason reflects `TBSALERT` around `75%` full.

---

## 2. Delete Rows and Observe No Immediate Tablespace Reclaim

Delete rows from the three HR tables as directed by the guide (employees1/2/3
in this practice flow), then commit.

Recheck fullness with your existing size query/script.

Expected:

- still about `75%` full

Why:

- deleting rows frees row/block space for reuse inside segments
- it does not automatically return extents to tablespace free pool

---

## 3. Run Segment Advisor Task

Execute advisor setup script:

```sql
@/home/oracle/labs/DBMod_Storage/segmentadvisortest.sql
```

Run the advisor task as instructed in script output.

Query recommendations:

```sql
SELECT task_name, recommendation
FROM   dba_advisor_tasks
WHERE  task_name LIKE '%SEGMENT%';
```

Expected guidance:

- shrink advice for segments in `TBSALERT`.

---

## 4. Identify Segments to Shrink

Run:

```sql
@/home/oracle/labs/DBMod_Storage/segmentstoshrink.sql
```

Expected:

- first three segments are recommended for shrink (per transcript output).

---

## 5. Shrink Recommended Tables

Perform shrink for:

- `HR.EMPLOYEES1`
- `HR.EMPLOYEES2`
- `HR.EMPLOYEES3`

Use the script/commands provided by the guide for these three tables.

---

## 6. Verify Reclaimed Space

Rerun fullness query/script for `TBSALERT`.

Expected:

- fullness drops from about `75%` to about `30%`.

This confirms shrink released unused segment space back to the tablespace.

---

## 7. Cleanup and Exit

Drop test tablespace and exit SQL*Plus:

```sql
DROP TABLESPACE tbsalert INCLUDING CONTENTS AND DATAFILES;
EXIT
```

---

## 8. Lab Result

You validated warning/critical alert behavior, confirmed that deletes alone do
not reclaim tablespace usage, used Segment Advisor recommendations, shrank
target segments, and reduced `TBSALERT` usage significantly before cleanup.
