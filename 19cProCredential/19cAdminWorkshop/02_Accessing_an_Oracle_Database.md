# 2 - Accessing an Oracle Database (Or: How Not To Log In Like A Maniac)

And look, before you can create, tune, or nobly "fix" anything, you actually have to connect to the database. Preferably in a way that does not spray your password into shell history, process lists, or whatever you are screensharing.

This chapter is your tour of the main tools and connection methods you will use as a 19c admin.

You will see:

- `sqlplus` - the grumpy but essential command-line workhorse
- SQL Developer and `sqlcl`
- DBCA for database creation and configuration
- EM Express and EM Cloud Control for browser-based admin
- Supporting utilities: listener/network tools, diagnostics, loaders, backup tools
- How `ORAENV` and `/etc/oratab` keep your environment sane

By the end of this module, you should be able to:

- Connect to an Oracle database instance using common methods
- Compare SQL*Plus, SQL Developer, SQLcl, DBCA, and EM tools
- Explain how `oraenv` and `/etc/oratab` control environment setup
- Identify which tool is appropriate for each admin task

---

## 1. SQL*Plus - The Classic Command-Line Client

`sqlplus` is the standard command-line tool for connecting to and working with Oracle:

- Runs in a shell or terminal
- Lets you submit SQL and PL/SQL once a session is established
- Can connect as:
 - A regular database user (username/password)
 - An OS-authenticated privileged user (for example `SYSDBA`)

### 1.1 OS authentication (`/ AS SYSDBA`)

When you are logged into the OS as a user that belongs to the Oracle DBA group (for example `oracle` on Linux), you can connect without specifying a database username or password:

```bash
sqlplus / AS SYSDBA
```

Notes:

- No username or password are used; anything you type in that position is ignored
- Your effective database privilege is mapped from your OS login and group membership
- It is critical to know which OS user you are logged in as, because that is who the database trusts

This is the usual way the database owner connects to perform high-level admin work on the server.

### 1.2 Password authentication

Standard database user connection:

```bash
sqlplus system/oracle_4U@service_name
```

The `@service_name` part can be:

- A TNS alias defined in `tnsnames.ora`, or
- An Easy Connect string:

```bash
sqlplus system/oracle_4U@//host:port/service_name
```

The listener uses this information to route you to the right database instance or PDB.

### 1.3 Do not put passwords on the command line

Putting credentials directly in the command is convenient and also a terrible idea:

- Shell history, process lists and diagnostic tools can expose your password
- Anyone with enough OS privilege can see what you typed

Safer pattern:

```bash
sqlplus
-- then at the prompt:
Enter user-name: system
Enter password: ********
```

You still get a SQL prompt, but your password does not sit in shell history like an unexploded bomb.

### 1.4 Running scripts from SQL*Plus

Once connected, use `@` to run operating-system scripts:

```sql
-- Connect (if needed)
CONNECT system@pdborcl

-- Run a script
@/home/oracle/admin/scripts/create_users.sql
```

In a one-liner such as:

```bash
sqlplus system/oracle_4U@pdborcl @create_users.sql
```

- The first `@` (after the password) defines the service
- The second `@` (before the script name) runs the script after the connection is established

### 1.5 Setting your environment with `ORAENV` and `/etc/oratab`

Before you can comfortably invoke Oracle tools, your terminal needs to know:

- Which instance you care about (`ORACLE_SID`)
- Where its binaries live (`ORACLE_HOME`)

On Unix or Linux systems installed with the standard installer you typically get:

- `/etc/oratab` - a small "Oracle table" listing each database SID and its home
- `oraenv` / `ORAENV` - a helper script that reads `/etc/oratab` and sets environment variables

Typical usage:

```bash
. oraenv # dot-space to source it
ORACLE_SID = [orcl] ? orcl
```

Then:

```bash
echo $ORACLE_HOME
```

will show the correct home. From that point, `sqlplus`, `lsnrctl`, `adrci`, `rman` and friends are all on your `PATH` and pointing at the right database, instead of you spelunking through `/u01/app/oracle/...` every time.

---

## 2. GUI and Developer-Friendly Tools

Command line is powerful; GUIs stop your brain melting when you just need to inspect a schema. Oracle gives you several options.

### 2.1 SQL Developer

SQL Developer is a graphical IDE:

- Install it on your desktop or a central jump host
- Define connections to one or more databases
- Browse schemas, tables, views, users and more in a tree
- Use worksheets to run SQL, generate scripts, and tweak objects with point-and-click

It is especially friendly for:

- Developers writing SQL and PL/SQL
- DBAs who want quick visual confirmation of structures and data

### 2.2 SQLcl - SQL*Plus with modern conveniences

`sqlcl` is a command-line tool that:

- Speaks the same language as SQL*Plus (core commands, `@` scripts)
- Adds developer niceties:
 - Command history
 - Aliases
 - Inline formatting and scripting helpers

It is basically SQL*Plus after a productivity makeover.

---

## 3. DBCA - Database Configuration Assistant

DBCA is the GUI (and silent-mode-capable) tool for:

- Creating new databases (CDBs and PDBs)
- Configuring major database options (some can be changed post-creation)
- Managing pluggable databases in some scenarios
- Managing database templates so you can clone standardised configurations

Limitations:

- DBCA does not install Oracle software
- DBCA does not configure the Oracle network (listener, TNS naming)

Those prerequisites must be handled first (installer and Net tools). DBCA then builds databases on top of that foundation.

