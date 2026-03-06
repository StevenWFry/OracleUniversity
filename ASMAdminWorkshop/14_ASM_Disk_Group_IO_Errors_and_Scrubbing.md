## Lesson 14 - ASM Disk Group IO Errors and Scrubbing (in which storage corruption gets audited, repaired, and occasionally publicly shamed)

This chapter covers how ASM reacts when I/O goes bad, how it uses mirrors and partner metadata to recover, and how scrubbing/check operations keep disk groups from quietly decaying into chaos.

By the end of this lesson, you should be able to:

- Explain ASM behavior for read/write errors with and without mirroring
- Describe how partner status metadata affects recovery decisions
- Run disk group scrubbing in detect-only and repair modes
- Use force/stop controls when scrub workload meets heavy I/O
- Validate and repair ASM metadata consistency using `CHECK` operations

---

## 1. How ASM Handles I/O Errors

When ASM encounters bad reads/writes, it tries to preserve service using redundancy first.

Read-side behavior:

- ASM prefers consistent mirror copies when available
- Without mirroring, logical corruption can become unrecoverable data loss

Write-side behavior:

- If possible, ASM rewrites to recover the bad area in place
- If in-place write is not usable, ASM writes to a new location and marks old space unusable
- If write recovery continues to fail, ASM can offline the affected disk

This is ASM doing incident response while your app is still pretending everything is fine.

---

## 2. Partner Status Table Role in Recovery Paths

For recovery/resync decisions, ASM consults partner status metadata (introduced earlier in this course).

That metadata helps ASM decide:

- Which partners are online
- Which mirror sources are valid
- How to rebuild safely after errors

If required partner metadata is unavailable in a failure path, disk group availability can be severely affected, including taking the group offline in worst-case scenarios.

---

## 3. Scrubbing: Logical Corruption Detection and Repair

Scrubbing scans for logical corruption and optionally repairs it.

Modes:

- Detect-only scrub:
  - Find/report corrupt blocks
  - Do not repair
- Repair scrub:
  - Find and repair from valid mirrored copies

Rebalance interaction:

- If content checking is enabled, rebalance workflows can proactively validate/repair as movement occurs

Relevant attribute:

- `content.check` (enabled for automatic validation behavior in supported workflows)

---

## 4. 23ai Improvement: Extent-Level Scrubbing

In newer behavior, scrubbing can operate at finer extent scope rather than broad heavy scans.

Benefits:

- Smaller repair targets
- Lower overhead
- Faster completion for known-problem regions

This is "surgery with a scalpel" instead of "surgery with a bulldozer."

---

## 5. On-Demand Scrub Operations

Typical command patterns:

```sql
ALTER DISKGROUP DATA SCRUB;
ALTER DISKGROUP DATA SCRUB REPAIR;
```

Targeted forms can scope scrub to specific file/block ranges and wait semantics, depending on syntax/options used in your environment.

Behavior notes from this lesson:

- Without `REPAIR`, corruption is reported only
- With `REPAIR`, ASM attempts correction

---

## 6. FORCE and STOP Controls

Under high I/O, scrub operations can be throttled or aborted by default protections.

Control options:

- `FORCE`:
  - Attempts scrub even when normal load checks would block it
- `STOP`:
  - Terminates active scrubbing when workload pressure is too high

Example:

```sql
ALTER DISKGROUP DATA SCRUB STOP;
```

Because sometimes "fix corruption now" loses to "keep production alive for the next hour."

---

## 7. Monitoring Scrub/Rebalance Progress

Use:

- `V$ASM_OPERATION`

This view shows active ASM operations (scrub/rebalance/etc.) and progress context.

If there are no rows:

- No active tracked ASM operation at that moment

For failures:

- Check alert log and trace files for root cause details (for example, operation denied under high I/O pressure)

---

## 8. Metadata Consistency Checks (`CHECK` / `REPAIR`)

You can validate disk group metadata consistency using:

```sql
ALTER DISKGROUP DATA CHECK;
ALTER DISKGROUP DATA CHECK REPAIR;
```

Conceptually similar to file-system consistency checks:

- `CHECK`: detect/report metadata inconsistency
- `CHECK REPAIR`: detect and attempt correction

Summary/errors are surfaced in SQL response and diagnostics (alert/trace outputs).

---

## 9. Deprecated Legacy Clauses

Older check/scrub clause patterns have been reduced or deprecated as ASM operations became more consolidated and robust in newer releases.

Practical guidance:

- Use current syntax and current diagnostics
- Avoid scripting around obsolete clauses unless maintaining legacy environments

---

## 10. Key Takeaways

- Mirroring is the difference between recoverable I/O corruption and potential data loss.
- ASM can recover in-place or remap writes; persistent failure can offline disks.
- Scrub can detect-only or detect+repair, with finer extent-level behavior in newer releases.
- Use `FORCE` cautiously and `STOP` when production load demands it.
- `ALTER DISKGROUP ... CHECK [REPAIR]` is your metadata integrity safety net.
