# Lab 12-1 - Cloning Remote PDBs in Hot Mode (Part 1)

Practice for Lesson 2: using additional PDB creation techniques, starting with
hot cloning preparation from a production-like source PDB.

---

## 1. Practice Context

In this practice sequence, you will:

- clone a regular PDB in hot mode
- create refreshable clone behavior (manual/automatic refresh)
- later perform relocation operations

Scenario:

- `PDB_SOURCE_FOR_HOTCLONE` has performance issues in production context.
- You need a hot clone (`PDB_HOTCLONE`) in `CDBTEST` for testing, while source remains online.

Assumptions noted:

- `ORCLCDB` contains at least `ORCLPDB1` and `ORCLPDB2`
- instance/listener may need startup via `dbstart.sh`

---

## 2. Verify Environment and Prep

If DB/listener are already running, skip startup.

Otherwise:

```bash
cd /home/oracle/labs
./dbstart.sh
```

---

## 3. Cleanup Existing Practice PDBs

Open terminal and go to lab directory:

```bash
cd /home/oracle/labs/DBMod_PDBs
```

Run cleanup script:

```bash
./cleanup_PDBs.sh
```

Expected:

- script drops prior practice PDBs in `ORCLCDB`
- if objects do not exist, informational errors may appear

---

## 4. Apply SQL*Plus Formatting Script

Run:

```bash
/home/oracle/labs/DBMod_PDBs/glogin_4.sh
```

Purpose:

- sets display formatting for query output used in this lab sequence

---

## 5. Run Hot Clone Setup Script

Execute:

```bash
./setup_hotclone.sh
```

When prompted, provide SYS password.

Script actions (as described in transcript):

- creates `PDB_SOURCE_FOR_HOTCLONE` from seed in `ORCLCDB`
- creates `CDBTEST`
- creates local user `SOURCE_USER` in `PDB_SOURCE_FOR_HOTCLONE`
- creates `SOURCE_USER.BIGTAB`
- inserts ~10,000 rows into `BIGTAB`
- performs supporting file copy/setup operations

Runtime note:

- this step takes time; wait for completion before moving to step 5 of the guide.

---

## 6. Part 1 Stopping Point

Part 1 ends once `setup_hotclone.sh` completes and environment is ready for the
next task in the hot clone workflow.
