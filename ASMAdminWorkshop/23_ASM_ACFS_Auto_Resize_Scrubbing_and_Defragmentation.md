## Lesson 23 - ASM ACFS Auto Resize, Scrubbing, and Defragmentation (in which file systems quietly maintain themselves until they absolutely do not)

And look, this chapter covers enhanced ACFS operations for keeping file systems healthy and performant: automatic growth, integrity scrubbing, and fragmentation cleanup.

By the end of this lesson, you should be able to:

- Configure ACFS automatic resize policies with increment and max limits
- Explain node-level auto-resize behavior in clustered environments
- Run ACFS scrub operations for metadata/data inconsistency checks
- Run ACFS defragmentation at directory or file scope
- Use diagnostic output to estimate and target remediation work

---

## 1. Why These Features Matter

Once ACFS is in production, storage health is not just "is it mounted?"

You need ongoing control for:

- Capacity growth under pressure
- Corruption/inconsistency detection
- Fragmentation management for file-layout efficiency

Because the system that "worked perfectly during deployment" will eventually meet real workloads and bad timing.

---

## 2. ACFS Automatic Resize

ACFS can auto-grow based on policy:

- Growth increment
- Maximum size cap

Command pattern:

```bash
acfsutil size -a <increment> -x <max_size> <mount_point>
```

Example:

```bash
acfsutil size -a 1G -x 10G /acfs/appdata
```

Interpretation:

- Grow in 1 GB steps
- Never exceed 10 GB

---

## 3. Node-Level Auto-Resize Behavior

In clustered operation, auto-resize state can diverge temporarily by node.

From this lesson:

- If auto-resize fails on one node, that node can be temporarily disabled for auto-resize
- Other nodes can continue normally
- If the underlying volume cannot extend (no free space), auto-resize effectively stalls across nodes
- System periodically rechecks capacity and can re-enable when growth succeeds

Manual recovery path can include remount/reconfiguration of increment/max policy where needed.

So auto-resize is resilient, but not magic.

---

## 4. ACFS Scrubbing

Scrubbing checks integrity and reports inconsistencies.

Scope can include:

- File-level
- Directory-level
- Recursive directory traversal

Findings may include:

- Metadata inconsistency indicators
- Data inconsistency location details (path/offset context)

Example patterns:

```bash
acfsutil scrub file /acfs/appdata/path/file1
acfsutil scrub dir /acfs/appdata/path
acfsutil scrub dir -r /acfs/appdata/path
```

Additional options can tune power/throttle behavior in supported command variants.

---

## 5. Defragmentation in ACFS

Fragmentation can affect file layout efficiency, especially in write-heavy and churn-heavy workloads.

ACFS supports defrag operations at:

- Directory scope
- File scope

Example patterns:

```bash
acfsutil defrag dir /acfs/appdata/path
acfsutil defrag dir -r /acfs/appdata/path
acfsutil defrag file /acfs/appdata/path/testfile
```

This can complement (not replace) database-level storage strategy when database files are placed on ACFS.

---

## 6. Estimation and Diagnostics

This lesson highlights using command diagnostics to estimate impacted extents/regions before heavy maintenance runs.

Practical approach:

1. Inspect/report first
2. Estimate scope
3. Run targeted scrub/defrag
4. Re-validate after operation

Because blind "fix everything" runs during peak load are how planned maintenance becomes folklore.

---

## 7. Operational Guidance

For production ACFS maintenance:

1. Set conservative auto-resize policy (increment + max)
2. Monitor volume free space trends
3. Run scrub in controlled windows
4. Use defrag selectively where evidence supports it
5. Capture before/after diagnostics for change records

Keep maintenance boring. Boring is good.

---

## 8. Key Takeaways

- Auto-resize uses policy controls (`-a` increment, `-x` max) and behaves per-node in clustered contexts.
- Scrubbing detects metadata/data inconsistencies and can be scoped to file or directory.
- Defragmentation can run at file or directory scope, including recursive traversal.
- Diagnostics and estimates should drive remediation scope, not guesswork.
- ACFS health is an ongoing operational discipline, not a one-time setup checkbox.
