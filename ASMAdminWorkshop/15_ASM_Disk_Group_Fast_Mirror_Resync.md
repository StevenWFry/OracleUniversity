## Lesson 15 - ASM Disk Group Fast Mirror Resync (in which your failed disk returns and ASM only reconciles what changed)

And look, this chapter covers fast mirror resync, which is basically ASM saying, "I am not rebuilding the entire planet if only part of it changed."

By the end of this lesson, you should be able to:

- Explain how fast mirror resync works after temporary disk outages
- Control resync parallelism and monitor operation estimates
- Understand interrupted-resync restart behavior in newer releases
- Configure failgroup repair-time alerting semantics
- Use preferred read failgroups and disk I/O stats for performance-aware design

---

## 1. Why Fast Mirror Resync Exists

If mirroring is enabled, ASM can survive disk-level failure by serving data from surviving copies.

Flow:

1. Disk fails or is taken offline
2. Workload continues from mirror copies
3. Disk returns
4. ASM resyncs changed extents instead of rebuilding everything

That "changed extents only" behavior is where the time savings come from.

Without mirroring, this recovery path does not exist, and corruption/failure can become unrecoverable data loss.

---

## 2. Temporary Disk Failure Timeline

During outage:

- Reads/writes continue on available mirror copies
- Availability is preserved if redundancy + free space are sufficient

During recovery:

- Brought-back disk is resynchronized with only delta changes
- Full rebalance-style heavy movement is avoided when not required

This is exactly why earlier chapters obsessed over capacity reserve under redundancy.

---

## 3. Resync Power and Parallelism

Resync uses power controls similar to rebalance.

Typical range:

- `1` to `1024` worker parallelism
- default baseline remains low (typically `1`)

Example SQL patterns:

```sql
ALTER DISKGROUP DATA ONLINE DISK DATA_0001 POWER 100;
ALTER DISKGROUP DATA ONLINE ALL POWER 32;
```

ASMCMD provides equivalent operational workflows.

Higher power means faster catch-up and higher resource impact. You know the drill by now.

---

## 4. Interrupted Resync: Old Pain vs New Behavior

Historically:

- If resync was interrupted (forced dismount, ASM failure, etc.), restart often meant replaying everything again

Modern 23ai behavior highlighted in this lesson:

- Work is tracked in phases
- Completed phases are checkpointed in metadata
- Restart resumes from last completed phase (unfinished phase is replayed, not whole operation)

This dramatically reduces repeat work after interruptions.

---

## 5. Estimating and Monitoring Resync

Use:

- `V$ASM_OPERATION` for running operations and estimated remaining time
- `asmcmd lsop` for command-line operational visibility

With richer partner metadata and improved operation tracking, estimates are generally more reliable in newer versions.

Because "ETA unknown" is technically honest but emotionally unhelpful.

---

## 6. Failgroup Repair Time Behavior

Failgroup repair-time settings exist as disk group attributes.

Transcript emphasis for newer behavior:

- In 23ai, elapsed `failgroup_repair_time` primarily acts as alert signaling rather than automatic destructive action

Examples:

```sql
ALTER DISKGROUP DATA SET ATTRIBUTE 'failgroup_repair_time'='48h';

CREATE DISKGROUP DATA NORMAL REDUNDANCY
  DISK ...
  ATTRIBUTE 'failgroup_repair_time'='12h';
```

ASMCMD attribute commands can set equivalent values.

---

## 7. Preferred Read Failgroups

ASM can distribute read traffic across mirror copies instead of forcing all reads through one "primary" path.

Benefits:

- Better read load distribution
- Reduced hotspot pressure
- Better site/node-local read options in clustered layouts

Configuration can involve:

- Disk group preferred read attributes
- Backward-compatible parameter paths (for example, `ASM_PREFERRED_READ_FAILURE_GROUPS`) where applicable

Topology examples in this lesson:

- 2-node normal redundancy: each site can favor different mirror copies
- High redundancy: read paths can be spread across more copies
- >3 nodes: nodes share preferred copies but still reduce cross-node contention

---

## 8. Disk I/O Statistics Views

Key views/commands:

- `V$ASM_DISK_IOSTAT`
- `asmcmd lsdsk` (with options for richer detail)
- `asmcmd iostat`

As with other `_STAT` patterns:

- Stats-focused output is designed for operational monitoring
- Discovery overhead is minimized compared with full discovery-oriented paths

Use these to validate whether your "performance strategy" is real or just PowerPoint.

---

## 9. Performance and Scalability Design Guidelines

Guidelines emphasized in this chapter:

- Separate disk groups by workload type (for example, data vs recovery)
- Keep disk sizes consistent within mirrored groups where possible
- Plan security/isolation tradeoffs:
  - Separate disk groups per database can improve tenant isolation
  - But increases operational overhead
- Shared disk groups simplify administration but reduce isolation boundaries

No universal answer. Pick the model that matches your risk, governance, and ops capacity.

---

## 10. Key Takeaways

- Fast mirror resync restores changed extents, not entire disks, after temporary failures.
- Resync power settings control speed-vs-resource tradeoff.
- 23ai restart behavior avoids redoing fully completed resync phases.
- `V$ASM_OPERATION` and `lsop` are core visibility tools for live resync tracking.
- Preferred read failgroups and disk-group design choices materially affect read performance and scalability.
