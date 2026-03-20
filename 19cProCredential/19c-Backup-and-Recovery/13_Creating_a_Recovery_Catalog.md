## Lesson 13 - Creating a Recovery Catalog (in which RMAN gets its own database apartment instead of sleeping in the control file closet)

And look, creating a recovery catalog is not mystical. It is just a sequence: prepare a database, create a schema owner, give that owner the right privileges, and then tell RMAN to build the catalog metadata. Very administrative. Very Oracle. Very easy to get wrong if you skip one grant and then act shocked.

By the end of this lesson, you should be able to:

- Prepare a database to host the RMAN recovery catalog
- Create the recovery catalog tablespace and owner
- Grant the privileges required for the catalog owner
- Connect RMAN to the catalog database and run `CREATE CATALOG`
- Understand the practical sizing and placement decisions behind the catalog

---

## 1. What You Are Actually Creating

When you create a recovery catalog, you are not creating some magical new backup
engine.

You are creating:

- a user schema
- in a separate database
- that will store RMAN metadata for one or more target databases

So the work breaks down into three real steps:

1. prepare the recovery catalog database
2. create the catalog owner and its tablespace
3. connect through RMAN and run `CREATE CATALOG`

That is it.

The trick is that Oracle spreads this simple idea across enough prerequisites to
make it feel like a tax form.

---

## 2. Before You Start

Think about sizing first.

The recovery catalog does not usually exist for just one tiny database. It often
supports multiple target databases, which means it stores metadata about:

- backup sets
- image copies
- archived redo logs
- control-file-derived RMAN metadata
- stored RMAN scripts

So the sizing question is not:

- "How much space does one schema need?"

It is:

- "How much metadata will I accumulate across all target databases over time?"

The catalog database is often modest in size, but it should not be treated like
a disposable toy.

Also, current Oracle documentation notes:

- the recovery catalog database should be separate from production targets
- the base recovery catalog must not be created in `SYS`
- current catalog guidance assumes Enterprise Edition for the catalog database

---

## 3. Create the Tablespace for the Catalog Owner

First create a dedicated tablespace for the recovery catalog owner.

Typical example:

```sql
CREATE TABLESPACE rcattbs
  DATAFILE '/u01/app/oracle/oradata/rcatcdb/rcatpdb/rcat01.dbf'
  SIZE 15M
  REUSE;
```

Practical notes:

- use an explicit path you understand
- size it sensibly for your environment
- `REUSE` only makes sense if you truly mean to reuse an existing file

In a real environment, many DBAs would also consider `AUTOEXTEND` and storage
placement more carefully instead of trusting a tiny starter file to bravely
carry the whole enterprise on its back.

---

## 4. Create the Recovery Catalog Owner

After the tablespace exists, create the catalog owner.

Typical example:

```sql
CREATE USER rcatowner IDENTIFIED BY strong_password
  DEFAULT TABLESPACE rcattbs
  TEMPORARY TABLESPACE temp
  QUOTA UNLIMITED ON rcattbs;
```

Then grant the required role:

```sql
GRANT RECOVERY_CATALOG_OWNER TO rcatowner;
```

This is the core setup.

The default tablespace tells Oracle where the catalog objects will live.
The quota lets the schema actually create them instead of being theoretically
important but practically useless.

---

## 5. Extra Privileges Oracle Also Cares About

Current Oracle documentation adds an important detail that classroom summaries
often skip:

- you should run the `dbmsrmansys.sql` script from `$ORACLE_HOME/rdbms/admin`
  to grant the additional privileges required by the
  `RECOVERY_CATALOG_OWNER` role

Representative example as `SYS`:

```sql
@$ORACLE_HOME/rdbms/admin/dbmsrmansys.sql
```

This is one of those tiny Oracle details that loves to sit quietly in the
documentation until you discover "I granted the role" was not, in fact, the end
of the story.

---

## 6. Connect to the Catalog Database from RMAN

Once the owner exists, connect RMAN to the database that will hold the catalog.

You connect as the catalog owner, not as `SYS`.

Example:

```text
rman catalog rcatowner/strong_password@rcatpdb
```

Or from inside RMAN:

```rman
CONNECT CATALOG rcatowner/strong_password@rcatpdb;
```

Important point:

- a connection to the target database is not required just to run
  `CREATE CATALOG`

That is because, at this stage, you are building the catalog schema itself.
You are not registering target databases yet.

---

## 7. Create the Recovery Catalog

After connecting as the catalog owner, run:

```rman
CREATE CATALOG;
```

That command creates the recovery catalog tables in the owner's default
tablespace.

Oracle also supports naming the tablespace explicitly:

```rman
CREATE CATALOG TABLESPACE rcattbs;
```

The creation can take several minutes because Oracle is building the schema
objects that RMAN will use for catalog metadata.

And that is the whole dramatic moment:

- connect
- run `CREATE CATALOG`
- wait
- now RMAN has somewhere official to store its long-term memory

---

## 8. Verify That the Catalog Exists

After creation, you can verify the schema objects from SQL*Plus:

```sql
SELECT table_name
FROM   user_tables
ORDER BY table_name;
```

If the catalog was created successfully, you will see the catalog tables owned
by the recovery catalog schema.

This is not glamorous verification, but it beats just trusting vibes.

---

## 9. Practice-Style Flow

The practice version breaks into two parts:

### Part 1: create the owner

Representative lab flow:

```sql
CREATE TABLESPACE rcattbs
  DATAFILE '/u01/app/oracle/oradata/rcatcdb/rcatpdb/rcat01.dbf'
  SIZE 15M
  REUSE;

CREATE USER rcatowner IDENTIFIED BY cloud_4U
  DEFAULT TABLESPACE rcattbs
  QUOTA UNLIMITED ON rcattbs;

GRANT RECOVERY_CATALOG_OWNER TO rcatowner;
```

### Part 2: create the catalog

Representative RMAN flow:

```text
rman catalog rcatowner@rcatpdb
```

Then:

```rman
CREATE CATALOG;
```

That is the essential lab sequence, minus the side quests about environment
variables, idle instances, and the usual amount of classroom stage management.

---

## 10. Practical Design Notes

Use one base recovery catalog for many databases when possible.

Why?

- easier centralized management
- easier script reuse
- easier long-term metadata retention

Also remember:

- the catalog database should be protected like any other important database
- the catalog owner should not be a privileged general-purpose account
- this lesson creates the catalog itself, not the target registration

Registration is the next step after the catalog exists.

---

## 11. Wrap-Up

Creating a recovery catalog means:

- create a tablespace
- create a catalog owner
- grant the right privileges
- connect RMAN as that owner
- run `CREATE CATALOG`

Simple in theory, administrative in practice, and extremely useful once you are
managing more than one serious database and need RMAN to remember things longer
than a control file comfortably can.
