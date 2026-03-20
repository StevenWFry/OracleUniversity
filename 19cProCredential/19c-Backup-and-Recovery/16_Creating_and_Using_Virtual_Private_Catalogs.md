## Lesson 16 - Creating and Using Virtual Private Catalogs (in which one recovery catalog gets subdivided so not every DBA can see every toy in the cupboard)

And look, a virtual private catalog exists because sometimes you want one central recovery catalog, but you do not want every catalog user seeing or managing every registered database like it is an all-you-can-eat buffet of metadata.

By the end of this lesson, you should be able to:

- Explain what a virtual private catalog (`VPC`) is
- Create a VPC owner correctly
- Enable the base recovery catalog for VPC use
- Grant restricted catalog access to a VPC owner
- Use a VPC owner to work with an allowed target or PDB
- Revoke VPC access when it is no longer needed

---

## 1. What a Virtual Private Catalog Is

A virtual private catalog is a restricted view of a base recovery catalog.

That means:

- one base recovery catalog can serve many databases
- different users can be given access only to the databases or PDBs they are
  supposed to manage

So the catalog stays centralized, but access is divided.

This is useful when:

- different DBAs manage different systems
- you want separation of administrative scope
- you want one RMAN repository without turning it into a metadata free-for-all

In plain English:

- base catalog = the whole filing room
- VPC = a key to only certain cabinets

---

## 2. What the Source Material Gets Slightly Wrong

The lecture describes VPC creation like you build another recovery catalog for
the VPC owner.

That is not how modern Oracle does it.

For current Oracle releases:

- you create a **database user** for the VPC owner
- you grant that user `CREATE SESSION`
- the **base recovery catalog owner** grants catalog access for specific
  databases or PDBs
- the virtual private catalog is created automatically the first time the VPC
  owner connects to the recovery catalog

So no, you do not create a separate base catalog for each VPC user like some
kind of elaborate bureaucratic nesting doll.

---

## 3. Enable the Base Catalog for VPC Use

Oracle uses Virtual Private Database (`VPD`) functionality behind the scenes to
implement virtual private catalogs.

This is not enabled by default for an existing base recovery catalog.

To enable it, connect to the recovery catalog database as `SYS` with `SYSDBA`
and run the required script from `$ORACLE_HOME/rdbms/admin`.

The current Oracle documentation highlights:

```sql
@$ORACLE_HOME/rdbms/admin/dbmsrmansys.sql
```

Older training material may show a variant involving `dbmsrmanvpc.sql`.
The important point is:

- the base recovery catalog must be prepared for VPC support using the Oracle
  admin script Oracle documents for your release

After that, connect RMAN as the base recovery catalog owner and, if required for
your environment, upgrade the base catalog:

```rman
UPGRADE CATALOG;
```

Important Oracle rule:

- only the **base** catalog can be upgraded
- you cannot use `UPGRADE CATALOG` while connected to a VPC

So if somebody tries to "upgrade the VPC" directly, Oracle will treat that idea
with exactly the level of respect it deserves.

---

## 4. Create the VPC Owner

Create a normal database user in the catalog database to own the virtual private
catalog schema.

Example:

```sql
CREATE TABLESPACE vpctbs
  DATAFILE '/u01/app/oracle/oradata/rcatcdb/rcatpdb/vpc01.dbf'
  SIZE 5M
  REUSE;

CREATE USER vpcowner IDENTIFIED BY strong_password
  DEFAULT TABLESPACE vpctbs
  QUOTA UNLIMITED ON vpctbs;

GRANT CREATE SESSION TO vpcowner;
```

Important current Oracle nuance:

- starting with Oracle Database `12.1.0.2`, the VPC owner needs `CREATE SESSION`
  only
- older material that grants `RECOVERY_CATALOG_OWNER` to the VPC user is not
  current guidance for modern VPC setup

So yes, this part is actually simpler now, which is a suspiciously humane thing
for Oracle to have done.

---

## 5. Grant Catalog Access from the Base Catalog Owner

Now connect RMAN as the **base recovery catalog owner** and grant the VPC owner
access to specific metadata.

Examples:

```rman
GRANT CATALOG FOR DATABASE prod1 TO vpcowner;
GRANT CATALOG FOR PLUGGABLE DATABASE orclpdb1 TO vpcowner;
GRANT REGISTER DATABASE TO vpcowner;
```

What these mean:

- `GRANT CATALOG FOR DATABASE ...`
  VPC owner can access catalog metadata for that database
- `GRANT CATALOG FOR PLUGGABLE DATABASE ...`
  VPC owner can access catalog metadata for that PDB
- `GRANT REGISTER DATABASE ...`
  VPC owner may register databases in the catalog

Important Oracle nuance:

- if you grant `REGISTER DATABASE`, then catalog privileges are implicitly
  granted for databases that user registers

