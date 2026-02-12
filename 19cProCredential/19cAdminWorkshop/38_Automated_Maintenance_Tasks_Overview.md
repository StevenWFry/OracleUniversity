# 38 - Automated Maintenance Tasks Overview

Welcome to the part where Oracle quietly does useful work at night so your SQL
can stop improvising bad decisions in broad daylight.

This chapter explains automated maintenance tasks: why they exist, what runs,
and when you should trust automation versus doing it yourself.

---

## 1. Why Automated Maintenance Exists

Oracle's optimizer needs fresh object statistics to choose efficient execution
plans. Without decent stats, plan quality drops, performance drifts, and users
start describing the app as "slow" with increasing emotional intensity.

Optimizer decisions depend on things like:

- row counts
- value distribution and repetition
- available access paths (indexes/full scans)
- estimated resource cost per path

So yes, statistics are optional in the same way brakes are optional.

---

## 2. Core Components in the Maintenance Window

During the automated maintenance window, Oracle runs a sequence of tasks:

1. automatic statistics collection
2. segment advisor analysis
3. automatic SQL tuning

The order matters:

- stats first (give optimizer current reality)
- fragmentation/segment info second
- SQL tuning third (advise based on freshest metadata)

---

## 3. Stale Statistics and Object Marking

Oracle tracks object changes over time. When change crosses stale threshold, the
object is flagged for stats refresh.

Default stale threshold discussed:

- ~10% data change (configurable)

You can override stale behavior at object level where needed.

This is how Oracle avoids recollecting stats blindly on everything, every time.

---

## 4. Segment Advisor's Role

Segment advisor contributes structural information, including fragmentation
signals and space behavior.

Why this matters:

- optimizer cost model quality
- DBA decisions on shrink/rebuild/reorg

So it is not just "space trivia." It affects plan outcomes and maintenance
priorities.

---

## 5. Automatic SQL Tuning Flow

SQL tuning advisor then reviews SQL identified as expensive/inefficient.

Input sources include:

- internal diagnostics (ADDM)
- high-frequency/high-cost SQL patterns

Typical recommendations:

- create or adjust indexes
- use SQL profile suggestions
- refresh/adjust statistics strategy
- change stale settings where plan quality is impacted

And yes, frequently executed SQL is prioritized over one-off curiosities.

---

## 6. OLTP vs Data Warehouse Strategy

### OLTP / transactional systems

Data changes continuously, row by row. Automated maintenance is highly valuable
because stale transitions are unpredictable and frequent.

### Batch-style warehouse systems

If loads are periodic and data stays mostly static between loads:

- collect stats manually after load
- optionally lock stats
- avoid unnecessary recurring automation where it adds little value

Different workload, different policy. One-size-fits-all tuning is how you get
consistent disappointment.

---

## 7. Scheduler Window Is Operationally Important

Maintenance jobs run inside scheduler windows. As admin, you should verify:

- when the window runs
- whether it aligns with real workload quiet periods
- whether jobs complete inside window duration

If windows are misaligned, you'll either miss maintenance or compete with
business load, which is a great way to annoy everyone at once.

---

## 8. Performance Consistency Is the Real Target

The chapter emphasizes a practical truth: users care less about peak speed and
more about predictability.

If a query normally takes ~2 seconds, users expect ~2 seconds every time. Plan
instability and stale metadata break that expectation faster than almost any
other tuning failure.

Automated maintenance is part of keeping that consistency contract intact.

---

## 9. Key Takeaways

- automated maintenance exists to keep optimizer inputs sane.
- stats -> segment analysis -> SQL tuning is intentional sequencing.
- stale thresholds should be reviewed, not blindly accepted.
- OLTP and warehouse patterns need different maintenance policies.
- scheduler windows are not cosmetic; they are execution reality.