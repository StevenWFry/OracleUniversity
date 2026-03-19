## Lesson 13 - Managing Databases in Virtualized DB Systems (where the appliance finally lets you do database work inside the DB system and then immediately asks whether you chose the right shape)

Welcome to the final chapter, where Oracle Database Appliance stops talking about the DB system as an abstract container and starts talking about the thing everyone actually cares about: the databases inside it, their shapes, their workloads, their patching order, and the deeply Oracle-flavored reality that one wrong sizing choice can turn "performance tuning" into "performance archaeology."

By the end of this lesson, you should be able to:

- Explain how database shapes relate to workload classes in a DB system
- Create a database in a DB system during provisioning or afterward
- View, modify, and manage databases inside a DB system
- Understand the current patching and update flow for DB homes and databases in a DB system

---

## 1. Database Shapes and Workload Classes (because Oracle prefers templates to improvisation)

Database shapes are Oracle's predefined sizing templates for databases on ODA.

They exist so you do not size every database by hand like a medieval blacksmith guessing how much iron a cartwheel needs.

Current ODA database creation metadata centers on three workload classes:

- `OLTP`
- `DSS`
- `IMDB`

And the database shape itself is chosen from the `odb...` family, such as:

- `odb1s`
- `odb1`
- `odb2`
- `odb4`
- `odb8`
- up through larger shapes such as `odb64`, depending on model and release support

The logic is straightforward:

- First identify the workload class
- Then estimate the CPU cores and memory profile needed
- Then select the shape that best matches the workload

Oracle strongly recommends using these shapes because they encode ODA performance best practices rather than leaving you to invent a custom sizing plan and then discover you accidentally built a beautifully optimized disaster.

### How the workload classes differ

**`OLTP`**

- For transaction-heavy systems
- Optimized around balanced SGA/PGA and responsive commit-driven activity

**`DSS`**

- For reporting, warehousing, and scan-heavy workloads
- Sized differently for decision-support patterns

**`IMDB`**

- For workloads designed to benefit from In-Memory features
- Allocates memory differently to favor in-memory usage

One practical reminder from Oracle's current docs:

- Database shapes are still database-level choices
- DB system memory and DB system CPU are separate platform-level constraints

So even in a DB system, the database shape does not live in a magical alternate universe. It still has to fit inside the real resources available to that DB system.

---

## 2. What Must Be Ready Before You Create a Database in a DB System (because the database does not appear by positive thinking)

Before creating a database in a DB system, make sure the foundation is already in place.

At minimum, that means:

- The DB system already exists
- The bare metal repository is updated with the correct Oracle Database clone files for the version you want
- The relevant DB home exists or can be created from the staged clone files
- The DB system has sufficient CPU and memory available for the new database

Current Oracle docs are very direct on the repository point:

- do not create the database first and hope the clone files will catch up later
- stage the required software first

Representative repository-side command on the bare metal host:

```bash
odacli update-repository -f /tmp/odacli-dcs-db-clone-file.zip
```

And to confirm DB-system version support more generally:

```bash
odacli describe-dbsystem-image
```

That command is still Oracle's preferred way of saying, "please stop assuming the version matrix and just ask the appliance."

---

## 3. Creating a Database During DB System Creation or Afterward (two paths, same database-shaped consequences)

There are two normal ways to create a database in a DB system.

### Option 1: Create the database as part of DB system creation

This happens during the DB system provisioning flow, on the `Database Information` section.

This is useful when:

- You already know the target workload
- You already know the shape
- You already know the network, naming, and storage inputs
- You want the DB system to arrive with a database already in place

### Option 2: Create the database after the DB system already exists

This is the path you use when:

- The DB system is already running
- You want to add another database later
- You need to stage additional software first
- You want tighter control over timing and resource consumption

Current Oracle releases support multiple databases inside a DB system.

One important version nuance:

