# 39 - Database Monitoring and Tuning Overview

Welcome to the module where "it feels slower" turns into measurable evidence,
because performance arguments without baselines are just organized guessing.

---

## 1. First Principle: Performance Is Perception

The chapter starts with the uncomfortable truth:

- performance is not an absolute number
- performance is user perception against expected normal behavior

If behavior is consistently the same, users usually accept it.
If behavior drifts from normal, users call it "slow" even if the database still
looks "busy but alive."

So your first job is to define normal.

---

## 2. Baselines: Your Definition of "Normal"

A baseline is a stored set of performance statistics representing healthy,
expected operation.

Tuning workflow:

1. establish baseline
2. detect deviation period
3. compare deviation vs baseline
4. identify differences
5. test fixes
6. apply proven fixes in production
7. continue monitoring

No baseline, no objective comparison. No objective comparison, no reliable root
cause.

---

## 3. Why Statistics Drive Everything

Oracle optimizer is cost-based. Plan quality depends on metadata quality.

It needs accurate object statistics, including:

- row counts
- data distribution
- repeat/unique value patterns
- object structure and access path cost signals

Bad stats = bad plan choices = unhappy humans.

---

## 4. Automated Maintenance Task Role

Automated maintenance tasks keep optimizer inputs current.

Core activities:

1. collect object stats for stale objects
2. analyze segment behavior/fragmentation context
3. run automatic SQL tuning workflows

This is the proactive loop that stops your system from silently aging into chaos.

---

## 5. Stale Statistics and Thresholds

Oracle tracks object change and marks objects stale when thresholds are crossed.

Default stale threshold discussed:

- around 10% data change (adjustable, including object-level strategy)

That flag drives auto stats refresh decisions during maintenance windows.

---

## 6. OLTP vs Warehouse Policy Differences

### OLTP / transactional systems

- continuous row-level change
- auto monitoring and refresh are very useful

### Batch-heavy warehouse systems

- large periodic loads
- often better to collect stats manually after load and lock as needed
- less value in constant auto-stat churn between static periods

Same database engine, very different maintenance rhythm.

---

## 7. AWR: Historical Repository Backbone

AWR (Automatic Workload Repository) stores historical performance statistics,
threshold context, and diagnostic data used for comparison and analysis.

Snapshots are the core mechanism:

- collected periodically (not continuously)
- used as point-in-time captures
- compared across intervals for behavior change

Think less "live camera feed," more "high-frequency snapshots assembled into an
operational timeline."

---

## 8. ADDM: Automated Differential Diagnosis

ADDM (Automatic Database Diagnostic Monitor):

- compares snapshots
- highlights top resource consumers
- surfaces high-wait SQL/sessions
- provides findings and recommendations

It runs automatically between successive snapshots and can be run manually for
specific periods when you need focused forensics.

---

## 9. CDB/PDB Considerations

In multitenant environments, monitoring scope matters:

- CDB/root-level behavior
- per-PDB behavior

Parameter controls such as PDB-level AWR snapshot behavior determine how deep
visibility goes per container. Use both root and PDB context where needed.

---

## 10. Advisors and Actionability

ADDM findings feed advisor workflows so you are not staring at raw counters
forever.

Potential action areas include:

- SQL tuning actions
- object/statistics strategy adjustments
- memory/undo/recovery advisor inputs
- workload prioritization and capacity changes

Not every heavy job is a problem. Some jobs are just large by design. The point
is to identify abnormal change, not punish legitimate workload.

---

## 11. Key Takeaways

- performance management starts with baselines, not intuition.
- AWR snapshots provide historical evidence.
- ADDM gives focused "what changed and why" insight.
- automated maintenance keeps optimizer inputs from going stale.
- tuning is iterative: compare, diagnose, test, apply, monitor.