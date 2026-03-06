## Lesson 12 - ASM Disk Group Management Part 2 (in which maintenance windows become competitive data choreography)

Part 1 covered inspection and basic add/drop behavior. Part 2 is the heavier operations menu: replace, rename, resize, mount/dismount, drop, and rebalance control when storage changes are not hypothetical.

By the end of this lesson, you should be able to:

- Replace one or many disks with controlled rebalance behavior
- Distinguish disk-group rename workflows from disk rename workflows
- Resize disks/failgroups and understand resulting rebalance implications
- Mount and dismount disk groups with proper cluster-aware tooling
- Monitor and estimate rebalance work before committing to maintenance actions

---

## 1. Replacing Disks in a Disk Group

You can replace one disk or multiple disks in a coordinated operation.

Why bundle operations:

- Fewer maintenance cycles
- One coordinated rebalance period
- Less "surprise movement" between successive small changes

Operational caveat:

- Replacement requires actual data movement
- Rebalance power cannot be effectively treated as "off" for replacement workflows

So yes, this is a maintenance-window activity, not a lunch-break hobby.

---

## 2. Renaming a Disk Group (Plan First, Execute Second)

Disk-group rename is usually a planned workflow, often described in two phases:

1. Generate/validate rename plan metadata
2. Execute rename

Utilities can run both phases together by default, or as separate planned steps.

Critical requirement:

- Disk group must not be in normal mounted/active-client use mode during rename workflow

Use this when naming standards need cleanup, not when production is busiest and everyone is pretending nothing is fragile.

---

## 3. Renaming Disks Inside a Disk Group

This is a different operation than renaming the disk group itself.

Typical SQL pattern:

```sql
ALTER DISKGROUP FRA MOUNT RESTRICTED;
ALTER DISKGROUP FRA RENAME DISK 'FRA_0001' TO 'FRA_0101';
```

You can rename multiple disks in one controlled maintenance cycle.

Key condition:

- Restricted mount is generally required for safe disk-rename workflows
- During this operation, the group is not in normal client-serving posture

---

## 4. Resizing Disks and Failgroups

You can resize:

- Specific disks
- All disks in a failgroup

Growth behavior:

- Increased capacity becomes available after resize

Shrink behavior:

- If shrunk regions contain extents, ASM must relocate data
- Rebalance becomes part of the shrink path

This is why "just make it smaller" is never just that.

---

## 5. Mount and Dismount Operations

SQL options exist:

- `ALTER DISKGROUP ... MOUNT`
- `ALTER DISKGROUP ... DISMOUNT`

ASMCMD options exist as well.

Cluster best practice remains:

- Prefer `srvctl` for clustered environments
- You get immediate state registration with Clusterware, not just local action

Because local-only actions plus delayed cluster awareness is how operational confusion reproduces.

---

## 6. Client Visibility Before Risky Changes

Before dismounting or dropping anything, check clients:

- `V$ASM_CLIENT`
- `asmcmd lsct -g <diskgroup>`

This tells you who is currently using the group so you do not discover active dependencies by pulling the floor out from under them.

---

## 7. Dropping a Disk Group

SQL examples:

```sql
DROP DISKGROUP DATA;
DROP DISKGROUP DATA INCLUDING CONTENTS;
DROP DISKGROUP DATA FORCE;
```

ASMCMD can perform equivalent drop operations.

Choose flags based on state:

- Normal drop for clean scenarios
- `INCLUDING CONTENTS` when files remain
- `FORCE` when group health/pathing is damaged and standard flow is blocked

As always, "force" means you are accepting a higher blast radius.

---

## 8. Rebalance Behavior: What Actually Moves

When you add a disk:

- New writes can target new capacity
- Rebalance then moves only enough existing extents to restore distribution balance

When you remove/replace/shrink:

- Required extents are relocated to preserve availability/redundancy

Modern behavior emphasis from this lesson:

- Critical file restoration paths are prioritized first
- Less critical/secondary movement follows

So the order of recovery/rebalance work now favors operational survivability over perfect symmetry.

---

## 9. Rebalance Power and Dynamic Control

`ASM_POWER_LIMIT` and statement-level power clauses control worker parallelism.

Characteristics:

- Dynamic and adjustable during operations
- Higher power = faster completion, higher resource contention
- Lower power = gentler footprint, longer maintenance

Tune to workload window, not just impatience.

---

## 10. Estimating Work Before Pulling the Trigger

Use planning helpers before disruptive actions:

- `EXPLAIN WORK` style operations for projected movement
- `V$ASM_ESTIMATE` for estimated allocation-unit migration scope

Examples include estimating impact for:

- Drop-disk scenarios
- Bringing disks online after outage/offline intervals

This is your "measure before surgery" step. Skipping it is a bold strategy with poor historical outcomes.

---

## 11. Key Takeaways

- Replace and resize operations are rebalance-driven maintenance events.
- Disk-group rename and disk rename are distinct workflows with different constraints.
- Restricted mount is typically required for disk renaming operations.
- Prefer `srvctl` for mount/dismount in clustered deployments.
- Use `V$ASM_OPERATION`, `V$ASM_ESTIMATE`, and client views before committing high-impact changes.
