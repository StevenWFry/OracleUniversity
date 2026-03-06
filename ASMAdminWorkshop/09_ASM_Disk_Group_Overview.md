## Lesson 9 - ASM Disk Group Overview (in which storage gets promoted from "some disks" to a full governance regime)

This is Part 1 of disk group administration. Disk groups are the core storage unit in ASM, so once you understand these, the rest of ASM stops feeling like a magic trick performed by a very expensive wizard.

By the end of this lesson, you should be able to:

- Explain why disk groups are the fundamental ASM storage unit
- Create and delete disk groups with SQL, ASMCA, and ASMCMD workflows
- Identify major disk group attributes and what they control
- Query disk group attributes with dynamic views and ASM tools
- Understand compatibility and content-type settings that influence failure tolerance

---

## 1. Why Disk Groups Matter

No disk group, no ASM storage lifecycle. Everything hangs off disk groups:

- File placement
- Mirroring defaults
- Allocation behavior
- Volume-manager-backed services
- File-system exposure paths for non-database consumers

If ASM is the city, disk groups are the districts with their own zoning laws.

---

## 2. Refresher: Where Disk Groups Sit in the Stack

Operational flow:

1. Disks are discovered and assigned to disk groups
2. Disk group attributes define default behavior
3. ASM instances start and mount disk groups
4. Database instances connect as ASM clients
5. Clients perform direct I/O to shared storage using ASM metadata

In cluster examples, not every node needs a local ASM instance in Flex mode, but every node still needs access to the shared storage underneath.

---

## 3. Creating and Deleting Disk Groups

You can manage disk groups through:

- ASMCA (GUI path, limited attribute surface)
- ASMCMD (`mkdg`, script-driven XML workflows, command-line operations)
- SQL (`CREATE DISKGROUP`, `ALTER DISKGROUP`, `DROP DISKGROUP`) for full control

Example SQL shape:

```sql
CREATE DISKGROUP FRA NORMAL REDUNDANCY
  DISK '/path/disk1','/path/disk2'
  ATTRIBUTE 'compatible.asm'='23.0', 'compatible.rdbms'='23.0';
```

After creation, metadata is cluster-shared, but each ASM instance must mount the disk group to make it usable on that node.

---

## 4. ASMCA vs SQL: Convenience vs Full Control

ASMCA gives you quick setup:

- Pick candidate disks
- Set core compatibility and sector settings
- Create group and validate status

But ASMCA does not expose every advanced attribute in one place.

SQL gives complete syntax surface for:

- Full attribute matrix
- Fine-grained policy control
- Repeatable scripted deployment

So ASMCA is the helpful wizard. SQL is the actual steering wheel.

---

## 5. Key Disk Group Attributes (the knobs that matter)

Attributes called out in this lesson include:

- Allocation unit size (`1 MB` to `64 MB`, default `4 MB`)
- Disk repair time (how long offline disks are tolerated before forced action)
- Compatibility levels (`compatible.asm`, `compatible.rdbms`, volume-manager compatibility)
- Content checks and content type policy
- Physical and logical sector size controls
- Preferred read behavior (cluster/topology-sensitive)
- Storage type indicators
- Access-control and permission-related controls
- Failgroup repair timing
- Thin-provisioning behavior (discarding unused space after rebalance in supported setups)

You will not use every one daily, but when you need them, you really need them.

---

## 6. Inspecting Attributes

Primary visibility options:

- `V$ASM_ATTRIBUTE`
- `ASMCMD lsattr`

Notes from this lesson:

- Attribute visibility in views depends on compatibility level
- For older compatibility levels, some fields may not populate as expected

If attribute output looks incomplete, check compatibility before blaming SQL.

---

## 7. Compatibility Settings: One-Way Upgrades

Compatibility settings unlock feature sets per disk group.

Behavior to remember:

- You can raise compatibility
- You generally cannot roll it back after raising it

Practical guidance:

- Keep ASM and RDBMS compatibility aligned where possible
- Keep volume-manager compatibility at modern supported levels for the features you need

Do not "just bump compatibility" without a rollback strategy, because rollback may be fictional.

---

## 8. Content Type and Failure Tolerance

Content type (`data`, `recovery`, `system`) influences how ASM places mirrored extents for related workloads.

Why it matters:

- Without content-type-aware placement, primary and mirror placement patterns can overlap in failure-prone ways
- With correct content-type strategy, ASM can spread related mirror dependencies across safer fault domains

Result:

- Better survival odds under multi-disk failure scenarios
- Less chance of losing all copies of a specific critical component from the same client set

This is where "just mirror it" becomes "mirror it intelligently."

---

## 9. Maintenance Operations in This Chapter Scope

This first disk-group chapter focuses on:

- Create/delete lifecycle
- Attribute inspection and adjustments
- Mount-state awareness

Later parts go deeper into advanced attribute behavior, policy edge cases, and maintenance workflows.

---

## 10. Key Takeaways

- Disk groups are ASM's core storage unit and policy boundary.
- ASMCA is useful, but SQL gives full attribute-level control.
- `V$ASM_ATTRIBUTE` and `lsattr` are your core attribute inspection tools.
- Compatibility changes are mostly one-way and should be planned.
- Content type settings can materially improve failure tolerance across mirrored copies.
