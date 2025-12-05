# 2 – Accessing an Oracle Database (Or: How Not to Log In Like a Maniac)

And look, before you can create or administer anything, you actually have to **connect** to the database—ideally in a way that doesn’t spray your password all over the command history. This chapter is your guided tour of the main tools and connection methods.

We’ll hit:

- `SQL*Plus` (the grumpy but essential command‑line workhorse)  
- SQL Developer and `SQLcl`  
- DBCA for database creation  
- EM Express and EM Cloud Control for GUI administration  
- A grab‑bag of other admin tools (listener, networking, diagnostics, loaders, backups)

---

## 1. SQL*Plus – The Classic Command‑Line Client

**SQL*Plus** is the default command‑line tool for connecting to and working with Oracle:

- Runs in a shell/terminal  
- Lets you submit SQL and PL/SQL once a session is established  
- Can connect as:
  - A regular database user (username/password)  
  - A privileged OS‑authenticated user (e.g. `SYSDBA`)

### 1.1 OS authentication (`/ AS SYSDBA`)

When you’re logged into the OS as a user that belongs to the Oracle DBA group (e.g. `oracle` on Linux), you can connect without specifying a database username/password:

```bash
sqlplus / AS SYSDBA
```

Notes:

- No username/password are used; they are **ignored** if you type them  
- Your database privilege is mapped from your **OS login**  
- It’s critical to know which OS user you’re logged in as—because that’s who the database trusts

### 1.2 Password authentication

Standard database user connection:

```bash
sqlplus system/oracle_4U@service_name
```

The `@service_name` part can be:

- A **TNS alias** defined in `tnsnames.ora`, or  
- An **Easy Connect** string:

```bash
sqlplus system/oracle_4U@//host:port/service_name
```

The listener uses this information to route you to the right database instance.

### 1.3 Do **not** put passwords on the command line

Putting credentials directly in the command is convenient and also a terrible idea:

- Shell history, process lists, and diagnostic tools can expose it  
- Anyone with enough OS privilege can see what you typed

Safer pattern:

```bash
sqlplus
-- then at the prompt:
Enter user-name: system
Enter password: ********
```

You still get a SQL prompt, but your password doesn’t sit around in your shell history like an unexploded bomb.

### 1.4 Running scripts from SQL*Plus

Once connected, use `@` to run OS scripts:

```sql
-- Connect (if needed)
CONNECT system@pdborcl

-- Run a script
@/home/oracle/admin/scripts/create_users.sql
```

The first `@` in a `sqlplus user/pass@service @script.sql` line defines the **service**; the second `@` runs the script **after** the connection is established.

---

## 2. GUI and Developer‑Friendly Tools

Command line is powerful; GUIs keep you from going cross‑eyed. Oracle gives you several.

### 2.1 SQL Developer

**SQL Developer** is a graphical IDE:

- Install it on your desktop or a central jump host  
- Define connections to one or more databases  
- Browse schemas, tables, views, users, and more in a tree  
- Use worksheets to run SQL, generate scripts, and tweak objects with point‑and‑click

It’s especially friendly for:

- Developers writing SQL/PLSQL  
- DBAs who want quick visual confirmation of structures and data

### 2.2 SQLcl / SQLclI – SQL*Plus with modern conveniences

**SQLcl** is a command‑line tool that:

- Speaks the same language as SQL*Plus (core commands, `@` scripts, etc.)  
- Adds developer niceties:
  - Command history  
  - Aliases  
  - Inline formatting and scripting helpers

It’s a bit like SQL*Plus went to therapy and decided to be more helpful.

---

## 3. DBCA – Database Configuration Assistant

**DBCA** is the GUI (and silent‑mode capable) tool for:

- Creating new databases (CDBs and PDBs)  
- Configuring major database options (some can be changed post‑creation)  
- Managing pluggable databases in some scenarios

Limitations:

- DBCA does **not** install Oracle software  
- DBCA does **not** configure the Oracle network (listener, TNS)  

Those prerequisites must be handled first (installer / Net tools), then DBCA builds databases on top of that foundation.

---

## 4. EM Express and EM Cloud Control

### 4.1 EM Express – per‑database mini console

For each Oracle 12c+ database, you can enable **Enterprise Manager Express (EM Express)**:

- Web UI running typically at `https://hostname:5500/em`  
- Connect as a database user (e.g. `SYSTEM`)  
- Can connect to CDB root or a specific PDB  
- Uses shared server by default to minimise load

What you can see/do:

- Home page with basic performance and activity charts  
- Configuration:
  - View and manage initialization parameters  
  - Review memory usage and available features  
  - Manage Resource Manager
- Storage:
  - Manage undo segments  
  - Manage redo logs, archive logging, and control files  
  - (Tablespace operations are intentionally limited)
- Security:
  - Manage **Users** and **Roles** only
- Performance:
  - Performance Hub  
  - SQL Tuning Advisor  
  - SQL Performance Analyzer

It’s intentionally **limited**—a focused admin console, not a full enterprise tool.

### 4.2 Enterprise Manager Cloud Control – the big one

**EM Cloud Control** is the full, centralised management platform:

- Manage:
  - Databases (Oracle and others)  
  - Application servers  
  - Operating systems  
  - Networks and storage  
  - Third‑party apps with plug‑ins
- Everything is registered into the EM repository and managed from one console

For non‑Oracle components:

- Install appropriate **management packs / plug‑ins**  
- Once integrated, they can be monitored and controlled alongside Oracle assets

Cloud Control is where you go when someone says “we want a single pane of glass for the entire environment” and seems to mean it.

### 4.3 Cloud Database Express

In Oracle Cloud deployments, additional **cloud‑specific** tooling integrates with databases and app servers:

- Web consoles tied to Oracle’s cloud infrastructure  
- Database Express and related cloud tools for provisioning, monitoring, scaling

The basic concepts are similar; the deployment model and wrappers are cloud‑aware.

---

## 5. Other Useful Command‑Line Tools

Admin life isn’t just SQL clients. You’ll also use:

- **Listener management**
  - `lsnrctl` – control the listener (start/stop/status)  

- **Network configuration**
  - `netca` – Net Configuration Assistant (create listeners, configure naming)  
  - `netmgr` – Net Manager GUI  
  - `sqlnet.ora`, `tnsnames.ora` – configuration files

- **Diagnostics**
  - `adrci` – Automatic Diagnostic Repository Command‑Line Interface
    - View and manage trace files, incidents, alert logs

- **Data movement**
  - `sqlldr` – SQL*Loader; load flat files into Oracle tables  
  - Data Pump:
    - `expdp` – Data Pump export  
    - `impdp` – Data Pump import

- **Backup and recovery**
  - `rman` – Recovery Manager
    - Backups, restores, duplication, archivelog management

You’ll see several of these tools in use throughout the admin workshop, especially when we get into backup/recovery, migration, and diagnostics.

---

## 6. Summary – You Have Options (Use Them Wisely)

By now you should be able to:

- Decide when to use:
  - `SQL*Plus` vs SQL Developer vs `SQLcl`  
  - DBCA for database creation  
  - EM Express for lightweight per‑database admin  
  - EM Cloud Control for enterprise‑wide management  
- Connect securely without putting passwords on the command line  
- Run scripts through SQL*Plus and understand how listeners and services fit into the picture  
- Recognise the other admin tools you’ll need (listener, Net tools, ADRCI, SQL*Loader, Data Pump, RMAN)

In short: you now know the difference between “I can technically connect to the database” and “I can connect **properly** and **safely** with the right tool for the job,” which is a surprisingly rare skill.  

---