- Prior to ODA release `19.23`, a DB system supported only one database
- Starting with `19.23`, new and properly updated DB systems can support multiple databases and multiple DB homes
- Oracle's DB-system FAQs note that multi-database-enabled DB systems use `dbsX`-style shapes, while older single-database DB systems may still appear with `odbX`-style shapes

That is useful, but it comes with a very obvious engineering consequence:

- every new database still consumes real DB system resources

So before creating another database inside an existing DB system, check the boring but important things:

- CPU availability
- DB system memory
- storage consumption
- workload overlap
- whether the host and DB system still have sane headroom

Because "it let me click Create" and "this was a good idea" are not the same statement.

---

## 4. Creating a Database in the BUI (the guided route to database existence)

The BUI flow for a DB system database looks very similar to bare metal database creation, just within the DB system context.

The general sequence is:

1. Open the Browser User Interface
2. Navigate to the database area for the DB system
3. Click `Create Database`
4. Enter the required database information
5. Submit the job and monitor it in `Activity`

Typical inputs include:

- Database name
- Database version
- Database edition
- Database class
- Database shape
- Database type such as `SI`, `RAC`, or `RACOne` where supported
- Storage selection
- Character set and language settings
- Admin password and optional TDE choices
- Network association where applicable

If you create the database during DB system creation, these values are filled in during the DB system wizard instead of the later standalone database create flow.

Either way, Oracle expects the same discipline:

- pick the right workload class
- pick the right shape
- confirm the software is staged first
- do not treat database creation like ordering lunch from a drop-down menu

---

## 5. Creating a Database with `odacli` (for people who prefer identifiers and flags over screenshots)

The current command set still follows the familiar ODA pattern:

- `create-database`
- `list-databases`
- `describe-database`
- `modify-database`
- `move-database`
- `update-database`

Representative commands:

```bash
odacli list-databases
odacli describe-database -i DATABASE_ID
odacli create-database -n salesdb -dh DB_HOME_ID -cl OLTP -s odb4
```

That example is intentionally minimal.

Real deployments often add more options for:

- backup configuration
- associated networks
- storage mode
- database type
- TDE
- PDB creation
- version selection

So the practical workflow is:

1. Make sure the required clone files are staged
2. Make sure the target DB home is available
3. Create the database with the correct class and shape
4. Track the job

This is not glamorous work, but it is the work that prevents you from creating a DSS database with OLTP assumptions and then blaming the hardware for your own optimism.

---

## 6. Managing and Modifying a Database in a DB System (because requirements change, and so do bad ideas)

Once the database exists, you can manage it through the same BUI `Database` area and related `Actions` menu.

The transcript's summary is directionally right:

- open the database
- click `Actions`
- choose `Modify`
- change supported parameters
- submit the job

Current Oracle database-management guidance allows modifications such as:

- Database class
- Database shape
- Associated networks
- Backup policy
- Database redundancy in supported configurations
- TDE wallet management changes in supported releases
- TDE key rotation in supported workflows

Representative CLI pattern:

```bash
odacli modify-database -i DATABASE_ID -s odb8 -cl DSS
```

You can also move a database to another database home of the same base version when needed:

```bash
odacli move-database -i DATABASE_ID -dh DEST_DBHOME_ID
```

That becomes important because in modern ODA patching, homes are patched out-of-place.

Which means lifecycle management is no longer just "change one knob and carry on."

Sometimes the real job is:

- patch or create the right destination home
- then move or update the database into it deliberately

---

## 7. Resource Management Reality Inside a DB System (the part where the appliance reminds you math still applies)

The transcript says additional databases can be created inside an existing DB system if enough CPU and memory are available.

That is correct, and Oracle's current documentation adds some useful reality around it:

- DB systems consume host huge pages and CPU resources
- multiple databases in one DB system share that DB system's finite memory and CPU
- as a rule of thumb, the combined database shapes running inside the DB system should not exceed the DB system shape
- changing DB system shape no longer automatically reshapes the databases inside it on newer releases
- decreasing DB system memory without adjusting database memory first can cause databases to fail to start

