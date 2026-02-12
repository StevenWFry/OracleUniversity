# Lab 16-1 - Managing Space in Tablespaces (Part 1)

Practice 3-1 begins with tablespace threshold management and alert generation,
then prepares data growth so warning levels are triggered.

---

## 1. Practice Goal

In this part, you will:

- set warning/critical thresholds for tablespace usage
- create and populate a test tablespace (`TBSALERT`)
- verify threshold values in metadata views
- trigger and observe server-generated outstanding alerts

This lab sequence supports Segment Advisor and later space-usage remediation.

---

## 2. Connect and Set Baseline Thresholds

1. Open terminal and set environment:

```bash
. oraenv
```

When prompted, enter:

```text
orclcdb
```

2. Connect to `ORCLPDB1` as `system`:

```bash
sqlplus system/cloud_4U@orclpdb1
```

3. Reset database-wide thresholds with `DBMS_SERVER_ALERT.SET_THRESHOLD`.

Lab values shown:

- warning: `85`
- critical: `97`

Transcript note:

- run command as one line or use proper continuation formatting.

---

## 3. Verify Default Thresholds from Root

Switch to root container and query `DBA_THRESHOLDS`:

```sql
ALTER SESSION SET CONTAINER = CDB$ROOT;

SELECT warning_value, critical_value
FROM   dba_thresholds
WHERE  metrics_name = 'Tablespace Space Usage';
```

Expected:

- warning `85`
- critical `97`

---

## 4. Create `TBSALERT` in `ORCLPDB1`

Switch back to `ORCLPDB1`:

```sql
ALTER SESSION SET CONTAINER = ORCLPDB1;
```

Create tablespace via provided script:

```sql
@/home/oracle/labs/DBMod_Storage/createTBSAlertTS.sql
```

Requirements from guide:

- file size `120M`
- locally managed
- automatic segment space management
- not autoextensible
- no per-tablespace thresholds at create time

Check free space:

```sql
@/home/oracle/labs/DBMod_Storage/tbsalertfreespace.sql
```

Expected initial state:

- around `99%` free

---

## 5. Set `TBSALERT`-Specific Thresholds

Run `DBMS_SERVER_ALERT.SET_THRESHOLD` again for `TBSALERT` with:

- warning: `55`
- critical: `70`

Then verify:

```sql
SELECT warning_value, critical_value
FROM   dba_thresholds
WHERE  object_name = 'TBSALERT';
```

Expected:

- warning `55`
- critical `70`

Check alert history message:

```sql
SELECT reason, suggested_action
FROM   dba_alert_history
WHERE  object_name = 'TBSALERT'
ORDER  BY sequence_id DESC;
```

Expected note:

- threshold-update informational entry

---

## 6. Run Segment Advisor Setup Script

Exit SQL*Plus and run:

```bash
/home/oracle/labs/DBMod_Storage/segment_advisor_setup.sh
```

Script behavior:

- creates/populates tables in `TBSALERT`
- increases used space toward threshold trigger point

---

## 7. Validate Space Usage and Wait for Alert

Reconnect:

```bash
sqlplus system/cloud_4U@orclpdb1
```

Check fullness (target around 60% used):

```sql
@/home/oracle/labs/DBMod_Storage/tbsalertsize.sql
```

Check free bytes:

```sql
SET ECHO ON
@/home/oracle/labs/DBMod_Storage/tbsalertfreespace.sql
```

Expected:

- about `39%` free (varies slightly)

Query outstanding alerts:

```sql
SELECT reason
FROM   dba_outstanding_alerts
WHERE  object_name = 'TBSALERT';
```

If no rows appear immediately:

- wait and retry (can take up to ~10 minutes)

Expected when populated:

- warning indicating tablespace is about `60%` full

---

## 8. Part 1 Stopping Point

Part 1 ends at the wait/retry stage for `DBA_OUTSTANDING_ALERTS` population,
right before continuing to the next segment of Practice 3-1.
