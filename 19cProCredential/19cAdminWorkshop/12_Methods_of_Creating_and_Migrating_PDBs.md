# 12 - Methods of Creating and Migrating PDBs (Clone It, Move It, Proxy It, Try Not To Typo It)

This lesson continues the multitenant journey: how to move PDB workloads, clone
them, and expose remote PDB data locally through proxy PDBs, which is surprisingly elegant for something involving this many links.

---

## 1. Proxy PDB Concept (Remote Looks Local)

A proxy PDB lets a local CDB expose a remote PDB through a database link so
sessions can query it as if it were local.

Flow in plain English:

- create DB link between CDB environments
- create proxy PDB in local CDB using that link
- point proxy to target remote PDB
- open proxy PDB and query normally

Result:

- SQL/session logic can treat remote data like local PDB data
- location complexity is hidden behind proxy + DB link plumbing

---

## 2. Prerequisites Mentioned for Remote Proxying

The lesson calls out key requirement:

- archiving enabled on both participating CDBs

Then:

- establish DB link
- ensure remote target PDB is operational
- create proxy on requesting CDB

---

## 3. Where Proxying Is Used

Two common patterns:

- one-off remote PDB access from another CDB
- application-container partitioning across regional CDBs, surfaced through proxy links

This supports global distribution while keeping application-facing SQL simpler.

---

## 4. Clone and Migration Methods Recap

From this and prior lesson context, core options include:

- create from seed (empty template-based PDB)
- clone existing PDB (full metadata/data copy)
- migrate/convert and plug from non-CDB paths
- remote access via proxy PDB + DB link

---

## 5. Demo Focus: Cloning `PDB1` to `PDB10`

The transcript demo uses clone syntax from an existing PDB and places files in
a dedicated destination path.

Representative command pattern:

```sql
CREATE PLUGGABLE DATABASE pdb10
FROM pdb1
CREATE_FILE_DEST = '/u01/app/oracle/oradata/ORCL/pdb10';
```

Then:

```sql
SHOW PDBS;
ALTER PLUGGABLE DATABASE pdb10 OPEN;
```

Validation query example:

```sql
SELECT table_name, owner
FROM   dba_tables;
```

Observation in demo:

- cloned PDB initially appears mounted, then opened
- dictionary/application objects from source PDB are present in clone

---

## 6. Practical Notes from the Demo

- destination path must be valid and correctly formatted
- clone command syntax is sensitive to quoting/path typos
- with local undo enabled, hot clone behavior is supported in modern multitenant flows

When clone statements fail, first suspects are usually:

- path/permission issues
- quoting/statement formatting
- incorrect file destination specification

---

## 7. Why This Matters Operationally

These methods give you options for:

- fast environment provisioning (dev/test copies)
- controlled workload movement between CDBs
- regional/data-locality patterns through proxy access
- scaling by partitioning data placement without rewriting app SQL heavily

Next logical step after creation/migration:

- validate services, open modes, and application connectivity against the new or proxied PDB path.

---

## 8. Proxy Sequence Across `CDB1` and `CDB2` (Detailed Flow)

Transcript sequence emphasized this practical order:

1. Ensure archiving is enabled on both CDBs.
2. Build DB link connectivity.
3. In `CDB2`, ensure remote target PDB is present and healthy.
4. In `CDB1`, create proxy PDB using the DB link and target remote PDB name.
5. Open proxy PDB and validate table access.

At that point, querying through proxy should behave like local PDB access from
application perspective.

---

## 9. Demo Troubleshooting Notes (Real Errors Seen)

The live demo showed common failure patterns while cloning `PDB1` to `PDB10`:

- using incomplete path in `CREATE_FILE_DEST`
- accidental extra quotes or blank characters
- typos in path prefixes (for example `/u01` vs mistyped variants)
- attempting clone before target directory exists

Working recovery pattern:

1. Create target OS directory first.
2. Recheck full absolute path.
3. Use clean command text with single, correct quoting.
4. Re-run clone command and confirm mounted -> open transition.

This is exactly why clone scripts should be saved and re-run from clean files
instead of repeatedly editing the same terminal line like you are defusing a bomb with one hand.
