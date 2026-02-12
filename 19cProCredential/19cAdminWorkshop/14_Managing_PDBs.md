# 14 - Managing PDBs (Open, Close, Tune, Drop, Try Not To Drop The Wrong One)

Chapter 3 of this module focuses on day-to-day pluggable database management:
open/close behavior, parameter scope, storage controls, proxy settings, and safe
drop/relocation handling, which is where "just one quick change" becomes a very long evening.

---

## 1. PDB Open and Close Modes

Close options:

- `ALTER PLUGGABLE DATABASE ... CLOSE` defaults to immediate behavior
- `CLOSE ABORT` can require recovery on next open
- `CLOSE NORMAL` / transactional variants wait for session/transaction outcomes

Open options:

- `OPEN` (read write)
- `OPEN RESTRICTED` (only users with restricted session privilege)
- `OPEN READ ONLY` (query access, no data changes)

---

## 2. Operational Management Inside a PDB

Most administration patterns mirror non-CDB behavior when connected to a PDB:

- tablespace/datafile operations
- default and temp tablespace configuration
- user and local administration settings
- PDB storage limits (important with ASM capacity planning)

PDB global name/service identity can also be renamed through PDB management SQL.

---

## 3. PDB Parameters and Inheritance

Parameter model:

- PDB initially inherits from CDB defaults
- PDB-specific tuning is stored in PDB metadata/dictionary
- unplug/plug keeps PDB-local parameter behavior portable

Example concept from lecture:

- `ddl_lock_timeout` set to different values in different PDBs
- CDB root value remains unchanged

So parameter scope is container-aware:

- change in PDB affects that PDB
- change in root affects root-level/default behavior

---

## 4. Not Every `ALTER SYSTEM` Is PDB-Scoped

PDB-scoped commands affect only current container context.

CDB-scoped commands are still root-level operations, including examples like:

- redo log switching
- checkpoint behaviors tied to instance/CDB structures

Rule:

- if operation targets shared instance/CDB internals, do it in root, not from a PDB while hoping Oracle will "figure it out."

---

## 5. Proxy PDB Host/Port Settings

For proxy use cases, host/port properties can be set to distinguish proxy
routing to remote listener endpoints.

If custom settings are not needed, reset behavior can revert to default CDB
connection routing definitions.

---

## 6. Drop and Relocation Rules

Drop behavior:

- PDB must be closed first
- include datafiles to avoid orphaned file references:

```sql
DROP PLUGGABLE DATABASE <name> INCLUDING DATAFILES;
```

Relocation behavior:

- true relocation removes source automatically after successful move

Unplug/plug behavior:

- source is not automatically removed
- manual drop is required if you want relocation-like outcome

---

## 7. Demo Flow Captured in Lesson

The transcript demo sequence:

1. Show PDB state:

```sql
SHOW PDBS;
```

2. Close/open mode transitions:

```sql
ALTER PLUGGABLE DATABASE pdb1 CLOSE;
ALTER PLUGGABLE DATABASE pdb1 OPEN READ ONLY;
ALTER PLUGGABLE DATABASE pdb1 CLOSE;
ALTER PLUGGABLE DATABASE pdb1 OPEN;
```

3. Connect to PDB over service and set parameter:

```sql
-- example connection style
-- sys/<pwd>@pdb1 as sysdba
ALTER SYSTEM SET ddl_lock_timeout = 10 SCOPE=BOTH;
SHOW PARAMETER ddl;
```

4. Return to root and verify root value differs:

```sql
SHOW CON_NAME;
SHOW PARAMETER ddl;
```

5. Drop behavior demonstration:

- drop fails if PDB open
- drop fails without datafile clause
- succeeds only after close + `INCLUDING DATAFILES`

---

## 8. Key Takeaways

- PDB lifecycle commands are straightforward, but mode/scope matters.
- PDB parameter tuning is container-local and portable.
- CDB internals still require root-level operations.
- Relocation and unplug/plug are not the same operationally.
- Safe drops require closed PDB plus explicit datafile handling, because orphaned files are just clutter with a support ticket attached.
