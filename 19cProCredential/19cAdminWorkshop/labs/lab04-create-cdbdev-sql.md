# Lab 4 - Creating `CDBDEV` with Raw SQL (No DBCA Safety Net)

And look, this is the part of the course where we politely take DBCA away and see whether you can still build a database without panicking. The good news is that the process is entirely predictable once you know the steps.

In this lab you create a **container database** called `CDBDEV` using the `CREATE DATABASE` SQL command, then verify that Oracle built what you asked for.

---

## 1. Lab Goal and Characteristics

You will create a new CDB named `CDBDEV` with these characteristics:

- Same passwords for `SYS` and `SYSTEM` as in the base CDB:

  ```text
  cloud_4U
  ```

- Oracle Managed Files (OMF) for datafiles and redo logs:

  ```text
  /u01/app/oracle/oradata
  ```

- CDB root contains:
  - Default TEMP tablespace: `TEMP`
  - Default permanent tablespace: `USERS`
  - Undo tablespace: `UNDOTBS1`

- EM Express HTTPS port target: `5501`
- CDB is created with no PDBs except the seed

---

## 2. Edit the `CREATE DATABASE` Script

1. Change to the lab directory:

   ```bash
   cd /home/oracle/labs/DBMod_CreateDB
   ```

2. Edit the script:

   ```bash
   gedit CrCDBDEV.sql &
   ```

3. Replace any placeholder passwords with the lab password:

   ```text
   cloud_4U
   ```

---

## 3. Prepare the `initCDBDEV.ora` PFILE

1. Set the SID and environment:

   ```bash
   . oraenv
   # When prompted:
   ORACLE_SID = [CDBTEST] ? CDBDEV
   ```

2. Copy the sample init file:

   ```bash
   cd $ORACLE_HOME/dbs
   cp init.ora initCDBDEV.ora
   ```

3. Edit `initCDBDEV.ora` and add these parameters:

   ```ini
   DB_CREATE_FILE_DEST='/u01/app/oracle/oradata'
   ENABLE_PLUGGABLE_DATABASE=true
   ```

4. Update the following parameters to match the lab:

   ```ini
   db_name='CDBDEV'
   audit_file_dest='/u01/app/oracle/admin/CDBDEV/adump'
   db_recovery_file_dest='/u01/app/oracle/fast_recovery_area'
   diagnostic_dest='/u01/app/oracle'
   dispatchers='(PROTOCOL=TCP) (SERVICE=CDBDEVXDB)'
   control_files=('/u01/app/oracle/oradata/ora_control01.ctl',
                  '/u01/app/oracle/fast_recovery_area/ora_control02.ctl')
   compatible='19.0.0.0'
   ```

   Note: Replace any `<ORACLE_BASE>` placeholders with `/u01/app/oracle`.

---

## 4. Verify Required Directories

```bash
mkdir -p /u01/app/oracle/admin/CDBDEV/adump
ls /u01/app/oracle/fast_recovery_area
ls /u01/app/oracle/oradata
```

---

## 5. Create the Database

1. Start SQL*Plus:

   ```bash
   sqlplus / as sysdba
   ```

2. Start the instance in `NOMOUNT` with your PFILE:

   ```sql
   STARTUP NOMOUNT PFILE='$ORACLE_HOME/dbs/initCDBDEV.ora';
   ```

3. Run the `CREATE DATABASE` script:

   ```sql
   @/home/oracle/labs/DBMod_CreateDB/CrCDBDEV.sql
   ```

4. If you hit errors:

   ```sql
   SHUTDOWN ABORT;
   ```

   Fix typos and restart from step 2.

5. Run the dictionary scripts:

   ```sql
   @?/rdbms/admin/catalog.sql
   @?/rdbms/admin/catproc.sql
   ```

6. Exit SQL*Plus:

   ```sql
   EXIT;
   ```

---

## 6. Add `CDBDEV` to `/etc/oratab`

```bash
echo "CDBDEV:/u01/app/oracle/product/19.3.0/dbhome_1:N" | sudo tee -a /etc/oratab
```

Verify:

```bash
cat /etc/oratab
```

---

## 7. Verify Database Characteristics

1. Set the environment and connect:

   ```bash
   . oraenv
   # When prompted:
   ORACLE_SID = [CDBDEV] ? CDBDEV

   sqlplus / as sysdba
   ```

2. Confirm CDB status:

   ```sql
   SELECT cdb FROM v$database;
   ```

3. Check datafile locations (OMF means long names and GUIDs for seed files):

   ```sql
   SET pagesize 100
   COLUMN name FORMAT A130
   SELECT name FROM v$datafile ORDER BY 1;
   ```

4. Confirm root tablespaces:

   ```sql
   SELECT tablespace_name, contents FROM dba_tablespaces;
   ```

5. Check EM Express port (manually created databases default to 0):

   ```sql
   SELECT dbms_xdb_config.gethttpsport() FROM dual;
   ```

6. Exit SQL*Plus:

   ```sql
   EXIT;
   ```

---

## 8. Clean Up (per the Activity Guide)

When instructed, remove `CDBDEV`:

```bash
/home/oracle/labs/DBMod_CreateDB/dropCDBDEV.sh
```

---

## 9. What You Just Survived

By the end of this lab you:

- Created `CDBDEV` using raw SQL
- Built a PFILE with the required parameters
- Ran `catalog.sql` and `catproc.sql` to finish setup
- Verified OMF datafile locations and core tablespaces
- Confirmed EM Express port behavior
- Cleaned up with `dropCDBDEV.sh`

In other words, you now know exactly what DBCA does on your behalf. Which is why DBCA is about to feel a lot more respectable.
