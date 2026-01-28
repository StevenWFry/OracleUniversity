# Module 1 - Oracle Database Creation and Administration (Architecture SpeedRun)

And look, before you start creating and administering databases, it helps to remember **what** you're actually creating and administering. Otherwise you're just clicking "Next, Next, Finish" in DBCA and hoping the database fairy knows what she's doing.

This module starts with a high-speed recap of Oracle Database architecture and deployment options, then moves into practical database creation and admin tasks in later chapters.

By the end of this module, you should be able to:

- Describe Oracle Database server architecture at a high level
- Explain multitenant concepts (CDB, root, PDBs)
- Compare single-instance and RAC deployments
- Summarize where sharding fits into scale-out architecture

---

## 1. Two Big Pieces: Database vs Instance

Oracle has two major components that people love to confuse:

- **Database** - the *physical* thing
  - A set of files on disk
  - Control files, redo log files, datafiles
  - Stores all persistent data and metadata

- **Instance** - the *logical* brains
  - Shared memory (SGA)
  - Background processes (DBWn, LGWR, CKPT, SMON, PMON, etc.)
  - Reads from / writes to the database files and serves user sessions

You never talk to the datafiles directly. The server processes:

1. Read data blocks from disk
2. Convert them into a **logical** in-memory structure
3. Place them in shared memory so multiple sessions can reuse them without repeated I/O

Background processes keep the instance and physical database in sync, handling:

- Writing dirty buffers
- Logging changes
- Recovery and cleanup
- Monitoring and housekeeping

On the client side:

- A **user process** runs your app or SQL Developer
- It contacts the **listener**
- The listener hands you off to a **server process** and private memory area in the instance
- That server process does the work on your behalf

---

## 2. Multitenant: CDB, Root, and PDBs

From 12c onwards, Oracle introduced the **multitenant** architecture:

- **Container Database (CDB)** - the top-level database
  - Has the usual control file, redo logs, and datafiles
  - Root container (`CDB$ROOT`) stores **Oracle metadata only**
  - You do *not* put user data in the root

- **Pluggable Databases (PDBs)** - where the actual application data lives
  - Each PDB is a self-contained logical database for an application
  - Created from a **PDB seed** (a template), then customized
  - Has its own tablespaces and datafiles

Benefits:

- Clean separation between applications
- Independent backup/recovery per PDB
- Better security isolation and management

With **application containers** and **proxy PDBs**, you can:

- Group related PDBs into an application container
- Use partitioning and database links to split data further
- Present each partition/PDB to the right application or tenant

---

## 3. SingleInstance vs MultiInstance (RAC)

Deployment choice #1:

- **Single-instance database**
  - One database, one instance, one host
  - Simple, but a single point of failure: if the instance/host dies, so does availability

- **Multi-instance database** (RAC - Real Application Clusters)
  - One database, **multiple instances** on different hosts
  - Hosts form a cluster managed by clusterware
  - Instances coordinate via a private interconnect, maintain row-level locking, and share access to the same database files

RAC gives you:

- High availability - if one instance/host fails, others keep serving
- Scalability - add instances (nodes) for more capacity

The cluster stack handles:

- Heartbeats between nodes
- Instance membership
- Failover and reconfiguration

---

## 4. Sharding: When One Database Isn't Enough

For truly **big** workloads, there's **Oracle Sharding**:

- "Divide and conquer" at the database level
- Data is partitioned into **shards** (often using partitioning logic)
- Each shard is a full Oracle database serving part of the overall data set

Key pieces:

- **Shards** - independent Oracle databases, each holding a subset of data
- **Shard Director** - central routing component; clients connect here, not to individual shards
- Requests are transparently routed to the appropriate shard(s)

You can have up to **48** shard databases in a single sharded configuration, all working together in parallel and mostly invisible to the client.

Partitioning + sharding lets you:

- Slice data by region, tenant, customer, or whatever key makes sense
- Isolate workloads and failures
- Scale horizontally without turning a single database into an unmanageable monster

---

## 5. Where This Module Goes Next

With the architecture refreshed, the rest of this administration module will focus on:

- Creating CDBs and PDBs via tools and scripts
- Configuring and managing instances and their memory/processes
- Managing storage structures (tablespaces, datafiles, undo, temp)
- Handling backup/recovery, high availability, and performance basics

This introductory chapter is the "map" for everything that follows. Next up: actually creating databases instead of just admiring their diagrams.

