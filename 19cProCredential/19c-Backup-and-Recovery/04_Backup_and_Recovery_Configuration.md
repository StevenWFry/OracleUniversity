## Lesson 4 - Backup and Recovery Configuration (in which the FRA becomes your pantry and you stop storing all your eggs in one control file)

This lesson is about getting your environment ready for real recovery: multiplexing, archivelog mode, and sizing the Fast Recovery Area so it does not explode mid-backup.

By the end of this lesson, you should be able to:

- Plan for maximum recoverability based on backup frequency
- Configure the Fast Recovery Area (FRA) size and location
- Monitor FRA usage and reclaimable space
- Multiplex control files and redo logs safely
- Configure archivelog mode and archive destinations

---

## 1. Maximum Recoverability (backup frequency is destiny)

Your recovery point is only as good as your last backup. If you back up rarely, your recovery window gets ugly fast. Decide how often you can back up based on:

- Acceptable data loss
- Backup duration
- Storage and retention policy

---

## 2. Fast Recovery Area (FRA) (the storage bunker)

The FRA stores:

- Archived redo logs
- Backup files
- Flashback logs
- At least one control file (recommended)
- Potentially a redo log member

Key parameters:

- `DB_RECOVERY_FILE_DEST` (location)
- `DB_RECOVERY_FILE_DEST_SIZE` (size)

Size it for today’s backups plus growth. If the FRA fills up during backup, the database can stall. That is not the vibe.

---

## 3. FRA Monitoring (because full disks do not recover themselves)

Track usage and reclaimable space to avoid outages:

- Reclaimable space includes older archive logs and backups
- Retention policies determine when space can be reused

Primary view:

- `V$RECOVERY_FILE_DEST`

You can also monitor via Enterprise Manager Cloud Control.

---

## 4. Multiplexing Control Files (single points of failure are rude)

Control files are required at startup and track every file in the database. You should have:

- At least two control files (standard)
- Separate disks/controllers

If one control file fails:

1. Shutdown the database (abort if necessary)
2. Copy a good control file to the failed location
3. Restart the database

If you use ASM, Oracle manages the copies for you, but you still need to know where they live.

---

## 5. Multiplexing Redo Logs (write once, not once-and-pray)

Redo logs should have:

- At least two members per group
- Members on different disks
- A reasonable number of members per group (too many slows writes)

To add a member from SQL:

```sql
ALTER DATABASE ADD LOGFILE MEMBER
  '/u01/app/oracle/oradata/ORCL/redo1A.log'
  TO GROUP 1;
```

This mirrors changes across members so a single disk failure does not stop your database.

---

## 6. Archivelog Mode (the difference between “recover” and “good luck”)

Archivelog mode:

- Copies full redo logs to archive logs
- Enables recovery to the last committed transaction
- Allows backups while the database is open

To enable it:

1. Shutdown the database
2. Start up in mount mode
3. `ALTER DATABASE ARCHIVELOG;`
4. `ALTER DATABASE OPEN;`

Archive destinations:

- `LOG_ARCHIVE_DEST`
- `LOG_ARCHIVE_DEST_n` for multiple locations

FRA is the default destination, but you can add more for redundancy.

---

## 7. Wrap-Up (configuration is recovery insurance)

You now know how to size and monitor the FRA, multiplex control files and redo logs, and enable archivelog mode with multiple archive destinations. These are the boring configuration steps that save you when the exciting failures happen.
