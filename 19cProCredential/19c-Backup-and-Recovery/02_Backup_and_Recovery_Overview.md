## Lesson 2 - Backup and Recovery Overview (in which the DBA becomes part medic, part detective, part meteorologist)

Welcome to the part where "backup and recovery" stops being a slogan and becomes your actual responsibility. This lesson explains what you do, what can break, and how to fix it without turning your company into a case study.

By the end of this lesson, you should be able to:

- Define DBA responsibilities for backup and recovery
- Explain MTTR and MTBF and why they control your sleep schedule
- Identify administrative privileges for backup work
- Classify common failure types and response options
- Distinguish complete vs incomplete recovery

---

## 1. Your DBA Responsibilities (the job description they forgot to email)

Backup and recovery starts with knowing:

- Where failures occur
- How to prevent or mitigate them
- What recovery level your business expects

You are responsible for the plan, the tools, and the aftermath.

---

## 2. The Two Clocks: MTTR and MTBF (time is your enemy)

You should always optimize for:

- **Mean Time to Recovery (MTTR):** time from failure to full recovery
- **Mean Time Between Failures (MTBF):** time between incidents

Goal: shorten MTTR and stretch MTBF as far as physics allows.

---

## 3. SYSDBA vs SYSBACKUP (privileges with boundaries)

Traditionally, backup work uses `SYSDBA`. But Oracle gives you a narrower, safer option:

- **SYSBACKUP** lets a user perform backup and recovery
- Works on open or closed databases
- Does **not** allow selecting arbitrary user tables
- The `SYSBACKUP` user exists by default, but should not be used directly

Why? Auditing. If the only thing you see is `SYSBACKUP`, you do not know who actually did the work.

---

## 4. Connecting to RMAN with SYSBACKUP (the quoting trick)

You can connect to RMAN using SYSBACKUP with a literal string:

```text
rman target " / as sysbackup"
```

The quotes are required because the slash needs to be treated as a literal. Once connected, RMAN shows the DBID by default.

---

## 5. What Recovery Your Business Requires (aka your SLA reality check)

Ask these questions:

- How real-time must recovery be?
- What is the tolerance for data loss?
- What is the tolerated downtime?
- Do SLAs require specific recovery objectives?

Point-in-time recovery might be mandatory, but it also means data loss. Decide up front how much loss is acceptable.

---

## 6. Physical, Logical, and Human Disasters (a buffet of bad days)

Not all failures look the same:

- Physical disasters (hurricanes, tornadoes) require off-site recovery
- Logical errors (bad code, bad data) require precision fixes
- Human mistakes (drops, deletes) require fast reversal

Your recovery plan must cover all three or it is not a plan, it is a wish.

---

## 7. Failure Types and Typical Responses (triage mode)

Common failure categories and how you respond:

- **Statement failure:** fix privileges or training
- **Process failure:** ensure resumable space allocation and adjust quotas
- **User failure:** instance cleans up abnormal disconnects automatically
- **Network failure:** use backup listeners and connect-time failover
- **Instance failure:** automatic crash recovery on startup
- **Media failure:** replace disks/controllers, relocate files, restore

There is no universal fix. There is only correct diagnosis.

---

## 8. User Errors and Fast Recovery Options (oops management)

For human mistakes:

- Flashback can rewind tables or transactions
- Recycle bin can recover dropped tables (if enabled)
- Otherwise, restore from backup and recover forward

Flashback is the difference between "fixed in five minutes" and "see you next maintenance window."

---

## 9. LogMiner and Redo (the forensic toolkit)

Oracle LogMiner lets you:

- Inspect redo and archived redo logs
- Diagnose what actually happened
- Use Enterprise Manager or SQL for analysis

Logs are your source of truth. Treat them like evidence.

---

## 10. Instance, Media, and Data Failures (the harder problems)

Instance failures:

- Power outages and crashes trigger automatic recovery
- Use alert logs, trace files, or Enterprise Manager to diagnose

Media failures:

- Disk, controller, filesystem, or storage network issues
- May require moving data, replacing hardware, or redefining storage

Data failures:

- Missing data files at OS level
- Physical corruption (checksum, block header, size mismatch)
- Logical corruption (inconsistent metadata or control files)
- I/O failures from OS limits or unavailable channels

These are the failures that separate "backup exists" from "recovery succeeds."

---

## 11. Wrap-Up (break time, then instance recovery)

You now understand DBA responsibilities, failure categories, and the difference between complete and incomplete recovery. After the break, we move into instance recovery, where the database tries to fix itself before you have to.
