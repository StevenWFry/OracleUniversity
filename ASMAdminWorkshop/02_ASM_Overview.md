## Lesson 2 - ASM Overview (in which storage gets a traffic controller and everyone is calmer)

Welcome to the part of Oracle storage where "just mount a disk" evolves into a full municipal government with zoning laws, safety codes, and a clipboard named ASM.

This lesson sets the stage for how ASM sits between clients and storage, why that matters, and what components you need to understand before touching production disk groups like a caffeine-powered goblin.

By the end of this lesson, you should be able to:

- Explain the high-level ASM architecture
- Describe how ASM presents storage metadata to clients
- Summarize key ASM benefits (striping, redundancy, rebalance, cluster awareness)
- Identify the installation order and Grid Infrastructure dependency
- List the core ASM components covered next

---

## 1. What ASM Changes

Historically, clients reached storage through the operating system file system (or a third-party volume manager). With ASM, storage is mapped into ASM first, and clients consume that ASM-managed storage layout.

Classic model:

- Client -> OS file system / volume manager -> storage

ASM model:

- Client -> ASM metadata layer -> storage

The important twist: ASM provides metadata and placement logic, while clients perform direct I/O using that metadata.

---

## 2. Is ASM in the Data Path?

Conceptually, ASM is in the middle. Operationally, ASM does not rewrite every block for the client.

ASM responsibilities:

- Tracks disk group metadata
- Knows where extents live
- Applies striping and redundancy policies
- Coordinates rebalance and disk membership

Client responsibilities:

- Read/write data directly to ASM-managed storage using ASM metadata

So to the client, it still feels like normal storage access, just with smarter placement and management behind the curtain.

---

## 3. Oracle Database vs Other ASM Clients

Oracle Database is ASM-native, so it can use ASM storage directly with no extra translation layer.

For other client file use cases (software homes, logs, generic files, images, app files), the lesson highlights using file system layers on top of ASM volumes, especially:

- Oracle ACFS (ASM Cluster File System)

ACFS gives file-system semantics while still benefiting from ASM-managed storage underneath.

---

## 4. Core ASM Benefits (the part your pager actually cares about)

### 4.1 File-level striping

- Striping decisions are managed at ASM file/extents level, not just at a generic logical volume layer.

### 4.2 Redundancy and mirroring

- Disk groups can define default redundancy.
- Individual files can use their own redundancy settings and override defaults when needed.

### 4.3 Automatic rebalancing

- When disks are added, removed, or resized, ASM rebalances extents in the background.
- Rebalance power/speed is tunable based on available resources and urgency.

### 4.4 Cluster-aware operation

- ASM is fully cluster-aware and designed to support shared storage use in clustered deployments.
- OCR and other cluster-related files can be stored in ASM with ASM-managed mirroring.

### 4.5 Lower admin friction

- Disk lifecycle events (offline/online/add/remove) are centrally managed, reducing manual storage babysitting.

---

## 5. Grid Infrastructure and Installation Order (no, you cannot install this backward and hope)

ASM comes with Grid Infrastructure installation.

High-level deployment order:

1. Configure raw/shared disk paths at OS level
2. Install Grid Infrastructure
3. During install, map candidate disks for ASM
4. Create/configure ASM storage (disk groups)
5. Install Oracle Database software after GI/ASM

This ordering matters because database creation tools expect ASM/Grid components to already exist.

---

## 6. Single Node vs Clustered Management

The same stack can be used in:

- Single-node mode (Oracle Restart + ASM)
- Multi-node cluster mode (Clusterware + ASM)

Clusterware can manage:

- ASM instance startup/shutdown
- Disk group mount/dismount
- ASM volume enable/disable
- Database instance lifecycle in clustered environments

---

## 7. ASM Building Blocks (preview of tomorrow's storage bureaucracy)

Next up, these components get their own deep dive:

- ASM instance
- Disk groups (base storage container)
- ASM disks
- Failure groups and mirroring behavior
- Allocation units (AUs)
- ASM files and striping layout

Think of it as:

- Disk group = your ASM "drive"
- Disks = capacity providers
- AUs/extents = allocation granularity
- Files = objects ASM distributes and protects according to policy

---

## 8. Key Takeaways

- ASM centralizes storage metadata and policy, while clients keep direct I/O.
- Oracle Database is ASM-native; other file clients can use ACFS on ASM volumes.
- Striping, redundancy, and automatic rebalance are the operational core.
- Grid Infrastructure is installed first; database software follows.
- Next step is component-level detail: instances, disk groups, failgroups, disks, AUs, and files.
