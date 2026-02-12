# Lab 35-1 - Move Data from One PDB to Another with Data Pump

Practice 3-1 moves schema objects from `ORCLPDB1` to `ORCLPDB2` for test
comparison (because nothing says fun like moving objects and then discovering one
missing tablespace at the end).

---

## 1. Practice Goal

Scenario:

- Source: `OE` schema in `ORCLPDB1`
- Target: import into `ORCLPDB2` as `OETEST`

You will:

1. export `OE` from `ORCLPDB1`
2. generate SQL from the dump for validation
3. import into `ORCLPDB2`
4. resolve import failures caused by missing target dependencies

Assumption:

- logged in as `oracle` OS user

---

## 2. Setup and Export from `ORCLPDB1`

1. Open terminal and set environment:

```bash
. oraenv
# accept default ORACLE_SID
```

2. Run setup script:

```bash
DP_setup.sh
```

3. Attempt schema export (expected to fail first if path is used directly):

```bash
expdp oe/cloud_4U@orclpdb1 schemas=oe \
  dumpfile=/u01/app/oracle/admin/orclcdb/dpdump/expoe.dmp
```

Expected failure:

- `ORA-39088: file name cannot contain a path specification`

Reason:

- Data Pump requires a database `DIRECTORY` object, not raw OS path in
  `DUMPFILE`.

4. Create and grant directory in `ORCLPDB1`:

```sql
sqlplus system/cloud_4U@orclpdb1
CREATE DIRECTORY dp_for_oe AS '/u01/app/oracle/admin/orclcdb/dpdump';
GRANT READ, WRITE ON DIRECTORY dp_for_oe TO oe;
EXIT;
```

5. Re-run export correctly:

```bash
expdp oe/cloud_4U@orclpdb1 schemas=oe \
  directory=dp_for_oe dumpfile=expoe.dmp
```

Expected:

- export job completes successfully

---

## 3. Validate What Was Exported (Generate SQL)

Generate SQL DDL from the dump:

```bash
impdp oe/cloud_4U@orclpdb1 \
  directory=dp_for_oe dumpfile=expoe.dmp sqlfile=oe_SQL.sql
```

Review:

```bash
cat /u01/app/oracle/admin/orclcdb/dpdump/oe_SQL.sql
```

This confirms object definitions (tables, constraints, indexes, sequences, etc.)
were included in export metadata.

---

## 4. Import into `ORCLPDB2` as `OETEST`

1. Prep target and drop leftover user if present:

```sql
sqlplus system/cloud_4U@orclpdb2
DROP USER oetest CASCADE;
-- ignore error if user does not exist
EXIT;
```

2. Attempt import (first try may fail if directory was only created in
   `ORCLPDB1`):

```bash
impdp system/cloud_4U@orclpdb2 \
  directory=dp_for_oe dumpfile=expoe.dmp \
  remap_schema=oe:oetest
```

Expected possible error:

- invalid directory object in `ORCLPDB2`

Why:

- directory objects are local to each PDB. Creating it in `ORCLPDB1` does not
  magically create it in `ORCLPDB2`.

3. Create directory in `ORCLPDB2` and retry:

```sql
sqlplus system/cloud_4U@orclpdb2
CREATE DIRECTORY dp_for_oe AS '/u01/app/oracle/admin/orclcdb/dpdump';
EXIT;
```

Re-run import command above.

---

## 5. Handle Partial Import Failure (Missing Tablespace)

Observed in transcript:

- import partially succeeds
- objects dependent on missing `TBS_APP2` fail (for example `ORDER_ITEMS` and
  related objects)

Create missing tablespace in target:

```sql
sqlplus system/cloud_4U@orclpdb2
CREATE TABLESPACE tbs_app2 DATAFILE SIZE 100M;
EXIT;
```

At this point, rerun import to load previously failed objects.

---

## 6. Retry Import After Creating `TBS_APP2`

Rerun the same import:

```bash
impdp system/cloud_4U@orclpdb2 \
  directory=dp_for_oe dumpfile=expoe.dmp \
  remap_schema=oe:oetest
```

Expected behavior:

- import completes with some "object already exists" style messages
- these are normal here, because many objects were already imported in the first
  partial run

So yes, errors appear. No, this is not the end of civilization.

---

## 7. Verify `OETEST` in `ORCLPDB2`

Connect as `OETEST` and validate objects:

```sql
sqlplus oetest/cloud_4U@orclpdb2
SELECT table_name FROM user_tables;
-- expected: ORDERS, ORDER_ITEMS

SELECT COUNT(*) FROM order_items;
-- expected: 665

SELECT index_name FROM user_indexes;
-- expected: 2 indexes (one system-named, one explicitly named)

SELECT sequence_name FROM user_sequences;
-- expected: ORDER_SEQ
EXIT;
```

---

## 8. One-Step Alternative: Import via Database Link (No Dump File Round Trip)

If you want to import directly from `ORCLPDB1` into `ORCLPDB2`:

1. Create DB link in `ORCLPDB2`:

```sql
sqlplus system/cloud_4U@orclpdb2
CREATE DATABASE LINK link_orclpdb1
  CONNECT TO oe IDENTIFIED BY cloud_4U
  USING 'orclpdb1';
```

2. Reset target user:

```sql
DROP USER oetest CASCADE;
EXIT;
```

3. Run `impdp` using `NETWORK_LINK`:

```bash
impdp system/cloud_4U@orclpdb2 \
  schemas=oe remap_schema=oe:oetest \
  network_link=link_orclpdb1
```

4. Re-verify:

```sql
sqlplus oetest/cloud_4U@orclpdb2
SELECT table_name FROM user_tables;
SELECT COUNT(*) FROM order_items;
-- expected: 665
EXIT;
```

---

## 9. Advantages and Drawbacks of `NETWORK_LINK`

Advantage:

- no intermediate dump file handling

Drawback:

- over an unencrypted link, data travels in clear text even if encrypted at rest
  in the source database

---

## 10. Lab Result

You completed two valid migration patterns:

- dump-file based export/import from `ORCLPDB1` to `ORCLPDB2`
- direct import over database link using `NETWORK_LINK`

And you validated the target schema (`OETEST`) with expected objects, indexes,
sequence, and row counts.
