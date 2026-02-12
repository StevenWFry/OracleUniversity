# Lab 11-3 - Configuring the Database for Oracle Connection Manager

Practice 5-3 configures the database side so services register through Oracle Connection Manager.

---

## 1. Practice Goal

Configure the database to register with CMAN by:

- adding a CMAN listener alias in server-side `tnsnames.ora`
- updating listener registration parameter(s)
- forcing immediate registration

---

## 2. Set Database Environment

Open terminal and set environment:

```bash
. oraenv
```

When prompted, enter:

```text
orclcdb
```

---

## 3. Add CMAN Alias in Server-Side `tnsnames.ora`

1. Go to network admin directory:

```bash
cd $ORACLE_HOME/network/admin
```

2. Edit file:

```bash
gedit tnsnames.ora &
```

3. Add a listener alias for CMAN (demo used copy/edit approach from existing entry):

```text
LISTENER_CMAN =
  (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1522))
```

4. Save and close.

---

## 4. Update Database Registration to CMAN

Connect as SYSDBA:

```bash
sqlplus / as sysdba
```

Set registration target to CMAN alias:

```sql
ALTER SYSTEM SET remote_listener='LISTENER_CMAN';
```

Force immediate registration:

```sql
ALTER SYSTEM REGISTER;
```

Exit SQL*Plus:

```sql
EXIT
```

---

## 5. Lab Result

The database is configured to resolve and register against Oracle Connection Manager (`localhost:1522`) using the `LISTENER_CMAN` alias and immediate re-registration.
