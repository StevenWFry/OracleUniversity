# 4 – Creating a Database Using SQL (For When You Enjoy Pain, Apparently)

And look, you now know how to let DBCA build a database while you sit back and judge its progress bar. But sometimes you need (or are forced) to create a database the old‑fashioned way: with a parameter file, `STARTUP NOMOUNT`, and a truly unreasonable `CREATE DATABASE` statement.

This chapter walks through that process so it is *predictable* rather than *terrifying*.

---

## 1. Big Picture: What Happens When You `CREATE DATABASE`

When you run:

```sql
CREATE DATABASE ...
```

Oracle does not magically pop a database into existence. Under the covers:

- The running **instance** calls internal packages
- Those packages:
  - Create the **control file(s)**
  - Create the **redo log file(s)**
  - Create the **datafiles** for SYSTEM, SYSAUX, UNDO, TEMP, and whatever else you specified
- If you told it to `CREATE DATABASE ... ENABLE PLUGGABLE DATABASE`:
  - That database becomes a **CDB root**
  - A **PDB seed** can be created at the same time or later
- If you do **not** enable pluggable databases:
  - You get a traditional non‑CDB

Creating the database with SQL is really just telling an already‑running instance, “Here is the full blueprint; build this.”

---

## 2. Prerequisite: A Parameter File and an Instance

You do **not** run `CREATE DATABASE` against thin air. You need:

1. A **parameter file** describing the instance
2. A **started instance** (NOMOUNT) using that parameter file

### 2.1 The parameter file (pfile / spfile)

Traditionally:

- On Linux/UNIX:

  - Text pfile:  
    `ORACLE_HOME/dbs/init<DB_NAME>.ora`
  - Binary spfile:  
    `ORACLE_HOME/dbs/spfile<DB_NAME>.ora`

- On Windows: equivalent files under `%ORACLE_HOME%\database`

The bare minimum to get an instance going is almost comically small:

```ini
db_name=orcl
```

Everything else can default. In reality you will also set things like:

- `control_files`
- `db_create_file_dest`
- `db_recovery_file_dest` / size
- `enable_pluggable_database`

But Oracle really only *requires* `db_name` to start an instance.

### 2.2 Converting between spfile and pfile

Because spfiles are binary, they are:

- Great for the database to manage itself
- Terrible for humans to read

From SQL*Plus as SYSDBA:

```sql
CREATE PFILE FROM SPFILE;
```

This writes a readable `init<DB_NAME>.ora` into `$ORACLE_HOME/dbs`. You can:

- Open it with `cat` or an editor
- Copy the relevant settings into a new pfile for a **brand‑new** database

The instructor demo showed exactly this:

1. `CREATE PFILE FROM SPFILE;`
2. `cd $ORACLE_HOME/dbs`
3. `ls` to see:
   - `spfileORCL.ora` (binary)
   - `initORCL.ora`  (text copy)
4. `cat initORCL.ora` to inspect/steal useful parameters

If you are building a database by hand, you would usually:

- Create `initNEWDB.ora` yourself
- Put at least `db_name=NEWDB` in there
- Add any extra parameters you want under your control

---

## 3. Starting the Instance (NOMOUNT)

Once you have a pfile for the new database:

1. Set the environment so Oracle knows which DB you mean:

   ```bash
   export ORACLE_SID=newcdb
   export ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
   cd $ORACLE_HOME/dbs
   ```

2. Edit `initnewcdb.ora` until you are happy with it.

3. Start the instance using that pfile:

   ```bash
   sqlplus / as sysdba

   SQL> STARTUP NOMOUNT PFILE='$ORACLE_HOME/dbs/initnewcdb.ora';
   ```

At this point you have:

- An **instance** (memory + background processes)
- **No** control files or datafiles yet

Now you can issue `CREATE DATABASE`.

---

## 4. The `CREATE DATABASE` Statement (a.k.a. The Wall of Text)

The simplest possible version looks like this:

```sql
CREATE DATABASE newcdb;
```

It will:

- Use sensible defaults
- Place files in default locations
- Create a non‑CDB

But you almost never do that. Real‑world commands look more like:

