# Lab 35-2 - Transporting a Tablespace Across Platforms (Simulated on One Host)

Practice 3-2 demonstrates transportable tablespace workflow with RMAN/Data Pump
metadata integration. In training, source and target are on the same host, but
the steps mirror cross-platform procedure.

---

## 1. Practice Goal

Transport `BARTBS` from `ORCLPDB1` to `ORCLPDB2` using RMAN transport workflow,
including:

- source tablespace read-only window
- backup with `TO PLATFORM` + Data Pump metadata export
- foreign tablespace restore into destination PDB

Assumptions:

- terminal open as Oracle OS user
- practice indicates when to point to `ORCLPDB1` vs `ORCLPDB2`

---

## 2. Prepare Source Objects

Run setup script (creates tablespace/user/table/data and logs setup output):

```bash
transtablespace.sh
```

---

## 3. Verify Prerequisites in SQL*Plus

1. Connect as SYS:

```bash
sqlplus / as sysdba
```

2. Verify source database mode:

```sql
SELECT open_mode FROM v$database;
-- expected: READ WRITE
```

3. Identify target platform string exactly (case matters):

```sql
COLUMN platform_name FORMAT A40
SELECT platform_id, platform_name, endian_format
FROM v$transportable_platform
WHERE UPPER(platform_name) LIKE '%LINUX%';
```

Training target used:

- `Linux x86 64-bit`

4. Set container and make source tablespace read-only:

```sql
ALTER SESSION SET CONTAINER=orclpdb1;
ALTER TABLESPACE bartbs READ ONLY;
EXIT;
```

---

## 4. Backup Source Tablespace with RMAN (`TO PLATFORM`)

1. Start RMAN and connect target:

```bash
rman target sys/cloud_4U@orclpdb1
```

2. Run backup with platform conversion + Data Pump metadata export:

```rman
BACKUP TO PLATFORM 'Linux x86 64-bit'
  TABLESPACE bartbs
  FORMAT '/tmp/bartbs_%U'
  DATAPUMP FORMAT '/tmp/bartbs_exp_%U.dmp';
```

3. Re-enable source write activity after backup:

```rman
SQL "ALTER TABLESPACE bartbs READ WRITE";
EXIT;
```

Note:

- In real multi-host transport, copy backup sets and dump files to target host.
- In this training environment, source/target share one host, so copy step is
  effectively skipped.

---

## 5. Prepare Destination PDB (`ORCLPDB2`)

1. Set environment for target CDB/PDB as needed.
2. Connect and set destination container:

```bash
sqlplus / as sysdba
```

```sql
ALTER SESSION SET CONTAINER=orclpdb2;
```

3. Create destination user and grant session privilege:

```sql
CREATE USER bar IDENTIFIED BY cloud_4U;
GRANT CREATE SESSION TO bar;
EXIT;
```

---

## 6. Restore Foreign Tablespace into Destination

1. Connect RMAN to destination:

```bash
rman target sys/cloud_4U@orclpdb2
```

2. Restore foreign tablespace and metadata:

```rman
RESTORE FOREIGN TABLESPACE bartbs
  FORMAT '/u01/app/oracle/oradata/ORCLCDB/ORCLPDB2/%U.dbf'
  FROM BACKUPSET '/tmp/bartbs_*'
  DUMP FILE FROM BACKUPSET '/tmp/bartbs_exp_*.dmp';
```

(Use exact generated backup/dump names in your environment.)

3. Verify in destination:

```rman
SQL "SELECT tablespace_name FROM dba_tablespaces WHERE tablespace_name='BARTBS'";
EXIT;
```

Expected:

- `BARTBS` exists in destination database.

---

## 7. Cleanup

Run cleanup script (drops original/transported artifacts and removes generated
backup/dump files):

```bash
transtablespace_cleanup.sh
```

---

## 8. Lab Result

You completed a full transportable tablespace workflow:

- source prepared and exported in read-only window
- RMAN backup created with platform conversion metadata
- destination restored using foreign tablespace semantics
- `BARTBS` verified in `ORCLPDB2`
- environment cleaned