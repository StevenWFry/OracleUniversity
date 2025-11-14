# Standard Operating Procedure: Backup and Recovery (RMAN)

## Purpose
Ensure reliable database backups and point-in-time recovery using RMAN with FRA.

## Steps
1. Verify FRA space:
   ```sql
   SELECT SPACE_USED/1024/1024/1024 AS USED_GB FROM V$RECOVERY_FILE_DEST;
   ```
2. Run backup:
   ```bash
   /u01/app/oracle/admin/scripts/backup_rman.sh
   ```
3. Validate:
   ```bash
   rman target / <<EOF
   VALIDATE DATABASE;
   EOF
   ```
