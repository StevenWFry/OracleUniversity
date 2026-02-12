# Lab 11-5 - Configuring the Oracle Database Server for Session Multiplexing

Practice 5-5 is the final lab in this module: enable multiplexing on the database side and verify CMAN is funneling multiple client sessions through shared dispatcher/circuit paths.

---

## 1. Practice Goal

Enable session multiplexing for Oracle Connection Manager and validate behavior with dynamic performance views.

---

## 2. Enable Multiplexing in Dispatcher Configuration

On the database host:

1. Set environment and connect as SYSDBA:

```bash
. oraenv
```

When prompted, enter:

```text
orclcdb
```

Then:

```bash
sqlplus / as sysdba
```

2. Set dispatcher attributes to include multiplexing:

```sql
ALTER SYSTEM SET dispatchers='(PROTOCOL=TCP)(MULTIPLEX=ON)';
```

3. Keep this SYSDBA session available for validation queries.

---

## 3. Baseline Session View (Before CMAN Client Logins)

In a SYSDBA session, run the practice query against `v$session` to inspect
server mode and client program details (as provided in your guide).

Typical outcome before CMAN test sessions:

- existing sessions may show `DEDICATED` server mode.

---

## 4. Start Client Sessions Through CMAN

In client terminals (`oraenv` set to `client`), connect using CMAN-routed alias:

```bash
sqlplus system/cloud_4U@C_ORCLCDB
```

Open two separate client sessions this way.

---

## 5. Recheck Session View

Back in SYSDBA session, rerun the `v$session` query.

Expected pattern from transcript:

- CMAN-routed sessions appear with non-dedicated/shared routing behavior.
- additional session rows appear as more client sessions connect.

---

## 6. Validate Multiplexing with `v$circuit`

Run:

```sql
SELECT saddr, circuit, dispatcher, server, SUBSTR(queue,1,8)
FROM   v$circuit;
```

Expected:

- only multiplexed CMAN-routed sessions are shown
- multiple client sessions may map through the same dispatcher

---

## 7. Cleanup

1. Exit all SQL*Plus sessions (DB and client terminals).
2. Shut down Connection Manager:

```bash
cmctl
```

At `CMCTL>`:

```text
ADMINISTER CMAN_LOCALHOST
SHUTDOWN
EXIT
```

---

## 8. Lab Result

You enabled dispatcher multiplexing, validated multiplexed session/circuit behavior through `v$session` and `v$circuit`, then cleanly shut down CMAN.
