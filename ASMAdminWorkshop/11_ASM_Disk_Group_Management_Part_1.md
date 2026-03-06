## Lesson 11 - ASM Disk Group Management Part 1 (in which adding one disk can trigger an entire moving company)

And look, this chapter focuses on operational disk-group management: how to inspect state, add/remove disks, and monitor rebalance activity without accidentally turning maintenance into an outage festival.

By the end of this lesson, you should be able to:

- Read disk group and disk metadata from ASM dynamic performance views
- Use ASMCMD commands to inspect disk groups, disks, and operations
- Add, drop, and undrop disks with SQL and ASMCMD workflows
- Understand when rebalance is required, deferred, or disabled
- Monitor live ASM operations and interpret "no rows selected" correctly

---

## 1. Viewing Disk Group Information

Primary SQL views:

- `V$ASM_DISKGROUP`
- `V$ASM_DISKGROUP_STAT`

`V$ASM_DISKGROUP`:

- Rich disk group information
- Includes discovery-related behavior
- Uses internal identifiers such as `GROUP_NUMBER`

`V$ASM_DISKGROUP_STAT`:

- Similar stats-oriented output
- Does not trigger fresh discovery behavior
- Better for lighter-weight repeated checks

Context matters:

- Connected to ASM instance: full ASM-side visibility
- Connected to database instance: visibility scoped to ASM resources relevant to that database context

---

## 2. Viewing Disk-Level Information

Primary SQL views:

- `V$ASM_DISK`
- `V$ASM_DISK_STAT`

`V$ASM_DISK`:

- Disk-level state/details
- Can be filtered by disk group or queried globally

`V$ASM_DISK_STAT`:

- Disk statistics without discovery-refresh behavior

These are your "what exists" and "how it is behaving" tables before making any structural change.

---

## 3. ASMCMD Views of the Same Reality

Useful commands:

- `lsdg` (disk group summary and metrics)
- `lsdsk` (disk-level information)
- `lsop` (current operations, including rebalance)

Typical `lsdsk` options can expose additional detail (timestamps, path/mount context, ownership and state columns depending on flags/platform).

If not connected to an active ASM instance context, output can be limited to what is available from disk headers.

In other words: if the output looks thin, check your connection context before declaring the command haunted.

---

## 4. Extending a Disk Group (Add Disk)

You can add capacity with SQL:

```sql
ALTER DISKGROUP DATA ADD DISK '/path/newdisk1';
```

Or equivalent ASMCMD workflows/scripted command execution.

Validation still applies:

- Disk compatibility must match group requirements
- Sector/attribute constraints must remain valid

Adding disks typically triggers rebalance so extents redistribute more evenly.

---

## 5. Dropping a Disk (and Why Rebalance Matters More Here)

Drop example:

```sql
ALTER DISKGROUP DATA DROP DISK DATA_0007;
```

Dropping is not just metadata cleanup:

- If data exists on that disk, ASM must relocate extents
- Rebalance is how "disk removed" becomes "data still accessible"

With mirroring in place, some workflows allow staged/deferred movement decisions. Without proper redundancy, dropping can become a very loud mistake very quickly.

---

## 6. Rebalance Control and Power

Core setting from prior chapter:

- `ASM_POWER_LIMIT`
  - `0` = disable automatic rebalance
  - `1` = default low-impact baseline
  - Up to `1024` = high parallelism / high resource use

You can control rebalance power via:

- Instance-level setting
- Operation-level directives in disk group statements

Tradeoff remains simple:

- Higher power -> faster completion, more resource pressure
- Lower power -> slower completion, less immediate impact

Choose based on workload window, not optimism.

---

## 7. Add and Drop in One Operation

ASM supports combined structural edits:

```sql
ALTER DISKGROUP DATA
  ADD DISK '/path/newdisk2'
  DROP DISK DATA_0005
  REBALANCE POWER 8;
```

Equivalent scriptable workflows can be done via ASMCMD command execution models.

Operationally, final state changes are tied to rebalance completion behavior.

---

## 8. Monitoring Active Operations

Primary monitoring view:

- `V$ASM_OPERATION`

If query returns no rows:

- No active ASM operations at that moment

ASMCMD equivalent:

- `lsop`

These are your "is anything currently moving?" truth sources during maintenance.

---

## 9. Undrop Disk Scenarios

You can undrop a disk when conditions allow, typically before full rebalance/drop finalization.

Example:

```sql
ALTER DISKGROUP DATA UNDROP DISKS;
```

Use case:

- Temporary outage/controller issue marked disk for drop path
- Underlying issue fixed before irreversible extent relocation/finalization

This is the closest thing ASM offers to "we panicked, then recovered gracefully."

---

## 10. Key Takeaways

- Use `V$ASM_DISKGROUP(_STAT)` and `V$ASM_DISK(_STAT)` for SQL-side visibility.
- Use `lsdg`, `lsdsk`, and `lsop` for fast operational checks in ASMCMD.
- Add/drop disk operations are inseparable from rebalance planning.
- `ASM_POWER_LIMIT` controls speed-vs-impact and should be tuned intentionally.
- `V$ASM_OPERATION` is the canonical live-operation monitor; no rows means no active operation.
