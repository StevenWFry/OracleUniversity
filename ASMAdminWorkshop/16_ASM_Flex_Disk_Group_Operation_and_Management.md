## Lesson 16 - ASM Flex Disk Group Operation and Management (in which storage policy becomes database-specific instead of one-size-fits-all)

Everything so far applied to general disk group administration. Flex disk groups add targeted policy control so different databases/PDBs can behave differently inside the same storage universe without everyone fighting over one default setting.

By the end of this lesson, you should be able to:

- Explain what makes a disk group "Flex" and when to use it
- Create or convert disk groups to Flex redundancy mode
- Understand file groups as per-entity policy containers
- Configure quota groups and attach file groups to quota governance
- Set file-group properties such as redundancy, priority, and restore behavior

---

## 1. Why Flex Disk Groups Exist

Flex disk groups are built for database-oriented policy customization.

Goal:

- Let storage behavior follow application/database context
- Avoid forcing every file type and tenant into one blanket redundancy/priority setting

Use cases include:

- Different policy for CDB vs individual PDBs
- Different treatment for data, archive, control, and recovery categories
- Cluster/volume-specific policy overlays where applicable

Think of Flex as "storage classes with opinions."

---

## 2. Creating or Converting to Flex

Flex is enabled by using Flex redundancy mode on the disk group.

Create pattern:

```sql
CREATE DISKGROUP DG_FLEX FLEX REDUNDANCY
  DISK '/path/d1','/path/d2', ...;
```

Conversion pattern (from mirrored groups):

```sql
ALTER DISKGROUP DG_NAME MOUNT RESTRICTED;
ALTER DISKGROUP DG_NAME CONVERT REDUNDANCY FLEX;
```

Baseline constraints:

- Compatibility must be `12.2` or higher
- Default allocation unit remains in the standard range (commonly 4 MB baseline)

---

## 3. File Groups: The Core Flex Policy Unit

A file group is where you define policy for a specific entity's files inside a disk group.

Key rules from this chapter:

- A disk group can contain many file groups
- A file group belongs to one disk group
- A file group is associated to one logical entity scope (for example, one database/PDB/volume context)
- An entity can have one file group per disk group, but can have different file groups across different disk groups

So:

- Same database + two disk groups -> two file groups possible (one per group)
- Same database + same disk group -> one file group for that entity in that group

---

## 4. What File Group Properties Control

File-group properties can define:

- Redundancy policy by file type
- Rebalance/remirror priority
- Restoration preference
- Quota linkage
- Ownership/operational metadata

Example idea:

- One policy for control-like critical structures
- Different policy for data files
- Different policy again for archive/recovery streams

This is where Flex stops being a checkbox and becomes a governance tool.

---

## 5. Quota Groups: Capacity Governance Layer

Quota groups define aggregate space limits and can be reused by multiple file groups.

Rules:

- A file group is attached to one quota group at a time
- A quota group can serve multiple file groups
- Quota logic is enforced at grouped policy level

Creation and attachment pattern:

```sql
ALTER DISKGROUP DG_FLEX
  ADD QUOTAGROUP QG_PDB1 QUOTA 500G;

ALTER DISKGROUP DG_FLEX
  MODIFY FILEGROUP FG_PDB1
  SET QUOTAGROUP QG_PDB1;
```

This lets you separate "how mirrored/performed" from "how much space allowed."

---

## 6. SQL Administration Patterns

Typical SQL operations include:

- Add file group
- Modify file group properties
- Add/modify quota groups
- Set file-category redundancy or priority behavior

Example structure:

```sql
ALTER DISKGROUP DG_FLEX
  ADD FILEGROUP FG_PDB1 FOR DATABASE PDB1;

ALTER DISKGROUP DG_FLEX
  MODIFY FILEGROUP FG_PDB1
  SET ATTRIBUTE 'rebalance.priority'='HIGH';
```

Exact property names vary by release/feature context, but the policy model is consistent.

---

## 7. ASMCMD Administration Patterns

ASMCMD supports parallel lifecycle operations for Flex objects, including:

- create/list/modify/remove file groups
- create/list/modify/remove quota groups
- move file groups between quota groups
- move files between eligible file groups when needed

Many scripted ASMCMD flows are XML-driven in this lesson context, which is great news for people who enjoy structured automation and mildly terrifying for everyone else.

---

## 8. Visibility Views for Flex Objects

Views highlighted:

- `V$ASM_FILEGROUP`
- `V$ASM_FILEGROUP_FILE`
- `V$ASM_FILEGROUP_PROPERTY`

Use these to audit:

- Which file groups exist
- Which files/entities map to them
- Which properties/priority/redundancy policies are active

Because if you cannot query it, you do not really control it.

---

## 9. Priority in Rebalance/Restore Work

File-group priority influences operation order for restoration/rebalancing.

Common tiers referenced:

- `HIGHEST`
- `HIGH`
- `MEDIUM` (default)
- `LOW`
- `LOWEST`

Use high priority for critical recovery paths, not for everything. If everything is urgent, nothing is.

---

## 10. Key Takeaways

- Flex disk groups provide per-entity, policy-driven storage behavior.
- File groups are the core policy unit; quota groups are the capacity-governance unit.
- One disk group can host many file groups for different entities.
- Compatibility 12.2+ is required for Flex feature set.
- Query `V$ASM_FILEGROUP*` views to validate policy state before and after changes.
