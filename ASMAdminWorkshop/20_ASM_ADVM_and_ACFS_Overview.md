## Lesson 20 - ASM ADVM and ACFS Overview (in which raw storage becomes a civilized file system with a lot of background staff)

This chapter introduces the ADVM + ACFS layer on top of ASM disk groups, which is how you expose ASM-backed storage like a regular file system for non-database consumers (and, technically, database files too, though with caveats).

By the end of this lesson, you should be able to:

- Explain how ADVM exposes ASM disk group space as usable volumes
- Describe how ACFS sits on top of ADVM volumes for file-system access
- Identify key background processes supporting ADVM/ACFS operations
- Understand mount/dismount and privilege expectations for ACFS workflows
- Describe stripe-column allocation behavior in ADVM volume space

---

## 1. Big Picture: Disk Group -> ADVM Volume -> ACFS

Architecture chain:

1. ASM disk group provides managed storage pool
2. ADVM (ASM Dynamic Volume Manager) carves volume space from that pool
3. File system (ACFS or supported alternative) sits on top of that volume
4. Clients access it like regular file-system storage

Use cases:

- Middleware/app files
- Images/text/other unstructured files
- Shared file-system needs in clustered environments

You can place database files there, but native ASM file handling is usually preferred for database workloads to avoid extra layers.

---

## 2. What ADVM Actually Does

ADVM is the translator between ASM-managed extents and OS-visible block-device behavior.

It handles:

- Volume creation and sizing
- Block-device presentation
- Metadata coordination with ASM

Once volume devices exist, ACFS (or supported FS layer) can mount and serve files to clients.

So ADVM is the adapter that lets "database storage engine" speak "file system."

---

## 3. ACFS Capabilities in This Context

ACFS is a general-purpose file system for single-node or clustered usage.

Capabilities referenced in this lesson context include operational features like:

- Snapshots
- Replication-oriented workflows (environment/version dependent)
- HA NFS integrations
- Security features (including encryption/auditing paths)
- Utility-driven management via `acfsutil`

Feature availability can vary by release/platform policy, but the architecture intent is stable: managed, scalable file services on ASM-backed storage.

---

## 4. Core Background Processes Mentioned

ADVM/ACFS operation relies on ASM-side process support, including roles described in this lesson:

- Volume driver background:
  - General volume administration tasks
- Volume background:
  - Volume availability/open-close lifecycle coordination
- Volume membership background:
  - Cluster/instance-state continuity for volume operations
- Proxy/communication background:
  - Coordination between ASM instance and volume-manager proxy paths

You do not need to memorize every acronym, but you do need to remember this is not "just a mount point." There is a lot happening behind the curtain.

---

## 5. Privileges and Mount Operations

In Linux/Unix-style environments:

- Mount/dismount actions usually require root-level privilege
- ASMCA can assist and surface commands, but privileged execution still matters

In practice:

- Storage/admin roles coordinate ADVM+ACFS setup
- OS privilege model governs final mount exposure

So yes, this is where database admins and sysadmins have to communicate like adults.

---

## 6. Volume Space and Stripe Columns

Inside ADVM volume space, allocation uses stripe-column behavior.

Concepts highlighted in this lesson:

- Stripe column count can range from 1 to 8
- Modern defaults commonly use 8 columns
- Stripe-column size defaults are often around 1 MB
- Writes are distributed round-robin across stripe columns

This round-robin allocation keeps distribution balanced and reduces concentration hotspots over time.

---

## 7. Write Distribution Example (Round-Robin)

With 8 stripe columns, incoming write chunks are placed:

1. Column 1
2. Column 2
3. ...
8. Column 8
9. Back to column 1

Subsequent files continue from current allocation state, not from scratch, so balance remains smooth across ongoing workload.

This is why volume-level distribution can look surprisingly even even under mixed write bursts.

---

## 8. Redundancy Context for ADVM/ACFS

Redundancy behavior still depends on underlying disk group policy:

- External redundancy means no ASM-side mirroring
- Mirrored disk groups preserve ASM redundancy semantics beneath ADVM/ACFS

Either way, ADVM allocation logic still aims to balance writes across available volume space.

---

## 9. Operational Guidance

When planning ADVM/ACFS rollout:

1. Validate disk group purpose/capacity first
2. Create and size ADVM volumes deliberately
3. Mount ACFS with correct OS privilege model
4. Baseline I/O and metadata behavior before production cutover
5. Use snapshots/management utilities as part of lifecycle, not as emergency-only tools

Because "we mounted it and hoped" is not a storage strategy.

---

## 10. Key Takeaways

- ADVM turns ASM storage into volume devices that file systems can consume.
- ACFS provides general-purpose file-system access on top of ADVM.
- Background processes coordinate volume state, metadata, and cluster continuity.
- Mount/dismount workflows require proper OS privilege handling.
- Stripe-column round-robin allocation is central to balanced ADVM write distribution.
