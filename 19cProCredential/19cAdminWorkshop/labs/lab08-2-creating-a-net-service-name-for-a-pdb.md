# Lab 8-2 - Creating a Net Service Name for a PDB

Time for Practice 2-2: create a clean Oracle Net alias for a pluggable database, test it, and prove you landed in the right container instead of accidentally chatting with the root.

---

## 1. Practice Goal

Create a net service name called `MyPDB1` for `ORCLPDB1` using Oracle Net Manager, then validate connectivity and container context.

---

## 2. Set Oracle Environment

1. Open a terminal.
2. Set environment variables to the CDB home:

```bash
. oraenv
```

When prompted, enter:

```text
orclcdb
```

---

## 3. Review Existing `tnsnames.ora` Entries

1. Navigate to Oracle Net admin directory:

```bash
cd $ORACLE_HOME/network/admin
ls
```

2. Open local naming file:

```bash
gedit tnsnames.ora &
```

3. Confirm existing entries (as discussed in transcript), including:

- `ORCLCDB` style entry for the container database
- existing PDB aliases (for example, `PDB1`, `PDB2`, or similarly named entries)

Close editor after review.

---

## 4. Create `MyPDB1` in Net Manager

1. Start Oracle Net Manager:

```bash
netmgr
```

2. In UI:

- Expand `Local` -> `Service Naming`
- Click `Service Naming`, then click green `+`
- Net service name: `MyPDB1`
- Protocol: `TCP/IP`
- Hostname: host FQDN or IP
- Port: `1521`
- Service name: `ORCLPDB1`

If needed, get host FQDN from terminal:

```bash
hostname -f
```

Example shown in demo:

```text
edvmr1p0.us.oracle.com
```

3. Click `Test`.

Expected first result:

- Failure with default `scott/tiger`.

4. Click `Change Login` and use:

- Username: `system`
- Password: `cloud_4U`

5. Click `Test` again; confirm successful test.
6. Click `Close`, then `Finish`.
7. Save explicitly:

- `File` -> `Save Network Configuration`

8. Exit Net Manager.

---

## 5. Confirm New Alias in `tnsnames.ora`

Open file and verify `MyPDB1` entry now exists:

```bash
gedit tnsnames.ora &
```

Look for alias block named `MyPDB1`.

---

## 6. Test Alias with `tnsping`

Run:

```bash
tnsping MyPDB1
```

Interpretation:

- `OK` indicates naming resolution and listener reachability are working.
- It does **not** guarantee full database service usability on its own.

---

## 7. Connect and Verify Container

1. Connect with SQL*Plus using the new alias:

```bash
sqlplus system@MyPDB1
```

Password:

```text
cloud_4U
```

2. Verify current container:

```sql
SHOW CON_NAME
```

Expected:

- `ORCLPDB1`

3. Exit SQL*Plus:

```sql
EXIT
```

---

## 8. Lab Result

You have:

- created `MyPDB1` Oracle Net alias
- saved it to `tnsnames.ora`
- validated listener connectivity with `tnsping`
- connected as `system` and confirmed container `ORCLPDB1`
