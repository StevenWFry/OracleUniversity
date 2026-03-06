## Lesson 4 - Oracle ASM Components Part 2 (in which disks, extents, and stripes become a full-time logistics operation)

Part 1 covered the "who" (instances, disk groups, failure groups). Part 2 covers the "how" data is physically organized and consumed: ASM disks, allocation units, extents, striping, file naming, and client behavior. This is the chapter where storage becomes a shipping company with anxiety.

By the end of this lesson, you should be able to:

- Describe what an ASM disk is and what storage types can back it
- Explain how ASM discovers disks and when ASM disk names are assigned
- Summarize allocation unit and extent behavior in ASM
- Differentiate coarse and fine striping
- Explain file-level mirroring precedence and identify ASM clients/tools

---

## 1. ASM Disks: What They Really Are (and why they all need name tags)

An ASM disk is a storage device presented to ASM, such as:

- Physical disk or partition
- Logical volume/LUN from external storage
- NFS-provided storage path (OS-mounted and exposed)
- Exadata grid disk

You typically provide one or more discovery paths (often with wildcards), and ASM maps candidate storage into ASM disks for later disk group membership.

---

## 2. Disk Discovery and Naming (before and after ASM adopts them)

Before a disk joins a disk group:

- It is identified by its OS/device path

After it is added to a disk group:

- ASM assigns and manages internal ASM disk identity metadata

Important operational note:

- Device names can differ across nodes; ASM still normalizes and manages them internally once discovered and grouped.

---

## 3. Allocation Units (AUs)

ASM writes in allocation units (AUs), with a configurable size:

- Range: 1 MB to 64 MB
- Default: 4 MB

AUs are the basic allocation chunks used to build extents and distribute data across disks.

---

## 4. ASM Files and Direct Client I/O (ASM points, clients shoot)

Any file managed inside ASM becomes an ASM file from a metadata perspective.

Model:

- ASM provides placement metadata
- Client performs direct reads/writes using that metadata

Oracle Database clients can write directly to ASM disk groups. Non-database clients usually consume ASM storage through ACFS on ASM volumes.

Common database-related file types in ASM include:

- Data files
- Control files
- RMAN backup files
- SPFILE/password files
- Flashback logs

---

## 5. Naming Conventions and Aliases

Disk group references use `+` notation:

- Example disk group `DATA` is referenced as `+DATA`

ASM-generated names can be verbose, so aliases can be created to simplify navigation and administration.

This is especially useful once directory depth and file count increase.

---

## 6. Extents and Variable Extent Sizing (yes, the chunks get bigger on purpose)

Data is written as extents distributed across disks in a disk group according to striping/redundancy policy.

Variable extent sizing is used to reduce metadata overhead:

- Smaller early extents, then larger effective extent sizes as file growth continues
- If AU is below 4 MB, growth tiers are described as:
1. First ~20,000 extents at base AU size
2. Next ~20,000 extents at 4x AU
3. Beyond ~40,000 extents at 16x AU
- If AU is above 4 MB, extent sizing is handled internally based on ASM allocation behavior

Each extent maps to storage in ASM-managed disks, while file growth keeps extending allocation in policy-driven chunks.

---

## 7. Striping Types

ASM describes two striping patterns:

- Coarse striping:
  - One AU is treated as one stripe unit
- Fine striping:
  - Uses 128 KB stripe units combined into larger allocation structures

Choice affects I/O layout behavior for different file/workload patterns.

---

## 8. Mirroring Precedence and Redundancy Options (disk-group defaults are suggestions, file rules are law)

Mirroring is enforced at the file level. Disk group redundancy sets the default, but file-level settings can override it.

Meaning:

- Two files in the same disk group can have different mirroring policies

Redundancy models referenced in this lesson:

- `EXTERNAL` (no ASM mirroring)
- `NORMAL` (two-way)
- `HIGH` (three-way)
- `FLEX` (starts at two-way, can be policy-driven to higher protection)
- Extended-cluster redundancy options as applicable to topology

Extent placement respects primary/mirror copy distribution through failure groups.

---

## 9. ASM Clients and Visibility Tools (who is touching your storage right now)

Many components can be ASM clients:

- Oracle databases
- Clusterware components (for example, OCR-related use cases)
- ACFS/volume-manager-backed file consumers
- Other utilities using ASM-managed storage

Useful visibility commands/views:

- `asmcmd lsclient -G <diskgroup>`
- `V$ASM_CLIENT` (from SQL*Plus connected to ASM)

These let you see which clients are using a given ASM instance/disk group.

---

## 10. Key Takeaways

- ASM disks are discovered storage devices; disk groups are built from them.
- AU size (default 4 MB) drives allocation behavior, with variable extent growth for scalability.
- ASM provides metadata, clients do direct I/O.
- Striping can be coarse or fine; mirroring defaults come from disk groups but can be overridden per file.
- `asmcmd lsclient` and `V$ASM_CLIENT` are core tools for tracking ASM consumers.
