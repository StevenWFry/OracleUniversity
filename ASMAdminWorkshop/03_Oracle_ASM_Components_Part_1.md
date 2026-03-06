## Lesson 3 - Oracle ASM Components Part 1 (in which storage gets organs, blood flow, and a scheduling department)

And look, this lesson moves from "what ASM is" to "what ASM is made of." We focus on the ASM instance, disk groups, redundancy options, and failure groups, which is where architecture diagrams stop being decorative and start deciding whether your weekend survives.

By the end of this lesson, you should be able to:

- Describe ASM instance memory and process architecture
- Explain how ASM clients connect in single-node and clustered designs
- Summarize Flex ASM behavior and failover routing
- Define ASM disk groups as the base storage unit
- Compare redundancy types and failure-group requirements

---

## 1. ASM Instance Architecture

The ASM instance is built similarly to an Oracle Database instance:

- SGA (shared memory)
- Background processes

But ASM does not serve regular table/index data, so there is no traditional database buffer cache for user blocks. Instead, ASM memory is focused on metadata and ASM-specific cache usage.

High-level idea:

- Database instance = user data + metadata + SQL execution support
- ASM instance = storage metadata + extent placement + disk group management

---

## 2. ASM Background Processes (the storage civil service)

Key processes to know:

- `RBAL`: rebalance coordinator
- `ARBn`: rebalance worker processes (spawned for parallel rebalance)
- `GMON`: monitors and manages disk group activities
- `MARK`: handles marking/recovery of allocation-related metadata states

Operationally, these processes support:

- Client access to ASM metadata
- Disk group state management
- Rebalance execution and coordination

In short, ASM background processes are the invisible bureaucracy keeping storage from turning into a Mad Max reboot.

---

## 3. Client Connectivity Models

ASM client behavior depends on deployment style.

### 3.1 Single node (Oracle Restart)

- One node, one ASM instance (typically local use)
- Local clients connect to that local ASM instance
- If the ASM instance is unavailable, clients on that node lose ASM metadata access until service is restored

### 3.2 Clustered (Flex ASM, aka metadata with backup dancers)

In cluster environments, Flex ASM is the default model:

- Default ASM cardinality is typically 3 instances
- You can run fewer (as low as 1) or more (commonly up to 5, depending on design)
- Clients can connect locally or remotely to an ASM instance based on load and availability

If one ASM instance fails:

- Clients are automatically redirected to another ASM instance
- Clusterware/Flex ASM picks a suitable target based on current overhead/load

If the failed instance returns:

- Client distribution can rebalance back across ASM instances

This is the "no drama, fewer sirens" path for metadata continuity in RAC-style environments.

---

## 4. ASM Disk Group: The Fundamental Storage Container (your storage city limits)

A disk group is the base unit of ASM storage. No disk group, no ASM space allocation.

Disk group characteristics:

- Built from one or more ASM disks
- Self-contained policy domain (striping/mirroring defaults)
- Holds files and directory structures for one or many databases/clients

Think of a disk group like a logical drive, except it has rules, enforcement, and a personality disorder.

Multiple databases can share a disk group safely; ASM separates file namespaces internally.

---

## 5. Redundancy Models in ASM (how paranoid would you like to be?)

The lesson covers these redundancy options:

- `EXTERNAL`: no ASM mirroring (protection expected from external storage, such as RAID)
- `NORMAL`: two-way mirroring
- `HIGH`: three-way mirroring
- `FLEX`: flexible redundancy model (commonly two-way default, configurable per policy/file-group strategy)

Flex redundancy also enables richer controls such as:

- File groups
- Quota management
- More granular file placement/mirroring behavior

So instead of one blunt storage policy for everything, Flex lets you get surgically specific.

---

## 6. Failure Groups (where mirrors go so disaster math still works)

Failure groups define fault-isolation boundaries for mirror copies.

Important rules:

- `NORMAL` redundancy requires at least 2 failure groups
- `HIGH` redundancy requires at least 3 failure groups
- Flex configurations follow the mirror degree used (2-way or 3-way)

You can place multiple disks in one failure group. That is normal.

Design intent:

- Mirror copies should land in different failure groups so one failure domain does not wipe all copies

This is why failure-group design is not cosmetic. It is your "nobody panic on Monday morning" math.

---

## 7. Component Relationships (quick mental map)

Use this chain:

1. ASM instance manages metadata and orchestration
2. Disk groups provide pooled storage
3. Failure groups define mirror fault domains
4. Redundancy policy decides how many copies exist
5. Rebalance processes keep distribution healthy when storage changes

If this map is clear, the rest of ASM administration becomes much less mysterious.

---

## 8. Key Takeaways

- ASM instances look like database instances structurally, but serve storage metadata rather than application data.
- Flex ASM allows client failover to surviving ASM instances when one instance goes down.
- Disk groups are the mandatory storage container in ASM.
- Redundancy is a policy choice: external, normal (2-way), high (3-way), or flex-managed strategy.
- Failure groups are central to safe mirroring and must match your redundancy design.
