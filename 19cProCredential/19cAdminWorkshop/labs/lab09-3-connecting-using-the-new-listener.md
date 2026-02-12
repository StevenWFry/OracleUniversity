# Lab 9-3 - Connecting to a Database Service Using the New Listener

Practice 3-3 is the payoff: confirm `LISTENER2` actually works by connecting through its non-default port.

---

## 1. Practice Goal

Use Easy Connect syntax to connect to a CDB service through `LISTENER2` on port `1561`.

---

## 2. Connect with Easy Connect

1. Open a new terminal.
2. Connect with SQL*Plus using host, port, and service:

```bash
sqlplus system/cloud_4U@localhost:1561/orclcdb
```

Expected:

- Successful login through `LISTENER2`.

Notes:

- If your environment includes a domain, service may be fully qualified.
- Domain comes from `DB_DOMAIN`; it may be blank depending on deployment.

---

## 3. Exit

In SQL*Plus:

```sql
EXIT
```

Close terminal window.

---

## 4. Lab Result

You verified that the new listener endpoint (`TCP/1561`) accepts client connections to the CDB service using Easy Connect syntax.
