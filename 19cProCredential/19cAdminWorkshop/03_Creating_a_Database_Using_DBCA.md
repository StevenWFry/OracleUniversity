# 3 - Creating a Database Using DBCA (Because Nobody Wants To Type `CREATE DATABASE` By Hand)

And look, you *can* build an Oracle database entirely by hand with a parameter file, `STARTUP NOMOUNT`, and a severalscreen `CREATE DATABASE` command. You can also churn your own butter. DBCA exists so you do not have to.

This chapter is about:

- What you should plan **before** you click "Create Database"
- What DBCA can and cannot do for you
- Using templates, workload types, and character sets sensibly
- The flow of the interactive DBCA wizard
- What silent mode looks like when you are scripting builds

By the end of this module, you should be able to:

- Plan a database before creation (storage, memory, options)
- Choose an appropriate DBCA template and workload type
- Select character set and understand `NLS_LANG` alignment
- Use DBCA interactively and in silent mode

---

## 1. Planning Before Creation (Yes, Really)

DBCA is a powerful tool, but it is not a mind reader. Before you launch it, you should know what you are trying to build.

### 1.1 Physical layout - storage and files

Questions to answer:

- Where will **datafiles** go?
- How will you **multiplex redo logs** and **control files**?
- Will you use a **fast recovery area**?
 - Which filesystem or ASM disk group?
 - How big should it be?
- Do you have a separate location for **backups**?

In other words:

- Decide filesystem vs ASM
- Decide directory / diskgroup naming conventions
- Decide where Oraclemanaged files (OMF) should live

DBCA will then set its filelocation options and parameters accordingly.

### 1.2 Instance characteristics - memory and options

On the logical side, think about:

- How much memory is available on the host?
- How much should the instance use?
 - Automatic Memory Management (AMM) vs manual SGA/PGA sizing
- How many **processes** (sessions) do you expect?
- Which components/features should be enabled?
 - EM Express?
 - Database Vault?
 - Label Security?
 - Sample schemas (HR, etc.)?

### 1.3 Things DBCA does *not* do

DBCA is not the entire stack:

- It **does not** install Oracle software
- It **does not** configure the Oracle network:
 - No listener
 - No `sqlnet.ora`
 - No TNS aliases

You must already have:

- Database software installed
- At least one listener created (via NetCA or manually)

DBCA then creates:

- The **instance** (memory + parameters)
- The **physical database** (control file, redo, datafiles)
- CDB/PDB structures if requested

---

## 2. Templates, Workloads, and Character Sets

### 2.1 Templates and workload types

DBCA can use:

- **Oraclesupplied templates**, such as:
 - General Purpose / Transaction Processing
 - Data Warehouse
- **Custom templates** you created from existing databases:
 - Same structural layout
 - Same options and settings
 - Great for cloning environments quickly

The workload choice mainly affects defaults:

- **General Purpose** - mixed OLTP + reporting + web apps
- **Data Warehouse** - geared toward analytic queries and batch loads

### 2.2 Character sets - database and national

You must choose:

- Database character set
- National character set (`NCHAR`, `NVARCHAR2`, `NCLOB`)

Consider:

- Languages and scripts to support
- Client platforms and their character sets
- Singlebyte vs multibyte / Unicode sets

Tradeoffs:

- Fixedsize multibyte sets:
 - Simpler handling, but can waste space if many characters are "small"
- Variablesize sets:
 - More compact overall, but conversions are trickier

The **client** must speak the same logical character set as the database. That is controlled by `NLS_LANG` on the client.

Example: if the database uses `AL32UTF8` but the client default is `WE8MSWIN1252`, on the client:

```bash
export NLS_LANG=.AL32UTF8
```

Now both sides speak the same encoding when exchanging data.

---

## 3. What DBCA Can Do For You

With the prerequisites in place, DBCA can:

- Create CDB and nonCDB databases
- Configure instance and database structures together
- Set many database options up front
- Delete a database **and** its physical files
- Work with container databases:
 - Create pluggable databases (PDBs)
 - Drop PDBs
 - Use the seed PDB as a base

You can invoke DBCA in two modes:

- **Interactive** - GUI pages, you click through
- **Silent** - command line with options, no prompts

---

## 4. DBCA in Silent Mode (NonInteractive)

Silent mode is great for scripted builds, but terrible for live demos because nothing happens on screen except a blinking cursor and eventual success.

Typical silent **create**:

```bash
dbca -silent \
 -createDatabase \
 -templateName General_Purpose.dbc \
 -gdbname orcl.example.com \
 -sid orcl \
 -createAsContainerDatabase true \
 -numberOfPDBs 1 \
 -pdbName pdb1 \
 -pdbAdminPassword PdbAdmin_4U \
 -sysPassword Sys_4U \
 -systemPassword System_4U \
 -memoryPercentage 40 \
 -emExpressPort 5500
```

To **delete** a database:

```bash
dbca -silent \
 -deleteDatabase \
 -sourceDB orcl \
 -sysDBAUserName sys \
 -sysDBAPassword Sys_4U
```

The idea is:

