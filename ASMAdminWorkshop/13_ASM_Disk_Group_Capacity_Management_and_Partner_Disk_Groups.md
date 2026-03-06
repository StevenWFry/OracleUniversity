## Lesson 13 - ASM Disk Group Capacity Management and Partner Disk Groups (in which "free space" turns out to be a complicated lie)

This chapter goes deeper into disk-group operations where capacity math, redundancy guarantees, and rebuild metadata all collide. If you only look at `FREE_MB` and call it a day, ASM will eventually send you a strongly worded outage.

By the end of this lesson, you should be able to:

- Calculate usable disk group capacity correctly under redundancy
- Interpret `TOTAL_MB`, `FREE_MB`, `REQUIRED_MIRROR_FREE_MB`, and `USABLE_FILE_MB`
- Explain what partner status tables do during failure and rebalance workflows
- Understand why partner metadata improves rebuild throughput in newer releases
- Recognize read-availability improvements during rebalance in modern ASM behavior

---

## 1. Capacity Management: Redundancy Changes the Math

Raw free space is not the same thing as safely usable space.

When redundancy is enabled, ASM must keep enough reserve to survive failure scenarios:

- `NORMAL` redundancy: reserve for at least one failgroup failure
- `HIGH` redundancy: reserve for at least two failgroup failures

If you ignore that reserve, you can end up with files that are no longer fully protected by mirrors, which is the storage equivalent of driving without seatbelts because "the road looks fine."

---

## 2. Core Capacity Columns You Actually Need

From `V$ASM_DISKGROUP`, key columns are:

- `TOTAL_MB`
- `FREE_MB`
- `REQUIRED_MIRROR_FREE_MB`
- `USABLE_FILE_MB`

What they mean:

- `REQUIRED_MIRROR_FREE_MB`:
  - Space reserved to preserve configured failure tolerance
- `USABLE_FILE_MB`:
  - Space you can really allocate to new files while still honoring redundancy policy

For `EXTERNAL` redundancy:

- `REQUIRED_MIRROR_FREE_MB` is typically zero because ASM is not doing internal mirroring

ASMCMD equivalent visibility:

- `lsdg` exposes similar capacity signals for operational checks

---

## 3. Example Capacity Calculation (Normal Redundancy)

Practical formula (normal redundancy):

```text
usable_file_mb = (free_mb - required_mirror_free_mb) / 2
```

Example from this lesson:

- 6 x 1 GB disks
- `FREE_MB` around 3.7 GB (after metadata overhead)
- `REQUIRED_MIRROR_FREE_MB` around 1.0 GB

Then:

```text
(3.7 - 1.0) / 2 = 1.35 GB usable
```

So yes, "3.7 GB free" can still mean "about 1.3 GB safely usable." Storage accounting is fun and nobody asked for it.

---

## 4. Partner Status Table (PST): Why Rebuild Got Faster

Partner status table metadata supports rebuild/rebalance intelligence by tracking disk-partner relationships and health state.

Tracked concepts include:

- Disk identity/number
- Disk online/offline status
- Partner disk mapping
- Failgroup context
- Heartbeat/state signals

Not every disk necessarily hosts the same PST layout copy density; metadata placement is controlled and finite.

Operationally, this metadata allows ASM to make faster, more targeted rebuild decisions when failures or topology changes occur.

---

## 5. Partner Disk Concepts and Scale

For mirrored redundancy behavior, partner-disk metadata helps identify where mirrored rebuild work should come from and where it should land.

Values highlighted in this lesson:

- Default partner count baseline: 8
- Maximum partner count: 24

Use fixed/internal ASM views for deeper introspection where available in your environment and release.

---

## 6. Rebuild Throughput Gains in 23ai

The lesson highlights major gains with upgraded partner-table behavior:

- Better multi-source rebuild selection
- Reduced hotspot concentration
- Throughput improvements (example range noted: roughly 25 MB/s to 150 MB/s)

Scale-dependent gains discussed:

- About 3x faster rebuild with 5 or more cells
- About 2x faster rebuild with 3 to 4 cells

Illustrative example:

- ~133 minutes in older behavior
- ~43 minutes in newer behavior

This is the difference between "maintenance window" and "why are people still on this bridge call."

---

## 7. Read Availability During Rebalance

Another major point in this lesson:

- Modern behavior keeps non-moving and relevant mirror paths readable during rebalance
- Read blocking/locking pain is greatly reduced compared to older patterns

That means rebalancing is less likely to feel like a temporary denial-of-service event.

---

## 8. Fast Mirror Resync Context

Fast mirror resync is part of the broader resiliency strategy:

- Tracks what changed while a disk path was unavailable
- Resynchronizes only what is needed instead of rebuilding everything from scratch

Combined with partner metadata improvements, this reduces rebuild scope and time in common failure/recovery workflows.

---

## 9. Capacity and Performance Checklist

Before major disk-group operations:

1. Check `V$ASM_DISKGROUP` capacity columns, not just `FREE_MB`
2. Verify redundancy reserve is healthy (`REQUIRED_MIRROR_FREE_MB`)
3. Confirm `USABLE_FILE_MB` is sufficient for incoming workload
4. Assess rebalance/rebuild impact windows and power settings
5. Monitor live operations so performance expectations match reality

Because "we had free space" is not a postmortem defense.

---

## 10. Key Takeaways

- Capacity planning in ASM is redundancy-aware, not raw-space-only.
- `REQUIRED_MIRROR_FREE_MB` protects failure tolerance; `USABLE_FILE_MB` is the safer planning number.
- Partner status tables improve rebuild decision quality and speed.
- 23ai-era partner metadata behavior can significantly reduce rebuild duration.
- Modern rebalance behavior improves read availability while storage movement is in progress.
