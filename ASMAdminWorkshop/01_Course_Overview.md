## Lesson 1 - Course Overview (in which storage reliability gets the full investigative-news treatment)

And look, welcome to the ASM survival guide: the part of Oracle where disks look calm, dashboards look green, and yet everyone still flinches when someone says "maintenance window."

By the end of this lesson, you should be able to:

- Identify who these notes are for and what baseline skills you need
- Summarize the ASM 23ai upgrades that actually matter in real life
- Recognize deprecated/unsupported ACFS features you should avoid
- Understand the progression from ASM basics to Flex ASM and ACFS
- Place ASM in the larger HA chain: Clusterware -> ASM -> RAC -> Data Guard

---

## 1. Who This Is For

Primary audience:

- Database administrators
- Application developers
- IT infrastructure professionals responsible for availability

If you've ever been paged at 2:13 AM because "storage is weird," congratulations, this is your people.

---

## 2. Prerequisites

You should already be comfortable with:

- Core Oracle database administration
- Basic cluster administration concepts for HA environments

Why:

- ASM is tightly integrated with Grid Infrastructure / Clusterware
- It runs on a single node (Oracle Restart) or multi-node clusters
- In both cases, ASM behavior directly affects uptime, failover, and recoverability

---

## 3. ASM 23ai Highlights Worth Caring About

Key improvements called out:

- Extent-based scrubbing for lower-level and more efficient issue handling
- Better recovery handling for damaged data discovered during rebalance
- Online ASM metadata patching
- Better read continuity during rebalance operations (add/remove disks and related storage changes)
- Faster redundancy rebuild after disk failure events

Translation: fewer "all hands" incidents and less dramatic storage therapy.

---

## 4. Deprecated / Unsupported Items

Items specifically called out as unsupported/deprecated:

- ACFS compression
- Snapshot remastering
- `acfsutil` replication reverse command

Do not design new systems around features that are already halfway out the door.

---

## 5. What You Should Be Able To Do

After working through this material, you should be able to:

- Explain ASM architecture and operational foundations
- Manage ASM instances and core operations
- Administer disk groups, files, directories, and templates
- Use ACFS for non-database shared storage use cases
- Handle advanced ASM/ACFS topics, including clone-related workflows

---

## 6. Roadmap

Progression through the note set:

1. ASM architecture and core concepts
2. ASM instance administration
3. Disk group administration fundamentals and operations
4. Flex ASM capabilities for resilient, scalable metadata access
5. Directory/file/template management in ASM
6. ACFS for shared file-system consumers
7. Advanced topics including mirrored-data-based point-in-time clone patterns

Think of it as going from "what is ASM" to "how to keep it from becoming tomorrow's incident report."

---

## 7. What Comes Next

Natural follow-on path:

1. RAC Administration
2. Data Guard for deeper HA/DR coverage

ASM is the storage spine. RAC and Data Guard are what you add when one server's bad day must not become everyone's bad week.

---

## 8. Key Takeaways

- ASM is for people responsible for availability, not just storage aesthetics.
- You need solid Oracle admin fundamentals before touching advanced ASM behavior.
- 23ai improvements focus on resilience during rebalance, recovery, and patching.
- Flex ASM and ACFS expand what you can manage and who can consume that storage.
- The full HA story continues with RAC and Data Guard.