```sql
CREATE DATABASE cdb1
  USER SYS IDENTIFIED BY "Sys_4U"
  USER SYSTEM IDENTIFIED BY "System_4U"
  ENABLE PLUGGABLE DATABASE
  LOGFILE GROUP 1 ('/u02/oradata/cdb1/redo01.log') SIZE 200M,
          GROUP 2 ('/u02/oradata/cdb1/redo02.log') SIZE 200M,
          GROUP 3 ('/u02/oradata/cdb1/redo03.log') SIZE 200M
  CHARACTER SET AL32UTF8
  NATIONAL CHARACTER SET AL16UTF16
  EXTENT MANAGEMENT LOCAL
  DATAFILE '/u02/oradata/cdb1/system01.dbf' SIZE 1G AUTOEXTEND ON
  SYSAUX DATAFILE '/u02/oradata/cdb1/sysaux01.dbf' SIZE 600M
  DEFAULT TEMPORARY TABLESPACE temp
    TEMPFILE '/u02/oradata/cdb1/temp01.dbf' SIZE 200M
  UNDO TABLESPACE undotbs1
    DATAFILE '/u02/oradata/cdb1/undotbs01.dbf' SIZE 400M
  SEED
    FILE_NAME_CONVERT (
      '/u02/oradata/cdb1/', '/u02/oradata/cdb1/pdbseed/'
    );
```

Key points:

- `ENABLE PLUGGABLE DATABASE` makes this a CDB, not a dinosaur
- You explicitly control:
  - Redo log groups and sizes
  - Character sets
  - SYSTEM/SYSAUX/TEMP/UNDO tablespaces
  - Where the PDB seed files go

This is exactly the sort of thing DBCA generated for you in the previous chapter. Now you see the raw SQL.

---

## 5. File Location Parameters: Let the Instance Help You

Typing full paths into `CREATE DATABASE` is a great way to make mistakes. A few parameters can save you:

### 5.1 `DB_CREATE_FILE_DEST`

Defines a default location for Oracle‑managed datafiles and tempfiles:

```ini
db_create_file_dest = '/u02/oradata/cdb1'
```

With this set, you can omit the file names in `CREATE DATABASE` and Oracle will create appropriately named files in that directory.

### 5.2 `PDB_FILE_NAME_CONVERT`

When you create pluggable databases (either the seed or later PDBs), you often want their files in a different directory tree. This parameter says:

> When you see files from path A, place the PDB files under path B instead.

Example:

```ini
pdb_file_name_convert =
  '/u02/oradata/cdb1/', '/u02/oradata/cdb1/pdbseed/'
```

Now when you create the seed or a PDB, Oracle rewrites file names according to that mapping.

### 5.3 `FILE_NAME_CONVERT`

This is a similar idea, but specified **in the DDL** (for example, in a `CREATE DATABASE ... SEED FILE_NAME_CONVERT(...)` clause or when cloning/creating PDBs) instead of as a global instance parameter.

Use it when:

- You want a one‑off mapping
- You do not want to change the instance‑wide defaults

---

## 6. After `CREATE DATABASE`: What Next?

If `CREATE DATABASE` completes successfully, you still have some housekeeping:

1. Open the database and make sure it looks sane:

   ```sql
   ALTER DATABASE OPEN;

   SELECT name, cdb, open_mode FROM v$database;
   ```

2. Run the catalog scripts (if your environment does not do this automatically):

   ```sql
   @?/rdbms/admin/catalog.sql
   @?/rdbms/admin/catproc.sql
   ```

3. Create a user tablespace (`USERS`), roles, and any additional tablespaces you want.
4. Configure EM Express, archive logging, and other options as needed.

This is exactly why DBCA is popular: it does all of this without making you remember script names at 3 a.m.

---

## 7. The Parameter‑File Demo (What You Saw on Screen)

The demo at the end of the chapter was there to demystify the parameter file:

1. Create a text pfile from the current spfile:

   ```sql
   SQL> CREATE PFILE FROM SPFILE;
   ```

2. Change to the database parameter directory:

   ```bash
   cd $ORACLE_HOME/dbs
   ls
   ```

   You will see things like:

   - `spfileORCL.ora` – binary server parameter file
   - `initORCL.ora`  – text copy we just created

3. Display the pfile:

   ```bash
   cat initORCL.ora
   ```

   You will see lines for `db_name`, control files, memory settings, and so on.

If you were building a new database manually, you would create a similar `initNEWDB.ora` by hand, set at least `db_name`, start an instance with it, and then run your `CREATE DATABASE` command.

---

## 8. Summary – Manual Creation Without Losing Your Mind

By now you should be able to:

- Explain what **really** happens when you run `CREATE DATABASE`
- Describe the role of the pfile/spfile and where they live
- Start an instance in `NOMOUNT` using a custom parameter file
- Read and adapt a `CREATE DATABASE` command that:
  - Enables pluggable databases
  - Defines redo, undo, temp, and default tablespaces
  - Places files in sensible locations
- Use `DB_CREATE_FILE_DEST`, `PDB_FILE_NAME_CONVERT`, and `FILE_NAME_CONVERT` to control file placement

You also know why DBAs tend to let DBCA generate this monster for them and only drop down to SQL when they absolutely have to. But if you *do* have to, you now have a plan instead of a headache.

