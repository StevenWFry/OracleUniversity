# Lab 8-1 - Configuring Oracle Network Access to a Database (Part 1)

Because nothing says "fun afternoon" like editing network files, launching DBCA in silent mode, and waiting 20 minutes while pretending that progress bars are emotionally stable.

This practice starts Lesson 2 network naming work by preparing a second CDB (`CDBTEST`) and setting up the environment needed for naming-method configuration.

---

## 1. Practice Goal

In this lab segment, you:

- Start local database and listener services
- Prepare and run a DBCA silent script to create `CDBTEST`
- Set up for local naming validation in later steps (`testorcl` service name)

From the practice overview:

- Configure network files to access a database on another server
- Configure access to a pluggable database
- Use local naming and test connectivity through a new network service name

---

## 2. Assumptions

- `CDBTEST` does **not** already exist
- You are logged in as the `oracle` OS user
- Lab credentials use:

```text
cloud_4U
```

---

## 3. Start Local Database and Listener

1. Open terminal from desktop.
2. Run the startup helper script:

```bash
cd /home/oracle/labs
./dbstart.sh
```

Expected flow shown in transcript:

- Listener starts
- Database mounts
- Database opens

---

## 4. Prepare DBCA Silent Script for `CDBTEST`

Use the provided template script:

```bash
cd /home/oracle/labs/dbmodnet/services
ls
```

You should see:

```text
CrCDBTEST.sh
```

Edit the script:

```bash
gedit CrCDBTEST.sh &
```

Replace password placeholders with:

```text
cloud_4U
```

Specifically validate values for arguments like:

- `-sysPassword`
- `-systemPassword`
- `-pdbAdminPassword`
- any other password flag in the script

Save and close the editor.

---

## 5. Execute Script

Make executable, then run:

```bash
chmod 755 CrCDBTEST.sh
./CrCDBTEST.sh
```

Runtime note from the instructor:

- Creation may take about 20 minutes.
- Recording pauses during database creation and resumes after completion.

---

## 6. After DBCA Completes, Verify Output

When script execution finishes, verify DBCA output shows:

- `100%` completion
- global database name
- SID
- log directory and log filename
- return to shell prompt

---

## 7. Verify `oratab` and Set Environment

1. Confirm new `CDBTEST` entry exists in `/etc/oratab`:

```bash
cat /etc/oratab
```

2. Set Oracle environment:

```bash
. oraenv
```

When prompted, enter:

```text
orclcdb
```

---

## 8. Back Up `tnsnames.ora`

1. Go to Oracle Net admin directory:

```bash
cd $ORACLE_HOME/network/admin
ls
```

2. Copy `tnsnames.ora`:

```bash
cp tnsnames.ora tnsnames.old
ls -l
```

Confirm both files exist and are writable by your user.

---

## 9. Build `TESTORCL` with Net Manager

1. Capture host FQDN:

```bash
hostname
```

2. Start Net Manager:

```bash
netmgr
```

3. In UI:

- Expand `Local` -> `Service Naming`
- Select `Service Naming`, click green `+`
- Net service name: `TESTORCL`
- Protocol: `TCP/IP`
- Hostname: your FQDN (example from demo: `edvmr1p0.us.oracle.com`)
- Port: `1521`
- Service: `CDBTEST`

4. Click `Test`.

Expected first result:

- Failure with default `scott/tiger` login.

5. Click `Change Login` and use:

- User: `system`
- Password: `cloud_4U`

6. Run `Test` again and confirm success.

7. Click `Close`, then `Finish`.

8. Save explicitly:

- `File` -> `Save Network Configuration`

9. Exit Net Manager.

---

## 10. Test New Naming Entry with SQL*Plus

1. Connect through alias:

```bash
sqlplus system@testorcl
```

Password:

```text
cloud_4U
```

2. Validate target instance and host:

```sql
SELECT instance_name, host_name
FROM   v$instance;
```

Expected:

- Instance shows `CDBTEST`
- Hostname matches your target server

3. Exit SQL*Plus:

```sql
EXIT
```

---

## 11. Lab Result

You have:

- created `CDBTEST` with DBCA silent mode
- added local naming alias `TESTORCL`
- verified successful Oracle Net connection through alias resolution
