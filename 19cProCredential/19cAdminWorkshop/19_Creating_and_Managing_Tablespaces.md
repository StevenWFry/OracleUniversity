# 19 - Creating and Managing Tablespaces (Because "Somewhere on Disk" Is Not Governance)

Chapter 2 of the storage module: how to create, alter, view, and drop
tablespaces without accidentally turning `SYSTEM` into your app's junk drawer.

---

## 1. Core Lifecycle: Create, Alter, Drop

Tablespace lifecycle operations:

- `CREATE TABLESPACE` (or `CREATE TEMPORARY TABLESPACE`, `CREATE UNDO TABLESPACE`)
- `ALTER TABLESPACE` for settings and file operations
- `DROP TABLESPACE` when no longer needed

Safety behavior:

- Oracle protects destructive operations on purpose
- if objects exist, you need explicit clauses like `INCLUDING CONTENTS` (and
 file clauses as applicable), so accidental deletion is harder than one typo

Because deleting half the database should require more effort than deleting one line in a text editor.

---

## 2. OMF (Oracle Managed Files) Simplifies the Boring Parts

With Oracle Managed Files, set destination parameters and Oracle handles file
naming/location defaults.

Key parameters called out:

- `db_create_file_dest` (datafiles)
- redo destination parameters (for online logs)

Then tablespace creation can be as simple as naming the tablespace while Oracle
does the repetitive plumbing.

Without OMF:

- you must specify file paths explicitly in DDL

---

## 3. Where to Create Tablespaces: Root vs PDB

At CDB root:

- usually DBA/admin support objects
- metadata/monitoring/operational helper objects

At PDB level:

- end-user application objects and schema data

Remember: root and PDB storage are different administrative neighborhoods. Do not pretend they are one giant suburb.

---

## 4. Default Permanent Tablespace Strategy

If you do nothing, users can end up defaulting into `SYSTEM` in some contexts.
That is bad.

Best practice:

- create explicit user-data tablespaces
- set default permanent tablespace at container level (`ALTER DATABASE ...`)
- optionally set per-user default (`ALTER USER ... DEFAULT TABLESPACE ...`)

Goal:

- keep metadata tablespaces for metadata
- keep user/application objects where they belong

---

## 5. Temporary Tablespace Strategy (More Than One Is Normal)

Oracle needs temp for internal operations, and users need temp for workload
operations. Do not lump every temporary behavior into one bucket and hope.

Design pattern from lecture:

- temp for Oracle/system operations
- temp (or temp groups) for user workloads
- separate temp for temp-table-heavy workflows

Temp tablespace groups can improve parallel temp behavior for heavy reporting/sort workloads.

---

## 6. Undo Strategy Note from Lecture

Lecture discussion also stresses planning undo behavior and switching strategies
when primary undo fills under pressure.

Operational theme:

- one default undo at a time
- monitor growth/retention behavior closely
- prepare controlled fallback/switchover behavior when capacity stress appears

The point is not "never fill undo." The point is "do not be surprised when it does."

---

## 7. Altering Tablespaces and Files

`ALTER TABLESPACE` handles:

- read write <-> read only state changes
- logging/nologging behavior
- add/resize/manage datafiles
- online/offline operations

Some operations can be run at file level directly, but tablespace-level control
is usually cleaner when managing policy/state for the whole logical unit.

---

## 8. Data Dictionary Views vs Performance Views

Metadata views tell you **what is defined**:

- `DBA_` views for current container
- `CDB_` views across all containers (from root)

Scope variants:

- `USER_*` = objects owned by current user
- `ALL_*` = objects current user can access
- `DBA_*` = all objects in current container
- `CDB_*` = all objects across containers

Performance (`V$`) views tell you **how it is behaving now**:

- I/O activity
- runtime usage/statistics
- operational efficiency signals

If dictionary views are the blueprint, `V$` views are the live traffic camera.

---

## 9. Datafile/Tempfile Visibility

You have corresponding view families for:

- datafiles
- tempfiles
- undo-related structures

Use dictionary views for static metadata and configuration.
Use performance views for runtime behavior and tuning evidence.

---

## 10. Practical Takeaways

- create tablespaces by workload behavior, not convenience.
- OMF reduces boilerplate and naming mistakes.
- never let user objects drift into `SYSTEM`.
- separate permanent/temp/undo strategy intentionally.
- use `DBA_`/`CDB_` for definition, `V$` for behavior.
- make destructive operations explicit and deliberate.

Next chapter continues with deeper operational examples and demos.
