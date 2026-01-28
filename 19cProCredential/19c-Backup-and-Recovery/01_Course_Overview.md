## Lesson 1 - Course Overview (in which backups, logs, and RMAN enter the chat wearing hard hats)

Welcome to Oracle Database 19c: Backup and Recovery, hosted by Barbara Waddoups, senior principal core instructor at Oracle University. If you thought this course might be about taking screenshots of your database and saving them to the cloud, adjust your expectations accordingly.

By the end of this lesson, you should be able to:

- Describe the scope of backup and recovery in Oracle
- Identify key components like redo logs and archive logs
- Explain the role of RMAN and the recovery catalog
- Recognize recovery options (complete, point-in-time, block)
- Understand where flashback and duplication fit in

---

## 1. What This Course Covers (the survival kit)

This course is about keeping databases alive, or at least resuscitating them when they faint. You will learn:

- Database architecture basics that matter for recovery
- How redo and archive logs keep your data from being a ghost story
- Failure types and how to recover from each
- Database configurations that support recovery
- The backup storage area and why it is not optional
- Multiplexing redo and archive logs for resilience

---

## 2. RMAN (your polite but relentless backup robot)

You will use Recovery Manager (RMAN) to:

- Connect to the database and configure backup/recovery settings
- Define retention policies
- Create and manage different backup types
- Query backup information in RMAN and at the system level

RMAN is not just a backup tool. It is also your audit trail, your recovery engine, and the reason your weekend might stay intact.

---

## 3. Recovery Catalog (the memory palace for backups)

You will learn:

- What a recovery catalog is and how to create it
- How RMAN uses it to store backup metadata
- How it supports multiple target databases
- How to store RMAN scripts for automation

Think of it as a backup librarian with a clipboard and a good memory.

---

## 4. Restore and Recovery Options (pick your adventure)

You will cover:

- Complete recovery vs. point-in-time recovery
- Block-level recovery for surgical fixes
- Data Recovery Advisor to diagnose failures
- When and how to recover based on failure type

Because not all disasters are created equal.

---

## 5. Flashback, PDB Snapshots, and Duplication (time travel, but for data)

You will learn:

- Flashback technology for logical failures
- Flashback to a time, SCN, log sequence, or restore point
- PDB snapshots for quick rollback
- Database duplication using RMAN DUPLICATE

This is the part of the course where you stop fearing mistakes and start scheduling them.

---

## 6. Target Audience (the cast of characters)

This course is for:

- Developers who are responsible for their own dev databases
- Backup and recovery specialists handling enterprise systems
- Database administrators who need reliable recovery plans

If your job involves data loss panic, you belong here.

---

## 7. Prerequisites (what you should know before the chaos)

You should already understand:

- Oracle database architecture basics
- Tablespaces (system vs user-defined)
- Control files and how many you should have
- SPFILEs and how to back them up
- Archive log mode and what to do if you are not in it
- Storage layout for data files and backups

In short: know your database anatomy before you try to perform CPR.

---

## 8. Learning Outcomes (the post-course powers)

After completing this course, you will be able to:

- Configure database and RMAN settings for recovery
- Choose the right recovery level for the situation
- Execute RMAN backup and recovery commands
- Manage backups and retention policies
- Use the recovery catalog for long-term metadata and scripts
- Use flashback to fix logical failures

---

## 9. What Comes Next (the certification ladder)

Next steps include:

- Oracle Database 19c: Admin Professional Credential Learning Path
- Exam 1Z0-083 for administration certification
- Deploy Patch and Upgrade Workshop
- Multitenant course
- Optional: Data Guard course for higher availability and certification prep

The course ends, but the resume line begins.

---

## 10. Wrap-Up (welcome to the fun part)

You now know what this course will cover, who it is for, and why RMAN is about to be your new best friend. Time to dive in and start saving databases from their worst decisions.
