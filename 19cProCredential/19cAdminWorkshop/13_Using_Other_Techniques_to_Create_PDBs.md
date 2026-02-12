# 13 - Using Other Techniques to Create PDBs (Clone, Plug, Relocate, Then Explain It In Change Control)

Last chapter covered creating empty PDBs from seed. This chapter is the
practical expansion pack: cloning existing PDBs, migrating non-CDBs, relocating
with near-zero downtime, and using proxy PDBs so remote data looks local.

---

## 1. Big Picture: What Changes and What Does Not

When you clone/unplug-plug a PDB:

- source generally stays available unless you explicitly drop/relocate it
- target gets a copied or linked representation depending on method

When you relocate a PDB:

- source is moved to target (not just duplicated)
- source is removed after successful relocation finalization

Do not treat clone and relocation as the same operation. They are not, and your rollback plan depends on that sentence being true.

---

## 2. Clone from Another PDB

Core syntax pattern:

```sql
CREATE PLUGGABLE DATABASE new_pdb
FROM source_pdb
CREATE_FILE_DEST = '/target/path';
```

Alternative file handling:

- use `FILE_NAME_CONVERT` instead of `CREATE_FILE_DEST`

After creation:

```sql
ALTER PLUGGABLE DATABASE new_pdb OPEN;
```

Simple, scriptable, and usually preferable to giant GUI workflow if you already
know what you are cloning and where files should go.

---

## 3. Hot Clone and Near-Zero Relocation via DB Link

Hot operations depend on:

- DB link between source and target
- local undo enabled on both sides

Why it matters:

- source can stay active during clone/relocation
- long-running transactions are handled with timeout behavior
- relocation can preserve application continuity better than cold cutovers

---

## 4. Non-CDB to PDB Migration Options

You can migrate non-CDB using multiple paths:

- create empty PDB, then Data Pump export/import
- create empty PDB, replicate with GoldenGate
- use `DBMS_PDB` conversion flow (describe/unplug metadata, then plug)
- DB link-based create/convert workflows

After non-CDB conversion, run non-CDB to PDB conversion steps to finalize
dictionary compatibility.

---

## 5. XML Unplug/Plug vs `.pdb` Archive

XML unplug:

- moves metadata definition only
- datafiles are copied/referenced separately (depending on copy mode)

`NOCOPY` behavior:

- metadata is plugged, datafiles are not copied to new location

`.pdb` archive unplug:

- bundles metadata + data payload together
- can be convenient for smaller migrations where single-file portability helps

---

## 6. DBCA vs SQL Commands

DBCA can perform clone/relocate workflows, but the lesson calls out an
important practical point:

- direct `CREATE PLUGGABLE DATABASE ...` commands are often shorter and clearer
- DBCA is optional for these tasks, not mandatory

Use DBCA when you need guided flow; use SQL when you need precise scripted
control and fewer clicks standing between you and the result.

---

## 7. Refreshable Clones

With DB link clone syntax, you can create refreshable copies:

- `REFRESH MODE MANUAL`
- scheduled auto refresh windows

Use case:

- reporting/test copies that follow source changes without full re-clone

In newer releases (18c+ context from lecture), refreshable clone patterns can
also participate in standby-like operational strategies.

---

## 8. Relocation Warnings Worth Remembering

Relocation workflow:

1. duplicate/move to target
2. finalize
3. source removed automatically on success

Operational best practice:

- validate target first
- keep rollback path in mind
- never assume nothing can fail mid-stream

Or, in technical terms: plan for Murphy, because Murphy has excellent attendance.

---

## 9. Proxy PDB and App Container Direction

Proxy PDB lets you expose remote PDB data locally through DB link plumbing:

- app SQL can read as if data were local
- cross-CDB distribution becomes transparent to application code

This is especially useful in application container designs where partitions live
in different regions/CDBs but must appear unified for global app behavior.

---

## 10. Key Takeaways

- Seed creation is only one method; real environments rely on clone/migrate/relocate patterns.
- DB link + local undo are foundational for hot clone and hot relocation.
- Relocation moves; cloning duplicates.
- XML and `.pdb` unplug methods serve different transport needs.
- Proxy PDB is how remote can behave local without rewriting every query.