So if you are managing more than one database inside the same DB system, keep the following separate in your head:

- DB system sizing
- database shape
- actual database memory settings

If you mix those up, the appliance will eventually educate you, and it will do so in the least charming way possible.

---

## 8. Updating Databases in a DB System (the patching sequence, or Oracle's preferred order for avoiding regret)

Patching in a DB system follows the same general philosophy as bare metal:

- stage software first
- run prechecks
- patch in the supported order
- do not freestyle the process

### What must be in place first

For current releases, Oracle documents these prerequisites:

- The relevant patch content must be staged in the bare metal repository
- The DB system image and infrastructure patches must match the DB system release path
- The required database clone files for the target database version must be in the repository

For example, Oracle's February 24, 2026 release notes for `19.30.0.0.0` list:

- the bare metal server patch
- the bare metal GI clone
- the bare metal RDBMS clone
- the DB system image for KVM
- and, for `26ai` DB systems, separate DB-system-specific GI, database clone, and DB system server patch files

So the current story is slightly more precise than "just upload a patch bundle and pray."

### The practical order

Oracle's documented flow for DB systems is:

1. Update the relevant repository content on the bare metal host
2. Update DB system infrastructure in the supported order
3. Patch Oracle Grid Infrastructure in the DB system
4. Run prepatch checks for DB homes or databases
5. Patch DB homes and/or update specific databases

### Representative current commands

On the bare metal host:

```bash
odacli update-repository -f /tmp/odacli-dcs-db-clone-file.zip
```

Inside the DB system:

```bash
odacli create-prepatchreport --dbhome --dbhomeid DB_HOME_ID -v target_release
odacli update-dbhome --id DB_HOME_ID -v target_release
```

Or for a specific database update into a chosen destination home:

```bash
odacli create-prepatchreport --database --database-id DATABASE_ID --to-home DEST_DBHOME_ID
odacli update-database --id DATABASE_ID --to-home DEST_DBHOME_ID
```

This is the key distinction:

- `update-dbhome` patches the database home and can move all databases from one source home to one destination home
- `update-database` targets a specific database and moves it into a chosen destination home

That is useful when you want control instead of a broad "move everybody together" operation.

### What to keep in mind

The transcript says infrastructure and database can be updated independently in a DB system.

That is directionally true, but the modern practical reading is:

- Oracle documents infrastructure-first patching order
- Database homes and databases are then patched with their own prechecks and jobs
- You should read the release notes and known issues before patching
- You should not skip the prepatch reports unless your preferred maintenance strategy is improvisational failure

---

## 9. Key Takeaways (the final revision sheet before the course stops politely feeding you answers)

- Database shapes on ODA map workload classes like `OLTP`, `DSS`, and `IMDB` to predefined Oracle sizing templates
- Creating a database in a DB system requires the right clone files to be staged first
- A database can be created during DB system provisioning or later inside an existing DB system
- Managing a database in a DB system uses the same broad ODA ideas as bare metal: create, view, modify, move, update
- Multiple databases can exist in one DB system, but they still compete for real CPU and memory
- On current ODA releases, DB system sizing changes do not automatically reshape the databases inside the DB system
- Patching databases in a DB system requires repository staging on bare metal plus prepatch and update work inside the DB system
- `update-dbhome` and `update-database` are related but not interchangeable tools

---

## 10. Wrap-Up (the databases have now officially entered the virtualized chat)

You now have the full DB-system database-management picture: how shapes map to workloads, what has to be staged before creation, how databases are created and modified inside a DB system, and how the modern patching flow separates infrastructure, DB homes, and individual databases. At this point the course has completed the full arc from appliance overview to actual operational database life inside ODA, which is Oracle's way of saying the training wheels are off and the maintenance windows are now spiritually your problem.
