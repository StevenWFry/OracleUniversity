## Lesson 1 - Course Overview (in which backups, logs, and RMAN enter the chat wearing hard hats)

Welcome to Oracle Database 19c: Backup and Recovery, hosted by Barbara Waddoups, senior principal core instructor at Oracle University. If you thought this course might be about taking screenshots of your database and saving them to the cloud, adjust your expectations accordingly.

By the end of this lesson, you should be able to:

- Describe the scope of backup and recovery in Oracle
- Identify key components like redo logs and archive logs
- Explain the role of RMAN and the recovery catalog
- Recognize recovery options (complete, point-in-time, block)
- Understand where flashback and duplication fit in

---

## Course Table of Contents

### Foundations and Configuration

- [Lesson 2 - Backup and Recovery Overview](02_Backup_and_Recovery_Overview.md)
- [Lesson 3 - Understanding Instance Recovery](03_Understanding_Instance_Recovery.md)
- [Lesson 4 - Backup and Recovery Configuration](04_Backup_and_Recovery_Configuration.md)
- [Lesson 5 - Backup and Recovery Configuration Demos and Practices](05_Backup_and_Recovery_Configuration_Demos_and_Practices.md)
- [Lesson 6 - Backup and Recovery Configuration Practices](06_Backup_and_Recovery_Configuration_Practices.md)

### RMAN and Backup Operations

- [Lesson 7 - Using Recovery Manager](07_Using_Recovery_Manager.md)
- [Lesson 8 - Using Recovery Manager Demos and Practices](08_Using_Recovery_Manager_Demos_and_Practices.md)
- [Lesson 9 - Backup Strategies](09_Backup_Strategies.md)
- [Lesson 10 - Creating Database Backups](10_Creating_Database_Backups.md)
- [Lesson 11 - Tuning RMAN Backup and Recovery Performance](11_Tuning_RMAN_Backup_and_Recovery_Performance.md)

### Recovery Catalog

- [Lesson 12 - Recovery Catalog Overview](12_Recovery_Catalog_Overview.md)
- [Lesson 13 - Creating a Recovery Catalog](13_Creating_a_Recovery_Catalog.md)
- [Lesson 14 - Managing Target Database Records in the Recovery Catalog](14_Managing_Target_Database_Records_in_the_Recovery_Catalog.md)
- [Lesson 15 - Using Stored Scripts in the Recovery Catalog](15_Using_Stored_Scripts_in_the_Recovery_Catalog.md)
- [Lesson 16 - Creating and Using Virtual Private Catalogs](16_Creating_and_Using_Virtual_Private_Catalogs.md)

### Restore and Recovery

- [Lesson 17 - Understanding Restore and Recovery Technology](17_Understanding_Restore_and_Recovery_Technology.md)
- [Lesson 18 - Diagnosing Failures](18_Diagnosing_Failures.md)
- [Lesson 19 - Data Recovery Advisor](19_Data_Recovery_Advisor.md)
- [Lesson 20 - Performing Complete Recovery](20_Performing_Complete_Recovery.md)
- [Lesson 21 - Performing Point in Time Recovery](21_Performing_Point_in_Time_Recovery.md)
- [Lesson 22 - Performing Incomplete Recovery Using the Data Recovery Advisor](22_Performing_Incomplete_Recovery_Using_the_Data_Recovery_Advisor.md)
- [Lesson 23 - Performing Block Media Recovery](23_Performing_Block_Media_Recovery.md)
- [Lesson 24 - Performing Additional Recovery Operations](24_Performing_Additional_Recovery_Operations.md)

### Flashback and Time-Based Recovery

- [Lesson 25 - Oracle Flashback Technology Overview](25_Oracle_Flashback_Technology_Overview.md)
- [Lesson 26 - Using Logical Flashback Features](26_Using_Logical_Flashback_Features.md)
- [Lesson 27 - Using Flashback Drop and Temporal Features](27_Using_Flashback_Drop_and_Temporal_Features.md)
- [Lesson 28 - Using Flashback Database](28_Using_Flashback_Database.md)
- [Lesson 29 - Using PDB Snapshots](29_Using_PDB_Snapshots.md)

### Duplication and Cloning

- [Lesson 30 - Database Duplication Overview](30_Database_Duplication_Overview.md)
- [Lesson 31 - Creating a Backup Based Duplicate Database](31_Creating_a_Backup_Based_Duplicate_Database.md)

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
