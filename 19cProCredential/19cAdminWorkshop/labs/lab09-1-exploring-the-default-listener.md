# Lab 9-1 - Exploring the Default Listener

Welcome to Practice 3-1, where we investigate Oracle's default listener and confirm dynamic registration is actually doing its job instead of just existing in theory.

---

## 1. Practice Goal

Explore the default listener (`LISTENER`) and validate dynamic service registration behavior.

From the practice overview:

- Databases/listeners are not auto-started with VM boot.
- Use `dbstart.sh` when needed.
- Check running processes with:
 - `pgrep -lf smon` (databases)
 - `pgrep -lf tns` (listeners)

---

## 2. Set Environment and Connect as SYSDBA

1. Open terminal.
2. Set environment:

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

## 3. Check Dynamic Registration Parameters

In SQL*Plus, inspect:

```sql
SHOW PARAMETER instance_name;
SHOW PARAMETER service_names;
SHOW PARAMETER local_listener;
SHOW PARAMETER remote_listener;
```

Expected interpretations from transcript:

- `instance_name` defaults to SID.
- `service_names` typically reflects global DB name (`db_name.db_domain`).
- `local_listener` is an alias (not necessarily the literal listener name).
- `remote_listener` is `NULL` in this setup.

Exit SQL*Plus when done:

```sql
EXIT
```

---

## 4. Review Server-Side `tnsnames.ora`

1. Go to Net admin directory:

```bash
cd $ORACLE_HOME/network/admin
ls
```

2. Open with `less`:

```bash
less tnsnames.ora
```

Look for alias used by `local_listener` parameter (example: `LISTENER_ORCLCDB`-style alias), which resolves to listener endpoint data:

- protocol (`TCP`)
- host
- port (`1521`)

Exit `less` with:

```text
q
```

---

## 5. Review `listener.ora`

Display listener configuration:

```bash
cat listener.ora
```

Expected:

- default listener entry (`LISTENER`)
- endpoint address with protocol/host/port

---

## 6. Explore Listener with `lsnrctl`

Start listener control utility (default target is `LISTENER` when name omitted):

```bash
lsnrctl
```

Then run:

```text
help
show current_listener
status
services
show log_status
show log_file
```

What to verify:

- current listener is `LISTENER`
- status output includes version, uptime, parameter file, log file, endpoints
- services show `READY` for dynamically registered services
- handlers show connection stats (`established`, `refused`, `state`)
- log status is enabled (`ON` / `1`)
- log file path is visible

If service status appears `UNKNOWN`, dynamic registration (LREG communication) is not healthy.

Exit:

```text
exit
```

---

## 7. Lab Result

You validated:

- default listener configuration and endpoint
- local vs remote listener parameter values
- listener runtime status and service registration details
- logging configuration and log file location
