# 3 – Creating a Database Using DBCA (Because Nobody Wants To Type `CREATE DATABASE` By Hand)

And look, you *can* build an Oracle database entirely by hand with a parameter file, `nomount` and a several‑screen `CREATE DATABASE` command. You can also churn your own butter. DBCA exists so you do not have to.

This chapter is about:

- What you should plan **before** you click “Create Database”  
- What DBCA can and cannot do for you  
- Using templates, character sets, and options sensibly  
- The flow of the interactive DBCA wizard  
- What the silent mode looks like when you are scripting builds

---

## 1. Planning Before Creation (Yes, Really)

DBCA is a powerful tool, but it is not a mind reader. Before you launch it, you should know:

### 1.1 Physical layout – storage and files

Questions to answer:

- Where will **datafiles** go?  
- How will you **multiplex redo logs** and **control files**?  
- Will you use a recovery/fast recovery area? If so:
  - Which disk / ASM disk group?  
  - How big should it be?  

The idea is to decide:

- Filesystem vs ASM  
- Directory structure or disk group naming conventions  
- Whether backups will land locally or somewhere else

DBCA will then set its file‑location options and parameters accordingly.

### 1.2 Instance characteristics – memory and options

On the logical side, you need to think about:

- How much memory is available on the host?  
- How much should the instance use?  
  - Automatic Memory Management (AMM) vs manual component sizes  
- How many **processes** (connections) do you expect?  
- Which components/features should be enabled?  
  - EM Express?  
  - Database Vault?  
  - Label Security?  
  - Sample schemas?

### 1.3 Things DBCA does *not* do

DBCA is not the entire stack:

- It **does not** install Oracle software  
- It **does not** configure the Oracle network:
  - No listener  
  - No `sqlnet.ora`  
  - No TNS aliases

You must already have:

- Software installed  
- At least one listener created (via NetCA or manually)

DBCA then creates:

- The instance (memory + parameters)  
- The physical database (controlfile, redo, datafiles)  
- CDB/PDB structures if requested

---

## 2. Templates, Workloads, and Character Sets

### 2.1 Templates and workload types

DBCA can use:

- **Oracle‑supplied templates**, such as:
  - General Purpose / Transaction Processing  
  - Data Warehouse  
- **Custom templates** you created from existing databases:
  - Same structural layout  
  - Same options and settings  
  - Faster creation of clones with consistent config

Workload choice affects defaults:

- General Purpose – mixed OLTP + reporting + web apps  
- Data Warehouse – geared toward large queries and batch loads

### 2.2 Character sets – database and national

You must choose:

- Database character set  
- National character set (NCHAR, NVARCHAR2, NCLOB)

Consider:

- Languages and scripts to support  
- Client platforms and their character sets  
- Single‑byte vs multi‑byte / Unicode sets

Trade‑offs:

- Fixed‑size multibyte sets:
  - Simpler handling, but can waste space if many characters are “small”  
- Variable‑size sets:
  - More compact overall, but conversions are more nuanced

The **client** must speak the same logical character set as the database:

- Controlled by `NLS_LANG` on the client  
- Example: if the database uses `AL32UTF8` but the client default is `WE8MSWIN1252`, set:

  ```bash
  export NLS_LANG=.AL32UTF8
  ```

So that client and server speak the same character encoding when exchanging data.

---

## 3. What DBCA Can Do For You

With the prerequisites in place, DBCA can:

- Create CDB and non‑CDB databases  
- Configure both instance and database structures together  
- Set many database options up front  
- Delete a database and its physical files  
- Work with container databases:
  - Create PDBs  
  - Drop PDBs  
  - Use the seed PDB as a base

It can be invoked in two modes:

- **Interactive** – GUI pages, you click through  
- **Silent** – command line with options, no prompts

---

## 4. DBCA in Silent Mode (Non‑Interactive)

Silent mode is great for scripted builds, but terrible for live demos because nothing happens on screen except a blinking cursor and eventual success.

Typical silent create:

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

Or to delete a database:

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
- DBCA does the rest without asking follow‑up questions

---

## 5. DBCA in Interactive Mode – Wizard Walkthrough

Now for the version you can show in a classroom without everyone falling asleep.

### 5.1 Start DBCA and pick an action

From a properly set environment:

```bash
dbca
```

First screen: choose what you want to do:

- Create a database  
- Configure database options  
- Delete a database  
- Manage templates  
- Manage pluggable databases

Choose **Create a database**, then move to advanced configuration to see all pages.

### 5.2 Instance type and template

You first decide:

- Type:
  - Single instance  
  - RAC  
  - RAC One Node
- Template / purpose:
  - Oracle‑supplied (for example General Purpose)  
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
  - Whether to pre‑create any PDBs, and their names (for example `ronpdb`)

### 5.4 Storage – file system vs ASM and locations

You choose:

- Storage type:
  - File system  
  - ASM (if configured)  
- Whether to use locations from the template (Oracle Flexible Architecture style), or pick custom paths

You also configure:

- Fast recovery area:
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

### 5.6 Options – Database Vault, Label Security, and friends

You can choose to enable:

- Oracle Database Vault  
- Label Security  
- Other options, depending on edition and licensing

Remember: the installer often puts the binaries on disk whether you are licensed or not; enabling features is on you.

### 5.7 Memory and sizing

Instance settings:

- Use Automatic Memory Management (AMM) or manually set components  
- Set overall memory target (for example 1500 MB)  
- Set process count (maximum number of sessions / processes)

If AMM is not allowed for your platform configuration, DBCA will force you to adjust the settings accordingly.

### 5.8 Character sets and connection mode

You pick:

- Database character set (for example `AL32UTF8`)  
- National character set

Connection mode:

- Dedicated server connections  
- Shared server connections

DBCA presents these as radio buttons, but you can later configure both via parameters if needed.

Sample schemas:

- You can choose to install the sample schemas (for example the HR schema) to quickly validate the database and demo features

### 5.9 Management – EM Express and/or Cloud Control

EM Express:

- Enable EM Express for this database  
- Choose a port (for example 5501)

Cloud Control:

- Optionally register the database with an existing EM Cloud Control installation during creation

### 5.10 Passwords

You set passwords for:

- `SYS`  
- `SYSTEM`  
- And optionally PDB admin users

You can:

- Use the same password for all (quick for demos, bad for production)  
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
- Creates dictionary views, dictionary objects and sample schemas (if selected)  
- Configures EM Express, registration with listener, and optional Cloud Control

If you are short on disk space, DBCA will complain early, sparing you from watching a progress bar fail at 93%.

---

## 7. Summary – Let The Tool Do The Boring Parts

By now you should be able to:

- Plan a database creation: storage layout, memory, options, character sets, CDB/PDB design  
- Understand what DBCA can and cannot do (it builds databases; it does not install software or network)  
- Use DBCA:
  - In **silent mode** for scripted, repeatable builds  
  - In **interactive mode** for guided, fully specified builds  
- Make sensible choices about:
  - Templates and workload types  
  - CDB vs non‑CDB, PDB naming and undo  
  - Character sets and sample schemas  
  - EM Express and Cloud Control integration

In short: you now know how to let DBCA handle the heavy lifting of database creation while you focus on the parts that actually require judgment, like “where should this go” and “what on earth are we going to call it.” That alone puts you ahead of a distressing number of installations in the wild.

