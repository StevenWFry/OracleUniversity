# Lab 10-1 - Configuring Shared Server Mode

Practice 4-1 configures shared server mode for regular database connections, not just XDB traffic.

---

## 1. Practice Goal

In this lab, you:

- verify current shared server and dispatcher settings
- enable multiple shared servers
- adjust dispatcher config so shared mode can serve general TCP sessions

---

## 2. Set Environment and Connect as SYSDBA

1. Open terminal.
2. Set Oracle environment:

```bash
. oraenv
```

When prompted, enter:

```text
orclcdb
```

3. Connect:

```bash
sqlplus / as sysdba
```

---

## 3. Check Current Shared Server Configuration

In SQL*Plus:

```sql
SHOW PARAMETER shared_servers;
SHOW PARAMETER dispatchers;
```

Transcript interpretation:

- DBCA commonly preconfigures dispatcher/service for Oracle XML DB (`XDB`)
- this can show `shared_servers = 1`
- that setup supports XDB protocols (such as HTTP/FTP), but not automatically all regular SQL client service patterns

---

## 4. Enable Shared Servers

Set three shared servers:

```sql
ALTER SYSTEM SET shared_servers = 3;
```

---

## 5. Reconfigure Dispatcher for General TCP Service

Check current dispatcher value again:

```sql
SHOW PARAMETER dispatchers;
```

If dispatcher is tied to a specific XDB service, reset to a general TCP dispatcher so it can serve broader service connections:

```sql
ALTER SYSTEM SET dispatchers='(PROTOCOL=TCP)';
```

Verify:

```sql
SHOW PARAMETER dispatchers;
```

Expected:

- dispatcher remains on TCP
- no service lock to `orclcdbXDB`/XDB-only target in the value

---

## 6. Exit

```sql
EXIT
```

---

## 7. Lab Result

You enabled shared server mode with `shared_servers = 3` and adjusted dispatchers to support general TCP shared-server connectivity rather than only XDB-scoped sessions.
