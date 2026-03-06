## Lesson 10 - Creating Database Backups (in which RMAN gives you options, and each option has consequences)

And look, this lesson is where backup strategy becomes actual commands: full and partial backups, incremental workflows, image copies, and the famous "this seemed simple until I had to restore it" moment.

By the end of this lesson, you should be able to:

- Create whole and partial backups with RMAN
- Back up CDB root, selected PDBs, and targeted tablespaces
- Distinguish backup sets from image copies in practical terms
- Configure control file/SPFILE autobackup behavior
- Understand incrementally updated image copy workflows

---

## 1. RMAN Setup and Core Flow

Before running RMAN, source your environment and confirm SID context.

Example:

```bash
export ORACLE_SID=cdb1
rman target " / as sysbackup"
```

Core backup command patterns:

```rman
BACKUP DATABASE;
BACKUP DATABASE PLUS ARCHIVELOG;
```

If you include `PLUS ARCHIVELOG`, you preserve more complete recovery options. If you skip it, recovery flexibility gets worse later, usually at the exact moment you need it most.

---

## 2. Whole vs Partial Backup Scope (CDB/PDB aware)

### Whole database backup (multitenant)

```rman
BACKUP DATABASE PLUS ARCHIVELOG;
```

This captures:

- CDB root
- all included PDB data files in scope
- archived redo logs (because you asked for them)
- control file and SPFILE (with proper autobackup/config behavior)

### Recover whole database

```rman
RECOVER DATABASE;
```

---

## 3. Backup Specific PDBs or Objects

You can target just selected pluggable databases:

```rman
BACKUP PLUGGABLE DATABASE sales_pdb, hr_pdb;
BACKUP PLUGGABLE DATABASE sales_pdb PLUS ARCHIVELOG;
```

You can also target specific tablespaces using PDB-qualified syntax:

```rman
BACKUP TABLESPACE sales_pdb:tbs2;
BACKUP TABLESPACE hr_pdb:system, sales_pdb:sysaux;
```

If you reference an unqualified tablespace (for example `sysaux`), RMAN interprets it in current/root context unless explicitly qualified.

---

## 4. Backup Set vs Image Copy

### Backup set (default RMAN style)

Compressed/proprietary RMAN format, smaller footprint:

```rman
BACKUP AS BACKUPSET TABLESPACE hr_data
  FORMAT '/u01/backup/%U';
```

Note: `AS BACKUPSET` is often implicit default, but stating it can make intent obvious in scripts.

### Image copy (`AS COPY`)

Bit-for-bit file copy:

```rman
BACKUP AS COPY DATAFILE '/u01/app/oracle/oradata/ORCL/users_01_db01.dbf';
BACKUP AS COPY ARCHIVELOG LIKE 'arch%';
```

Image copies are bigger and slower to produce, but they can make restore/switch operations faster and simpler.

---

## 5. Control File and SPFILE Protection

Recommended persistent setting:

```rman
CONFIGURE CONTROLFILE AUTOBACKUP ON;
```

With this enabled, RMAN automatically captures current control file and SPFILE metadata in backup workflows. This is exactly the kind of boring safeguard that saves you from very expensive improvisation later.

---

## 6. Full vs Incremental Refresher

### Full backup

- All used blocks from data files.

### Incremental level 0

- Base equivalent used for incremental strategy lineage.

### Incremental level 1 cumulative

- Changes since last level 0.

### Incremental level 1 differential

- Changes since last incremental backup.

Cumulative is usually larger. Differential is usually smaller and more frequent-friendly.

---

## 7. Incrementally Updated Image Copy (rolling-forward strategy)

This workflow keeps an image copy progressively current using level 1 incrementals.

Example command pair:

```rman
RECOVER COPY OF DATABASE WITH TAG 'DAILY_INC';
BACKUP INCREMENTAL LEVEL 1 FOR RECOVER OF COPY WITH TAG 'DAILY_INC' DATABASE;
```

How it behaves:

1. Day 1: no prior copy -> RMAN creates baseline image copy.
2. Day 2: no prior incremental to apply -> RMAN produces first level 1 incremental.
3. Day 3 onward: RMAN applies previous incremental to roll image copy forward, then takes next incremental.

Result: you maintain a near-current image copy plus incremental chain.

This is one of those strategies that feels magical when tested properly and catastrophic when never rehearsed.

---

## 8. Practical Strategy Notes

1. Always pair data backup design with archive-log availability.
2. Use tags consistently (`DAILY_INC`, etc.) for operational clarity.
3. Choose backup set vs image copy based on restore-time objectives, not habit.
4. For multitenant systems, always be explicit about scope (root vs PDB).
5. Validate restore paths regularly, not just backup completion.

---

## 9. Wrap-Up (before fast incrementals)

You now have the command-level map for creating database backups with RMAN across:

- whole database scope
- partial PDB/tablespace scope
- backup set and image copy formats
- incremental roll-forward image copy workflow

Next up in the course flow: fast incremental backups and how to make incremental strategy move like it has somewhere to be.
