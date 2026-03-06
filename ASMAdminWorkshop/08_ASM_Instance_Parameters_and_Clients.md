## Lesson 8 - ASM Instance Parameters and Clients (in which metadata traffic gets a seating chart and rebalance gets a throttle)

This chapter is where Flex ASM stops being a concept and becomes operations: cardinality, client placement, relocation behavior, and the parameters that decide whether your storage layer glides or wheezes.

By the end of this lesson, you should be able to:

- Explain ASM cardinality in Flex ASM and how Clusterware enforces it
- Identify and relocate ASM clients using dynamic performance views
- Distinguish cluster-managed instance count from ASM initialization parameters
- Tune key ASM parameters (`ASM_DISKGROUPS`, `ASM_DISKSTRING`, `ASM_POWER_LIMIT`, `MEMORY_TARGET`, `INSTANCE_TYPE`)
- Choose the right tool (`srvctl`, `asmcmd`, SQL*Plus, listener tools) for each operation

---

## 1. Flex ASM Cardinality: How Many ASM Instances Should Exist

In Flex ASM, instance count is cluster-managed and called **cardinality**.

Core behavior:

- Minimum ASM instances in cluster: 1
- Maximum ASM instances in cluster: 5
- Cluster tries to keep the configured cardinality running automatically

Important distinction:

- Cardinality is **not** an ASM initialization parameter
- It is a Clusterware service-level setting

So this is not "ALTER SYSTEM magic." This is "Clusterware staffing policy."

---

## 2. What Happens When Cardinality Changes

If cardinality is increased and free nodes exist:

- Cluster starts additional ASM instance(s) automatically on nodes without one

If cardinality exceeds available nodes:

- Cluster runs as many as possible
- No dramatic error festival just because requested count > node count

Example outcome:

- Cardinality set to 4
- Four-node cluster with capacity available
- You end up with four ASM instances running and managed

---

## 3. ASM Clients: Visibility and Placement

Client visibility tools:

- `GV$ASM_CLIENT` for cluster-wide client-to-ASM mapping
- `ASMCMD`/ASMCA views for quick operational checks

Why `GV$` matters:

- It is cluster-wide (grid view), not local-instance-only visibility
- You can see which database/client is attached to which ASM instance

Because before moving clients, you should know where they actually are. Revolutionary concept.

---

## 4. Client Relocation: Automatic and Manual

Automatic relocation happens when:

- ASM instance fails
- New ASM instance is added
- Load balancing policies kick in

Manual relocation is available using SQL:

```sql
ALTER SYSTEM RELOCATE CLIENT '<client_name>'
  FROM INSTANCE '<asm_inst_from>'
  TO INSTANCE '<asm_inst_to>';
```

Use manual relocation when you want controlled drain behavior before maintenance.

Operational note from this lesson:

- If you manually move clients away, cluster does not necessarily "undo" your decision immediately even with load balancing enabled
- This gives you a stable window to shut down the source ASM instance cleanly

---

## 5. Large-Cluster Picture

A large cluster can run:

- Many database instances
- Fewer ASM instances
- Optional ADVM proxy components for ACFS paths

Clients may connect:

- Locally to ASM on same host
- Remotely to ASM on other nodes

Flex ASM spreads metadata service load without requiring per-node ASM everywhere, which is exactly what keeps big clusters from becoming an administrative petting zoo.

---

## 6. ASM Initialization Parameters: What Actually Changes Behavior

### 6.1 `INSTANCE_TYPE`

- Mandatory identity for ASM instance role
- Value: `ASM`

### 6.2 `ASM_DISKGROUPS`

- Controls disk groups to mount at startup
- In this lesson's model, null/default means broad automatic mounting behavior
- Critical groups (OCR/SPFILE-related) must be available for cluster stability

### 6.3 `ASM_DISKSTRING`

- Controls disk discovery path patterns
- Supports explicit paths and wildcards (`?`, `*`, platform-dependent forms)
- Null/default means broad discovery behavior based on platform/default paths

### 6.4 `ASM_POWER_LIMIT`

- Rebalance power dial
- Range: `0` to `1024`
- Default: `1`
- `0` disables automatic rebalance work
- Higher value -> more parallel rebalance workers -> faster movement, higher resource pressure

### 6.5 `MEMORY_TARGET`

- Total ASM memory budget (shared + process-related private needs)
- Default baseline commonly around `1G` in this note set
- ASM usually needs less memory than database instances, but tune for metadata load and environment scale

---

## 7. SPFILE vs PFILE in ASM

Parameter source can be:

- SPFILE (preferred for cluster-managed operations)
- PFILE (manual edits and manual lifecycle discipline)

Cluster best practice:

- Keep ASM SPFILE on shared storage
- Avoid per-node drift and "why is node 3 different?" mysteries

`ALTER SYSTEM` can target all ASM instances or specific ones with `SID` scoping when needed.

---

## 8. Disk Group Mounting and Operational Dependencies

Some disk groups are optional in daily workload flow.
Some are very much not optional.

Groups carrying critical assets must be up for cluster health, including examples like:

- OCR-related files
- ASM/shared SPFILE dependencies
- Core cluster service metadata paths

If required groups are not mounted, ASM may start but cannot serve clients meaningfully, and then your cluster starts making choices you will not enjoy.

---

## 9. Listener and Service Control

ASM listener lifecycle can be managed with listener tooling (`lsnrctl`) and observed from OS contexts where privileges permit.

But in clustered deployments, prefer `srvctl`:

- Action execution + Clusterware registration happen together
- State consistency is immediate
- Fewer split-brain-of-the-admin-tool moments

Example mindset:

- Local tool for local quick checks
- `srvctl` for authoritative clustered state transitions

---

## 10. Key Takeaways

- Cardinality is a Clusterware setting, not an ASM init parameter.
- Flex ASM can run with fewer ASM instances than nodes while still serving all clients.
- `GV$ASM_CLIENT` is your source of truth before manual client relocation.
- `ASM_POWER_LIMIT` is the rebalance speed-vs-resource dial.
- Prefer shared SPFILE and `srvctl`-driven control for cluster consistency.