- You tell DBCA exactly what you want (`-createDatabase`, `-deleteDatabase`)
- You supply all required parameters on the command line
- DBCA does the rest without asking followup questions

---

## 5. DBCA in Interactive Mode - Wizard Walkthrough

Now for the version you can show in a classroom without everyone falling asleep.

From a properly set environment:

```bash
dbca
```

### 5.1 Pick an action

First screen: choose what you want to do:

- Create a database
- Configure database options
- Delete a database
- Manage templates
- Manage pluggable databases

Pick **Create a database**, then move to advanced configuration to see all pages.

### 5.2 Instance type and template

You first decide:

- Instance type:
 - Single instance
 - RAC
 - RAC One Node
- Template / purpose:
 - Oraclesupplied (for example General Purpose)
 - Custom template you created earlier

### 5.3 Database and instance name, CDB/PDB layout

Next page: naming.

- Global database name:
 - `db_name.db_domain` (for example `rondb.example.com`)
 - This becomes the full service name seen on the network
- Instance name:
 - Usually defaults to `db_name` (for example `rondb`)

Decide if this will be:

- A container database (CDB) or not
- If CDB:
 - Use local undo or shared undo
 - Whether to precreate any PDBs, and their names (for example `ronpdb`)

### 5.4 Storage - file system vs ASM and locations

Choose:

- Storage type:
 - File system
 - ASM (if configured)
- Whether to use locations from the template (Oracle Flexible Architecture style), or pick custom paths

Configure the **fast recovery area**:

- Location
- Size
- Whether archiving is enabled

### 5.5 Listener selection

DBCA detects existing listeners and asks:

- Use an **existing** listener?
- Or **create a new** one?

Creating a new listener:

- Under the covers, DBCA invokes Net Configuration Assistant (NetCA)
- NetCA builds the listener and returns control to DBCA

Using an existing listener:

- DBCA registers the new database with that listener

### 5.6 Options - Database Vault, Label Security, and friends

You can choose to enable:

- Oracle Database Vault
- Label Security
- Other options, depending on edition and licensing

Remember: the installer often puts the binaries on disk whether you are licensed or not; enabling features is on you.

### 5.7 Memory and sizing

Instance settings:

- Use Automatic Memory Management (AMM) or manually set SGA/PGA components
- Set overall memory target (for example 1500 MB)
- Set process count (maximum number of sessions/processes)

If AMM is not allowed for your platform configuration, DBCA will insist that you adjust the settings before continuing.

### 5.8 Character sets and connection mode

You pick:

- Database character set (for example `AL32UTF8`)
- National character set

Connection mode:

- Dedicated server connections
- Shared server connections

DBCA presents these as radio buttons, but you can later configure both via parameters if needed.

Sample schemas:

- You can choose to install sample schemas (for example the HR schema) to quickly validate the database and demo features

### 5.9 Management - EM Express and/or Cloud Control

EM Express:

- Enable EM Express for this database
- Choose a port (for example 5501 instead of the default 5500)

Cloud Control:

- Optionally register the database with an existing EM Cloud Control installation during creation

### 5.10 Passwords

You set passwords for:

- `SYS`
- `SYSTEM`
- And optionally PDB admin users

You can:

- Use the same password for all (quick for demos, terrible for production)
- Or specify different passwords per account

### 5.11 Final review and creation

Before DBCA starts creating files, you get a summary page:

- Chosen options
- Initialization parameters
- Storage locations (datafiles, redo, control files)
- Recovery area settings

You can:

- Adjust initialization parameters
- Tweak file locations and names
- Choose to:
 - Create the database now
 - Save the configuration as a template
 - Generate scripts instead of directly creating the database

Generating scripts:

- Useful when DBCA is available on one host, but you must create the database on another
- You run the generated SQL and shell scripts in the correct order to build the database manually

Once you click **Finish**, DBCA builds the database:

- Creates the instance and starts it
- Executes `CREATE DATABASE` with your settings
- Creates dictionary objects and sample schemas (if selected)
- Configures EM Express, registration with the listener, and optional Cloud Control

If you are short on disk space, DBCA will complain early, sparing you from watching a progress bar fail at 93%.

---

## 6. Summary - Let The Tool Do The Boring Parts

By now you should be able to:

- Plan a database creation: storage layout, memory, options, character sets, CDB/PDB design
- Understand what DBCA can and cannot do (it builds databases; it does not install software or network)
- Use DBCA:
 - In **silent mode** for scripted, repeatable builds
 - In **interactive mode** for guided, fully specified builds
- Make sensible choices about:
 - Templates and workload types
 - CDB vs nonCDB, PDB naming and undo
 - Character sets and sample schemas
 - EM Express and Cloud Control integration

In short: you now know how to let DBCA handle the heavy lifting of database creation while you focus on the parts that actually require judgment, like "where should this go?" and "what on earth are we going to call it?" That alone puts you ahead of a distressing number of installations in the wild.

---

## Lab 3 Link

When you are ready to practice this in the lab environment, use:

- [Lab 3 - Creating `CDBTEST` with DBCA Silent Mode](labs/lab03-create-cdbtest-dbca-silent.md)

