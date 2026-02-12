# Lab 17-3 - Exploring OS and Password File Authentication

Practice 1-3 examines how Linux group membership maps to Oracle administrative
authentication and how password-file users appear in `V$PWFILE_USERS`.

---

## 1. Practice Goal

You will:

- inspect OS group/user metadata for `oracle`
- verify why OS-authenticated `oracle` can use `SYSDBA`
- query password-file-backed administrative identities in the database

Assumption:

- logged in to compute node as `oracle` OS user

---

## 2. Explore OS Authentication Inputs

View Linux groups:

```bash
cat /etc/group
```

Confirm current OS user:

```bash
whoami
```

Expected:

- `oracle`

Inspect `oracle` entry in passwd file:

```bash
grep oracle /etc/passwd
```

Find primary group by ID:

```bash
grep oinstall /etc/group
```

Find all groups containing `oracle`:

```bash
grep oracle /etc/group
```

Alternative one-command summary:

```bash
id
```

Key interpretation:

- membership in `dba` group allows OS-authenticated `SYSDBA` behavior.

---

## 3. Explore Password File Authentication

Connect as SYSDBA:

```bash
sqlplus / as sysdba
```

Describe password-file users view:

```sql
DESC v$pwfile_users;
```

List users present in password file:

```sql
COL username FORMAT A20
SELECT username
FROM   v$pwfile_users;
```

Transcript result:

- `SYS`
- `C##CDB_ADMIN1` (from prior lab steps)

Check SYS account status and SYSDBA flag:

```sql
SELECT account_status, sysdba
FROM   v$pwfile_users
WHERE  username = 'SYS';
```

Expected:

- account status `OPEN`
- `SYSDBA = TRUE`

---

## 4. Exit

```sql
EXIT
```

Close terminal.

---

## 5. Lab Result

You validated OS group-based administrative authentication (`oracle` in `dba`)
and confirmed password-file administrative users and status through
`V$PWFILE_USERS`.
