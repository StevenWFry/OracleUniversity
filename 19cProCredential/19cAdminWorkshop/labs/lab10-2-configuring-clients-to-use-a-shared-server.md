# Lab 10-2 - Configuring Clients to Use a Shared Server

Practice 4-2 configures a client-side net service that explicitly requests a shared server connection, then validates it from the database side.

---

## 1. Practice Goal

Create a `tnsnames.ora` entry (`test_ss`) that uses shared server mode, connect through it, and verify shared circuits are active.

---

## 2. Set Environment

1. Open terminal.
2. Set Oracle environment:

```bash
. oraenv
```

When prompted, enter:

```text
orclcdb
```

---

## 3. Add Shared-Server TNS Entry

1. Go to network admin directory:

```bash
cd $ORACLE_HOME/network/admin
pwd
```

2. Back up `tnsnames.ora`:

```bash
cp tnsnames.ora tnsnames.ora.4-2
ls
```

3. Edit file:

```bash
gedit tnsnames.ora &
```

4. Copy an existing CDB entry and add new alias `test_ss`, with shared server connect data.

Example shape:

```text
test_ss =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = your-host.example.com)(PORT = 1521))
    (CONNECT_DATA =
      (SERVICE_NAME = orclcdb)
      (SERVER = SHARED)
    )
  )
```

Note from transcript:

- service must point to a valid existing DB service (for example `orclcdb`).
- a typo in service/password was corrected during demo.

Save and close.

---

## 4. Connect Through `test_ss`

Use SQL*Plus with the new alias:

```bash
sqlplus system@test_ss
```

Password:

```text
cloud_4U
```

Keep this session open.

---

## 5. Verify Shared Server Circuit in Second Session

1. Open second terminal.
2. Set environment and connect as SYSDBA:

```bash
. oraenv
sqlplus / as sysdba
```

3. Query shared server circuits:

```sql
SELECT dispatcher, server, saddr, queue
FROM   v$circuit;
```

Expected while `test_ss` session is connected:

- one or more rows showing dispatcher/server mappings.

---

## 6. Disconnect Client and Recheck

1. In first terminal (`system@test_ss`) session:

```sql
EXIT
```

2. In SYSDBA session, rerun:

```sql
SELECT dispatcher, server, saddr, queue
FROM   v$circuit;
```

Expected after disconnect:

- `no rows selected` (as shown in transcript for this lab flow).

---

## 7. Exit and Close

In SYSDBA session:

```sql
EXIT
```

Close both terminals.

---

## 8. Lab Result

You configured a client alias (`test_ss`) for shared server access, connected successfully, and verified connection lifecycle behavior via `v$circuit`.
