# Lab 16-2 - Using Compression Advisor for Indexes

Practice 3-2 evaluates index compression levels using `DBMS_COMPRESSION`,
applies the best option, verifies space savings, and then reverts.

---

## 1. Practice Goal

You will:

- create baseline uncompressed index state
- compare `ADVANCED LOW` vs `ADVANCED HIGH` recommendations
- rebuild index with best compression level
- verify reduced block usage
- revert index and verify original footprint

Assumption:

- logged in as `oracle` on compute node

---

## 2. Setup Baseline Index

Run setup script:

```bash
/home/oracle/labs/DBMod_Storage/setupindex.sh
```

Expected:

- index `ITEST` created on `HR.TEST`
- compression initially disabled

---

## 3. Connect and Verify Initial State

Connect as HR to `ORCLPDB1`:

```bash
sqlplus hr/cloud_4U@orclpdb1
```

Check index compression level:

```sql
SELECT index_name, compression, prefix_length
FROM   user_indexes
WHERE  index_name = 'ITEST';
```

Expected:

- compression disabled

Check index segment blocks:

```sql
SELECT segment_name, blocks
FROM   user_segments
WHERE  segment_name = 'ITEST';
```

Transcript baseline:

- `1152` blocks

Exit SQL*Plus (terminal stays open):

```sql
EXIT
```

---

## 4. Review Compression Package Script (Reference)

Inspect predefined package script (as directed in lab):

```bash
cat $ORACLE_HOME/rdbms/admin/dbmscomp.sql
```

---

## 5. Run Compression Advisor - Advanced Low

Reconnect as HR:

```bash
sqlplus hr/cloud_4U@orclpdb1
```

Run advisor script:

```sql
@/home/oracle/labs/DBMod_Storage/compressionindexlow.sql
```

Transcript result:

- estimated compressed size around `809` blocks
- ratio around `1`

---

## 6. Run Compression Advisor - Advanced High

Run:

```sql
@/home/oracle/labs/DBMod_Storage/compressionindexhigh.sql
```

Transcript result:

- estimated compressed size around `130` blocks
- ratio around `8`

Conclusion:

- `ADVANCED HIGH` is clearly better in this dataset.

---

## 7. Rebuild Index with Advanced High Compression

```sql
ALTER INDEX itest REBUILD COMPRESS ADVANCED HIGH;
```

Verify compression setting:

```sql
SELECT index_name, compression, prefix_length
FROM   user_indexes
WHERE  index_name = 'ITEST';
```

Expected:

- compression `ADVANCED HIGH`

Verify new space usage:

```sql
SELECT segment_name, blocks
FROM   user_segments
WHERE  segment_name = 'ITEST';
```

Transcript result:

- about `256` blocks

---

## 8. Revert to Initial (Uncompressed) State

Rebuild uncompressed:

```sql
ALTER INDEX itest REBUILD NOCOMPRESS;
```

Recheck block usage:

```sql
SELECT segment_name, blocks
FROM   user_segments
WHERE  segment_name = 'ITEST';
```

Expected:

- returns to baseline footprint (transcript: `1152` blocks)

Exit SQL*Plus:

```sql
EXIT
```

---

## 9. Lab Result

You used Compression Advisor to compare index compression levels, applied
`ADVANCED HIGH` for significant reduction, validated results, and confirmed
reversibility by restoring uncompressed index size.
