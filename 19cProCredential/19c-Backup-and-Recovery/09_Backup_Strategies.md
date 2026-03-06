## Lesson 9 - Backup Strategies (in which "just back it up" turns into actual engineering)

And look, backup strategy is not about collecting random backup files like baseball cards. It is about deciding what you need to recover, how fast, and with how much pain.

By the end of this lesson, you should be able to:

- Compare backup types RMAN can produce
- Explain full, level 0, level 1 cumulative, and level 1 differential logic
- Choose between image copies and backup sets
- Use strategy options like block change tracking and incrementally updated backups
- Align backup design to workload patterns, including Data Guard and data warehouses

---

## 1. Why Strategy Matters

How often you back up and what format you choose directly affects:

- recovery time
- storage usage
- operational complexity

Fast backup with slow restore is still a bad day. Cheap storage with unusable retention is also a bad day.

---

## 2. Backup Terminology (old vs current language)

Historically:

- **Cold backup**: database shut down, copy files (consistent by shutdown)
- **Hot backup**: database open, tablespaces put in backup mode during copy

Current RMAN-centric terminology focuses more on:

- whole vs partial
- full vs incremental
- image copy vs backup set

Same goal, better tooling, fewer manual heroics.

---

## 3. RMAN Backup Types That Matter

### Full backup

- Contains all used blocks in data files.
- No incremental lineage marker.

### Incremental level 0

- Operationally equivalent starting point to full for incrementals.
- Used as base for later level 1 incrementals.

### Incremental level 1 cumulative

- Captures changes since the most recent level 0.
- Usually larger than differential level 1.

### Incremental level 1 differential

- Captures changes since the most recent incremental backup (level 1 or level 0 base path).
- Usually smaller and more frequent-friendly.

In plain English: cumulative is "since base," differential is "since last step."

---

## 4. Image Copy vs Backup Set

### Image copy

- Bit-for-bit file copy.
- Large on disk.
- Very fast and simple restore path.

### Backup set

- RMAN proprietary backup format with compression options.
- Smaller footprint.
- Restore involves RMAN reconstruction/decompression workflow.

So image copy is big but quick to use. Backup set is efficient but more processing-heavy at restore time.

---

## 5. RMAN Execution Targets

RMAN can back up to:

- disk (often FRA-backed locations)
- tape/SBT channels (for example Oracle Secure Backup)
- compatible third-party media manager integrations

If you use tape, channel planning matters. If you ignore channels, throughput will remind you.

---

## 6. Strategy Option: Block Change Tracking (BCT)

BCT keeps a change map so incremental backups can find changed blocks faster.

Use case:

- frequent incremental backups
- relatively low change rate between backups

Rule of thumb from this lesson flow:

- most effective when block change percentage is low (for example well below heavy-change extremes)

If almost everything changes every cycle, BCT advantage shrinks and complexity stays.

---

## 7. Strategy Option: Incremental Forever / Incrementally Updated Backups

Pattern:

1. Create base image copy (level 0 equivalent foundation)
2. Take periodic incrementals
3. Apply incrementals to roll image copy forward

Benefit:

- keeps a near-current restoreable copy
- can reduce recovery steps compared with rebuilding from older full + long chain

Also pairs well with `SWITCH` workflows when you need to redirect to a valid image copy quickly.

---

## 8. Multiplexing, Channels, and Throughput

Key knobs:

- channel count
- files per set
- `MAXOPENFILES`
- device type (disk vs SBT)

General principle:

- tune channels to available I/O/tape drives
- avoid creating hardware bottlenecks with optimistic configuration

RMAN cannot outrun slow storage controllers no matter how inspirational your command block is.

---

## 9. Strategy Comparisons

### Full + incremental chain strategy

- full/level 0 base
- incremental updates over time
- archived logs for final roll-forward

Good for storage efficiency, but restore path may involve more steps.

### Incrementally updated image copy

- bigger storage footprint
- faster failover-like file replacement paths (`SWITCH`)
- good when recovery speed is strict

### Backup offload to physical standby (Data Guard)

- run heavy backup load on standby instead of primary
- backups remain usable for primary/standby recovery paths
- helps protect primary performance windows

If your primary is business-critical and latency-sensitive, this is often the adult choice.

---

## 10. Read-Only and Data Warehouse Considerations

### Read-only tablespaces

- Use `SKIP READONLY` when appropriate.
- Back up read-only data after changes, not every cycle forever.

### Data warehouse patterns

- Partition older data into read-only structures where possible.
- Compress backup sets for large historical segments.
- Back up volatile areas more frequently than static segments.
- Index backup frequency can be tuned based on change behavior.

Because backing up 100% of a warehouse at the same cadence "just to be safe" is how backup windows become geological time.

---

## 11. Practical Checklist

1. Define RPO/RTO first, then choose backup format.
2. Pick retention and storage model before scheduling jobs.
3. Decide image copy vs backup set per recovery-speed target.
4. Enable BCT only when incremental profile benefits.
5. Use standby offload if primary workload is sensitive.
6. Reassess strategy quarterly as data growth changes.

---

## 12. Wrap-Up

Backup strategy is where RMAN features become business resilience. You now have the design map for:

- backup type selection
- incremental model choice
- performance tuning knobs
- standby offload options
- workload-specific optimizations

And yes, this is all before the outage. That is the whole point.
