# Lab 9-2 - Creating a Second Listener

Practice 3-2 builds a second listener (`LISTENER2`) on a non-default port, then verifies dynamic service registration catches up after startup.

---

## 1. Practice Goal

Create and manage a second listener:

- listener name: `LISTENER2`
- protocol: `TCP`
- non-default port: `1561`

Then verify that database services register dynamically with this new listener.

Assumption:

- Logged in as `oracle` OS user.

---

## 2. Add `LISTENER2` Alias to `tnsnames.ora`

1. Get host FQDN:

```bash
hostname -f
```

2. Go to Oracle Net admin directory:

```bash
cd $ORACLE_HOME/network/admin
```

3. Back up current file:

```bash
cp tnsnames.ora tnsnames.ora.3-2
ls
```

4. Edit file:

```bash
gedit tnsnames.ora &
```

5. Add alias entry for `LISTENER2` (copy existing local listener alias as template), using:

- host: your FQDN
- port: `1561`
- protocol: `TCP`

Example pattern:

```text
LISTENER2 =
  (ADDRESS = (PROTOCOL = TCP)(HOST = your-host.example.com)(PORT = 1561))
```

6. Save and close.

---

## 3. Update `LOCAL_LISTENER` Parameter

1. Open new terminal and set environment:

```bash
. oraenv
```

When prompted, enter:

```text
orclcdb
```

2. Connect as SYSDBA:

```bash
sqlplus / as sysdba
```

3. Show current local listener:

```sql
SHOW PARAMETER local_listener;
```

4. Confirm parameter is dynamic/system-modifiable:

```sql
SELECT name, isses_modifiable, issys_modifiable
FROM   v$parameter
WHERE  name = 'local_listener';
```

5. Set both aliases:

```sql
ALTER SYSTEM SET local_listener='"LISTENER_ORCLCDB","LISTENER2"';
```

6. Verify:

```sql
SHOW PARAMETER local_listener;
```

7. Exit SQL*Plus (keep terminal open):

```sql
EXIT
```

---

## 4. Add `LISTENER2` to `listener.ora` via Net Manager

Why this matters:

- Dynamic registration does not require static DB entries in `listener.ora`.
- But `lsnrctl` management of named listeners needs listener address resolution.

Steps:

1. Back up listener file:

```bash
cd $ORACLE_HOME/network/admin
cp listener.ora listener.old
ls
```

2. Start Net Manager:

```bash
netmgr
```

3. In UI:

- Expand `Local` -> `Listeners`
- Click green `+`
- Listener name: `LISTENER2`
- Select `LISTENER2` and click `Add Address`
- Protocol: `TCP/IP`
- Hostname: your FQDN
- Port: `1561`

4. Optionally review:

- General Parameters
- Logging and Tracing
- Authentication
- Database Services (expected empty for new listener)

5. Save and exit:

- `File` -> `Save Network Configuration`
- `File` -> `Exit`

6. Confirm entry written:

```bash
gedit listener.ora &
```

You should see `LISTENER2` with endpoint data.

---

## 5. Check and Start `LISTENER2` with `lsnrctl`

1. Start listener control utility:

```bash
lsnrctl
```

2. Initial check (expected not running yet):

```text
status LISTENER2
```

Expected:

- no listener / connection refused

3. Start listener:

```text
start LISTENER2
```

4. Check status again:

```text
status LISTENER2
```

Immediately after startup, service list may be empty.

5. Wait about 60 seconds, then recheck:

```text
status LISTENER2
```

Now services should appear via dynamic registration.

Note:

- Transcript says `RFS`; this behavior is generally associated with listener registration process (`LREG`) timing.

6. Exit utility:

```text
exit
```

---

## 6. Lab Result

You have:

- created listener alias `LISTENER2` in `tnsnames.ora`
- updated `LOCAL_LISTENER` to include both local aliases
- added `LISTENER2` endpoint config in `listener.ora`
- started and validated `LISTENER2`
- confirmed delayed dynamic service registration behavior after startup
