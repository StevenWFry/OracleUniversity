## Appendix B-1 - Point-in-Time Database Clones (in which ASM mirror copies become your instant clone machine)

This appendix covers point-in-time PDB cloning using ASM mirrors: split a mirror copy, create a clone from it, then rebalance to restore mirror protection on the source side.

By the end of this appendix, you should be able to:

- Explain mirror-split cloning for large PDBs
- List redundancy prerequisites for ASM clone workflows
- Run the two-phase clone flow: prepare mirror then create PDB
- Clone locally and across CDBs with a database link
- Monitor clone status and recover cleanly from failures

---

## 1. Why This Exists

Classic full-copy clone of a very large PDB can be slow, heavy, and generally hated by everyone with an outage window.

Mirror-split clone is faster because:

- Data is already mirrored in ASM
- You split the mirrored copy at a specific timestamp
- You build the clone from that split copy

Then rebalance recreates the missing mirror on the source side.  
So you get speed without permanently sacrificing redundancy.

---

## 2. Prerequisites

ASM must support mirroring for this workflow.

Valid redundancy models:

- Normal redundancy
- High redundancy
- Flex redundancy

If there is no mirror, there is nothing to split, and this entire appendix turns into wishful thinking.

---

## 3. Core Behavior After Split

When you split the mirror:

- Source and split copy are identical at the split timestamp
- New writes to source do not update split copy
- Clone created from split copy becomes independent
- Future writes in clone do not affect source

In short: same starting point, separate futures.

---

## 4. Two-Phase Local Clone Flow

### Phase 1: Prepare and split mirror copy

Run from the source PDB context.

Conceptual command pattern:

```sql
ALTER PLUGGABLE DATABASE PREPARE MIRROR COPY <mirror_copy_name>;
```

This freezes that mirror copy at point-in-time for clone use.

### Phase 2: Create new PDB from mirror copy

Run from CDB root.

Conceptual command pattern:

```sql
CREATE PLUGGABLE DATABASE <new_pdb_name>
  FROM <source_pdb_name>
  USING MIRROR '<mirror_copy_name>';
```

After successful creation, run rebalance on source disk group to restore full mirror layout.

---

## 5. Remote Clone Flow (Different CDB)

If target is in another CDB:

1. Create source-side user with required privileges
2. Grant at least:
   - `CREATE SESSION`
   - `CREATE PLUGGABLE DATABASE`
3. Create DB link on target CDB root
4. Create target PDB from source via DB link + mirror copy reference

Conceptual pattern:

```sql
CREATE PLUGGABLE DATABASE <target_pdb>
  FROM <source_pdb>@<db_link_name>
  USING MIRROR '<mirror_copy_name>';
```

The mirror mechanics are ASM-side; the DB link handles cross-CDB connectivity.

---

## 6. Monitoring Views

Useful views called out in this appendix:

- `V$ASM_DBCLONE_INFO` for clone process status
- `V$ASM_FILEGROUP` for file group metadata
- `V$ASM_FILEGROUP_PROPERTY` for clone/mirror-related properties

Use them to verify:

- split copy exists
- clone status (running/succeeded/failed)
- file group state tied to the operation

---

## 7. Failure and Cleanup Logic

Important operational nuance:

- If you abort before starting clone creation, mirror can generally be returned/reused
- If clone creation starts and fails, split copy is typically not reusable as-is

Cleanup path from lesson flow:

1. Drop prepared mirror copy
2. Rebalance disk group
3. Recreate mirror protection

Conceptual pattern:

```sql
ALTER PLUGGABLE DATABASE DROP MIRROR COPY <mirror_copy_name>;
ALTER DISKGROUP <dg_name> REBALANCE WAIT;
```

`WAIT` keeps the session attached until completion, so you can confirm recovery is done.

---

## 8. Operational Checklist

1. Confirm mirrored ASM redundancy is active
2. Prepare mirror copy in source PDB
3. Create clone PDB from mirror copy
4. Validate clone status in `V$ASM_DBCLONE_INFO`
5. Rebalance source disk group to restore mirror count
6. If failure occurs, drop bad mirror copy and rebalance again

Because "we will fix it later" is not a storage strategy.

---

## 9. Key Takeaways

- Point-in-time clone via ASM mirror split is built for fast large-PDB provisioning.
- The process is two-phase: prepare split copy, then create PDB from it.
- Split copy is point-in-time static; source and clone diverge immediately after split.
- Cross-CDB clone uses DB link plus the same mirror-copy model.
- Rebalance is required to restore mirror protection on source after clone operations.
