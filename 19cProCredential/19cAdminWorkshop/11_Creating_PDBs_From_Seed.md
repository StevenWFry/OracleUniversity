# 11 - Creating PDBs From Seed (Because Oracle Brought a Starter Kit, and You Are Using It)

This starts the module on creating and administering pluggable databases
(`PDBs`). Before creating one from seed, you need the architecture straight:
what is shared in the CDB, what is isolated in each PDB, and why services and
undo mode matter.

---

## 1. CDB/PDB Architecture Refresher

A container database (`CDB`) includes shared infrastructure plus one or more
pluggable databases (`PDBs`).

Shared at CDB level:

- control files
- redo logs
- core dictionary/system metadata
- single instance background process framework

PDB-level isolation:

- application/user data stays inside each PDB
- direct cross-PDB data access is blocked by design
- cross-PDB data access requires explicit mechanisms (for example DB links)

---

## 2. What Shared Files Mean in Practice

Control file:

- tracks structural metadata for CDB and all PDBs
- each PDB has identity metadata used for management/diagnostics

Redo/Archive:

- redo stream is shared across PDB activity
- change records still identify source context internally
- archive logs therefore also contain mixed CDB/PDB activity

Security/visibility rule:

- connect to PDB service -> access that PDB's data
- connect to root/CDB context -> metadata visibility, not unrestricted user data across PDBs

---

## 3. Local Undo vs Shared Undo

Key evolution:

- older multitenant behavior relied on CDB-level shared undo
- modern recommended configuration is local undo for PDB autonomy

Why local undo matters:

- better PDB independence
- enables cleaner PDB-scoped operations (flashback/clone/backup workflows)
- reduces coupling to root undo behavior

Bottom line:

- local undo is the operational default you generally want.

---

## 4. The PDB Seed (`PDB$SEED`)

`PDB$SEED` is mandatory in a CDB:

- created as part of CDB creation
- kept open read-only by design
- used as template for creating new empty PDBs

You cannot drop seed just because you "don't plan to use it."

---

## 5. PDB Creation Methods Mentioned

You can create PDBs by:

- creating from seed (template-based empty PDB)
- cloning an existing PDB (data + metadata copy)
- SQL-driven creation with explicit file placement
- converting/migrating from non-CDB paths into PDB workflows

Important constraint:

- non-CDB does not become a CDB directly; CDB is created as a container shell, then workloads are migrated/plugged appropriately.

---

## 6. Temp, Password, Wallet, and Service Notes

Temporary usage:

- generally prefer local temporary structures per PDB over shared temp behavior.

Password/auth artifacts:

- instance-level password file behavior is CDB/instance scoped.

Wallet/TDE context:

- wallet strategy may be shared or separated by design depending on release and policy.

Client connectivity:

- applications should connect using PDB service names, not instance/SID if they need PDB data.
- Oracle auto-creates a service for each PDB name; additional services can be added for workload control.

---

## 7. Why Consolidation Helps (and Still Needs Capacity Planning)

Multitenant reduces operational footprint:

- fewer instances to manage
- consolidated background process overhead
- centralized administration model

But consolidation is not free:

- one undersized instance cannot safely absorb many old standalone workloads
- CPU, memory, and IO must be sized for combined load

---

## 8. App Container / Proxy Direction (High-Level)

Transcript also highlights broader patterns:

- application containers can partition application data footprints
- proxy/remote PDB patterns can expose distributed data footprints as one logical model
- global deployments can place data closer to users while preserving unified access semantics

Theme:

- distributed scale through partitioned PDB placement ("divide and conquer" for data and workload locality).

---

## 9. What Comes Next

With architecture covered, the next practical step is creating PDBs from
`PDB$SEED` and validating file placement, service registration, and open-mode
behavior, before anyone says "it worked on my laptop" about a database cluster.