This is helpful, but it also means you should know exactly what you are
authorizing instead of scattering grants around like birdseed.

---

## 6. How the Virtual Private Catalog Is Actually Created

Once the VPC owner has catalog privileges, the VPC is created automatically the
first time that user connects to the catalog with RMAN.

Example:

```text
rman catalog vpcowner@rcatpdb
```

That first connection initializes the virtual private catalog schema for the
user.

So despite the classroom narration, there is no separate `CREATE CATALOG` step
for the VPC owner in current Oracle behavior.

The VPC appears because:

- the base catalog exists
- the user exists
- the user has appropriate access granted
- the user connects

Which is much cleaner than making every restricted user go build a tiny kingdom
of metadata from scratch.

---

## 7. Using the VPC Owner for Backup Work

After the VPC owner has access, that user can connect with:

- the target database or PDB they are allowed to manage
- their VPC connection to the recovery catalog

Example:

```text
rman target sysbackup@orclpdb1 catalog vpcowner@rcatpdb
```

Then they can run allowed RMAN work, for example:

```rman
BACKUP PLUGGABLE DATABASE orclpdb1;
LIST BACKUP;
```

The base catalog owner can still see the broader picture.
The VPC user sees only the slice they were granted.

Which is the whole point of the exercise.

---

## 8. Revoking VPC Access

If a user no longer needs access, revoke it from the base catalog owner session.

Examples:

```rman
REVOKE CATALOG FOR DATABASE prod1 FROM vpcowner;
REVOKE CATALOG FOR PLUGGABLE DATABASE orclpdb1 FROM vpcowner;
REVOKE REGISTER DATABASE FROM vpcowner;
```

This is useful when:

- a DBA changes responsibilities
- a user leaves the team
- the catalog scope needs tightening

Because "they probably won't use it anymore" is not a security model.

---

## 9. Upgrading Older VPC Setups

If you are dealing with an older recovery catalog and older virtual private
catalogs, Oracle provides scripts to upgrade them to the newer VPD-based model.

Current Oracle guidance for this path includes:

- run `dbmsrmansys.sql` as `SYS`
- upgrade the base recovery catalog
- then upgrade associated virtual private catalog schemas as documented for the
  release

The key point is:

- old VPC setups and modern VPC setups are not identical

So if lecture notes from older releases sound slightly off, that is not your
imagination. That is Oracle version drift, one of the database world's most
persistent hobbies.

---

## 10. Practice-Style Flow

The practice content boils down to three parts.

### Part 1: enable VPD functionality

As `SYS` on the catalog database:

```sql
@$ORACLE_HOME/rdbms/admin/dbmsrmansys.sql
```

Then as the base catalog owner in RMAN:

```rman
UPGRADE CATALOG;
```

### Part 2: create the VPC owner

```sql
CREATE TABLESPACE vpctbs
  DATAFILE '/u01/app/oracle/oradata/rcatcdb/rcatpdb/vpc01.dbf'
  SIZE 5M
  REUSE;

CREATE USER vpcowner IDENTIFIED BY cloud_4U
  DEFAULT TABLESPACE vpctbs
  QUOTA UNLIMITED ON vpctbs;

GRANT CREATE SESSION TO vpcowner;
```

Then, as the base catalog owner in RMAN:

```rman
GRANT CATALOG FOR PLUGGABLE DATABASE orclpdb1 TO vpcowner;
```

### Part 3: use the VPC owner

Connect with the allowed target and the VPC owner:

```text
rman target sysbackup@orclpdb1 catalog vpcowner@rcatpdb
```

Then:

```rman
BACKUP PLUGGABLE DATABASE orclpdb1;
LIST BACKUP;
```

That is the practical story:

- create restricted user
- grant access to a specific slice of catalog metadata
- let that user work only within that slice

Beautifully bureaucratic, which in this case is exactly what you want.

---

## 11. Practical Takeaways

Use VPCs when:

- one organization manages many databases through one base recovery catalog
- different DBAs should see only their assigned databases or PDBs
- central control matters, but broad visibility does not

Remember:

- the base recovery catalog owner controls the grants
- the VPC owner is just a restricted user, not a second base catalog owner
- modern VPC creation is simpler than the older lecture material suggests

That last point matters, because blindly following old syntax in Oracle is one
of the faster ways to create a very educational error message.

---

## 12. Wrap-Up

Virtual private catalogs let you divide one base recovery catalog into
restricted administrative scopes.

The real flow is:

1. enable VPC support on the base catalog
2. create the VPC owner user
3. grant `CREATE SESSION`
4. grant database or PDB catalog access from the base owner
5. let the VPC owner connect and work within that limited scope
6. revoke access later when needed

Which means you get centralized RMAN metadata without turning every catalog user
into an omniscient backup demigod.