Common use cases:

- Create or delete a CDB or non-CDB
- Create / drop pluggable databases
- Configure options on an existing database

Example: configuring options on an existing database:

1. Start DBCA
2. Choose "Configure Database Options"
3. Select the `ORCL` database
4. Authenticate as `SYS` / `SYSDBA`
5. See which components are enabled (for example APEX), and enable or disable as needed

Installers tend to lay down all components; licensing does not. It is your job to know what you are licensed for and avoid casually turning everything on.

---

## 4. EM Express and EM Cloud Control

### 4.1 EM Express - per-database mini console

For each Oracle 12c+ database, you can enable Enterprise Manager Express (EM Express):

- Web UI running typically at `https://hostname:5500/em`
- Connect as a database user (for example `SYSTEM`)
- Can connect to the CDB root or a specific PDB
- Uses shared server by default to minimise load

What you can see and do:

- Home page with basic performance and activity charts
- Configuration:
 - View and manage initialization parameters
 - Review memory usage and available features
 - Manage Resource Manager
- Storage:
 - Manage undo segments
 - Manage redo logs, archive logging, and control files
 - Tablespace operations are intentionally limited
- Security:
 - Manage users and roles only
- Performance:
 - Performance Hub
 - SQL Tuning Advisor
 - SQL Performance Analyzer

Useful extra cues:

- The home page shows database status and type:
 - Single instance vs RAC
 - CDB (multitenant) vs non-CDB
- The top-right corner shows:
 - Which database user you are logged in as (for example `SYSTEM`)
 - A Logout link so you can close privileged sessions cleanly

EM Express is intentionally limited: a focused admin console, not a full enterprise tool.

### 4.2 Enterprise Manager Cloud Control - the big one

Enterprise Manager Cloud Control is the full, centralised management platform:

- Can manage:
 - Databases (Oracle and others)
 - Application servers
 - Operating systems
 - Networks and storage
 - Third-party applications with plug-ins
- Everything is registered into the EM repository and managed from one console

For non-Oracle components:

- Install appropriate management packs or plug-ins
- Once integrated, they can be monitored and controlled alongside Oracle assets

Cloud Control is where you go when someone says "we want a single pane of glass for the entire environment" and sounds worryingly serious.

### 4.3 Cloud Database Express

In Oracle Cloud deployments, additional cloud-specific tooling integrates with databases and app servers:

- Web consoles tied to Oracle's cloud infrastructure
- Database Express and related tools for provisioning, monitoring and scaling cloud databases

The basic concepts are similar; the deployment model and wrappers are cloud-aware.

---

## 5. Other Useful Command-Line Tools

Admin life is not just SQL clients. You will also use a stack of supporting tools.

### 5.1 Listener management - `lsnrctl`

`lsnrctl` controls the listener:

```bash
lsnrctl status
```

This shows:

- Listener name and version
- Host and port it is listening on
- Which services and instances are registered

You can explore commands with:

```bash
lsnrctl help
lsnrctl help status
```

### 5.2 Network configuration - Net Manager and NetCA

Net Manager (`netmgr`) is a graphical tool:

- Edit `sqlnet.ora` (naming methods, tracing, etc.)
- Edit `tnsnames.ora` (service naming aliases)
- Edit `listener.ora` (listeners)

It uses a tree view:

- Profile - things that end up in `sqlnet.ora`
- Service Naming - TNS entries for databases and PDBs
- Listeners - listener definitions

Important: when you are done, you must use "File -> Save Network Configuration". Closing the window does not automatically save changes.

Net Configuration Assistant (`netca`) is a wizard-style tool:

- Create and configure listeners
- Configure local naming (TNS entries)
- Configure directory naming

It is the "click Next a few times" approach to basic network setup.

### 5.3 Diagnostics - `adrci`

`adrci` is the Automatic Diagnostic Repository Command-Line Interface. It lets you inspect:

- Alert log
- Incidents and problems
- Trace files and dumps

Typical usage:

```bash
adrci
adrci> help
adrci> show alert
```

You can drill into specific items (for example `show incident`, `show problem`) and see when errors, ORA- messages or warnings occurred.

### 5.4 Data movement - SQL*Loader and Data Pump

- `sqlldr` - SQL*Loader; loads flat files into Oracle tables, usually when data is coming from non-Oracle sources
- Data Pump:
 - `expdp` - Data Pump export
 - `impdp` - Data Pump import

Data Pump is the modern replacement for the older `exp`/`imp` utilities, and supports high-speed exports/imports with granular control.

### 5.5 Backup and recovery - `rman`

`rman` (Recovery Manager) handles:

- Backup and restore operations
- Database duplication and cloning
- Archivelog backup and deletion
- Incremental backup strategies

You will see `rman` extensively in the backup and recovery chapters.

---

## 6. Summary - You Have Options (Use Them Wisely)

By now you should be able to:

- Decide when to use:
 - `sqlplus` vs SQL Developer vs `sqlcl`
 - DBCA for database creation and configuration
 - EM Express for lightweight per-database administration
 - EM Cloud Control for enterprise-wide management
- Connect securely without putting passwords on the command line
- Run scripts through `sqlplus` and understand how listeners and services fit into the picture
- Recognise the other admin tools you will need (listener tools, Net tools, `adrci`, SQL*Loader, Data Pump, `rman`)

In short: you now know the difference between "I can technically connect to the database" and "I can connect properly and safely with the right tool for the job", which is a surprisingly rare skill.

