# 5 – Managing Startup and Shutdown (Or, How Not To Unplug the Database Battery)

And look, now that you know how to **create** databases, we should probably talk about how to **start and stop** them without causing a small career‑ending incident.

Oracle has two big moving parts:

- The **instance** – memory + background processes
- The **database** – the set of physical files on disk

Startup and shutdown is about bringing those two into a consistent relationship, not just “turning it off and on again.”

---

## 1. Startup Phases: Saddling, Mounting, and Actually Riding

Oracle’s terminology is suspiciously close to riding a horse:

- First you put the **saddle** on
- Then you **mount** the horse
- Then you actually **ride**

Translate that to the database:

- `STARTUP NOMOUNT` – saddle on the horse (instance only)
- `MOUNT` – you are sitting on the horse, but not moving yet (database control file read)
- `OPEN` – you kick the horse and start riding (database available to users)

### 1.1 `STARTUP NOMOUNT` – only the instance

What happens:

- Reads the parameter file (pfile / spfile)
- Allocates the SGA and starts background processes (DBWn, LGWR, SMON, PMON, etc.)
- **Does not** touch control files or datafiles yet

Use this mode when:

- You want to create a brand‑new database with `CREATE DATABASE`
- You need to change control file locations/parameters before mounting
- You are doing very low‑level surgery as SYSDBA

Admins can connect, but **user data is not accessible** yet.

### 1.2 `MOUNT` – database attached, but closed

In this phase:

- Oracle opens and reads the **control file(s)**
- It checks the end‑of‑file flags to see how the database was last closed:
  - If the control file shows a clean shutdown, no recovery is needed
  - If the database crashed or was aborted, instance recovery is required

If a crash is detected, `SMON` starts **instance recovery** while the DB is still mounted.

At `MOUNT`:

- A DBA can:
  - Perform media recovery (`RECOVER DATABASE`)
  - Change some file names/paths
  - Switch between open modes (for example to read‑only)
- Regular users still cannot access objects.

### 1.3 `OPEN` – database available

Once the control file checks out and required recovery is done, you can:

```sql
ALTER DATABASE OPEN;
```

Now:

- Datafiles and redo logs are open
- Normal sessions can connect and do work

This is your normal operating state.

---

## 2. Open Modes: Read‑Write, Read‑Only, Restricted

Oracle gives you several ways to open the database; some are friendlier than others.

### 2.1 Read‑write (default)

Standard `STARTUP` or `ALTER DATABASE OPEN`:

- Users can read and modify data (depending on privileges)
- DML and DDL are allowed

### 2.2 Read‑only

Used when you want to let people look but not touch.

Sequence:

```sql
SHUTDOWN IMMEDIATE;
STARTUP MOUNT;
ALTER DATABASE OPEN READ ONLY;
```

In read‑only mode:

- Queries are allowed
- No DML/DDL changes are permitted

Useful for:

- Reporting replicas
- Taking consistent snapshots

### 2.3 Restricted mode

Sometimes you want the database up, but only for privileged users.

```sql
STARTUP RESTRICT;
-- or, if already open
ALTER SYSTEM ENABLE RESTRICTED SESSION;
```

Only users with the `RESTRICTED SESSION` privilege (typically DBAs) can connect.

Use this for:

- Maintenance windows
- Structural changes you do **not** want running alongside end‑user traffic

---

## 3. Shutdown Modes: From Polite to “Rip the Cable Out”

Shutdown is where DBAs reveal whether they are patient, cautious, or secretly arsonists.

### 3.1 `SHUTDOWN NORMAL`

Behavior:

- New connections are **refused**
- Existing sessions are allowed to **finish and disconnect voluntarily**
- The database only shuts down *after* the last session logs out

You will almost never use this on purpose:

- If one user leaves a forgotten session open all weekend, your shutdown waits all weekend.

### 3.2 `SHUTDOWN TRANSACTIONAL`

Behavior:

- New connections are refused
- Sessions **without** active transactions are disconnected
- Sessions **with** active transactions are allowed to:
  - Finish their transactions
  - Commit or roll back
  - Then they are disconnected

Use this when:

- You need the database down
- But you absolutely must not cut off important transactions mid‑flight

### 3.3 `SHUTDOWN IMMEDIATE` (the adult choice)

This is the **normal, recommended** shutdown.

Behavior:

- New connections are refused
- Existing sessions are **killed**
- Any active transactions are **rolled back**
- The database closes and dismounts cleanly

On the next startup:

- No instance recovery is required
- `STARTUP` goes straight through NOMOUNT → MOUNT → OPEN without drama

### 3.4 `SHUTDOWN ABORT` (the nuclear option)

Behavior:

- New connections are refused
- The instance is **killed immediately**
- Oracle does **not**:
  - Roll back transactions
  - Close datafiles cleanly
  - Release all resources

The database is left in a *crashed* state. On the next startup:

- Oracle must run **instance recovery**
- If redo is missing or corrupt, you may be in trouble

Use `SHUTDOWN ABORT` only when:

- Other shutdown modes hang or fail
- Or the instance is already misbehaving badly

Think of it as yanking the battery cable off a running car engine. Yes, it stops. No, that does not mean it was a good idea.

---

## 4. Instance Recovery: What Happens After an Abort or Crash

After a crash or `SHUTDOWN ABORT`, the next `STARTUP` must perform **instance recovery**:

1. **Roll forward (SMON)**
   - `SMON` reads redo logs
   - Replays all changes since the last DBWn write
   - This includes both **committed and uncommitted** changes
2. **Open the database**
   - Once the roll‑forward phase reaches a consistent SCN, the database can open
   - Users can now connect
3. **Roll back (PMON / rollback segments)**
   - Uncommitted changes from the roll‑forward are undone
   - `PMON` does this **on demand**:
     - If no one touches those rows, cleanup can be leisurely
     - If a session needs a row, the rollback for that row happens immediately

The crucial point:

- Instance recovery assumes your **redo logs are intact**
- If redo is damaged or missing, recovery can fail

This is why repeatedly using `SHUTDOWN ABORT` or `STARTUP FORCE` on a healthy database is playing with fire.

---

## 5. `STARTUP FORCE`: Double Trouble in One Command

`STARTUP FORCE` is basically:

```sql
SHUTDOWN ABORT;
STARTUP;
```

in a single word.

Useful when:

- The instance is stuck in a bad state and will not respond to normal shutdowns

Dangerous when:

- You are just impatient and use it as your **default** restart method

If the database is healthy and responsive, the safer sequence is:

```sql
SHUTDOWN IMMEDIATE;
STARTUP;
```

Startup force is like turning your car off by disconnecting the battery because you cannot be bothered to turn the key. It *works*, but you should not be surprised when your radio presets and clock get reset.

---

## 6. Opening and Closing Pluggable Databases

Everything you just learned about startup and shutdown applies to the **CDB** as a whole. Pluggable databases (PDBs) add another layer: they live *inside* the CDB and have their own open/close lifecycle.

Key rule:

- If the **CDB** is not open, **none** of the PDBs are accessible.

Once the CDB root is open, each PDB can be:

- Open read‑write
- Open read‑only (for migrations, checks, etc.)
- In a restricted state

### 6.1 PDB “MOUNTED” vs instance “MOUNTED”

When a PDB is closed while the instance is up, its state in `V$PDBS` will be shown as **MOUNTED**.

That does **not** mean:

- The whole instance is only mounted
- The CDB is not open

It only means:

- This particular PDB is down
- Other PDBs in the same CDB may be open and happily serving sessions

So “PDB is MOUNTED” simply means “PDB is closed, but the CDB is up.”

### 6.2 Use `ALTER PLUGGABLE DATABASE`, not `SHUTDOWN`

You manage PDBs with:

```sql
ALTER PLUGGABLE DATABASE PDB1 OPEN;
ALTER PLUGGABLE DATABASE PDB1 CLOSE;
ALTER PLUGGABLE DATABASE ALL  OPEN;
```

SQL*Plus *will* accept `STARTUP`/`SHUTDOWN` when you connect directly to a PDB service, but in a multitenant world that is a trap:

- If your session is actually connected to the **CDB root** and you casually type `SHUTDOWN`, you just took out **every** PDB in that CDB.

With `ALTER PLUGGABLE DATABASE`:

- At the CDB root, Oracle will always scope the command to PDBs, not the instance
- You cannot accidentally kill the whole CDB when you meant to bounce just one tenant

### 6.3 Saving and discarding PDB open state

By default, after a CDB restart, PDBs **do not** automatically reopen in the same mode they had before. You can change that with **saved state**:

```sql
-- While PDB1 is open in the mode you like
ALTER PLUGGABLE DATABASE PDB1 SAVE STATE;
```

Now, the next time you start the CDB and open the root, Oracle will automatically bring `PDB1` back to that saved state.

If you later decide you *do not* want that behaviour:

```sql
ALTER PLUGGABLE DATABASE PDB1 DISCARD STATE;
```

The trigger that auto‑opens `PDB1` is effectively removed, and you go back to manual control.

### 6.4 Who is allowed to start and stop things?

Only privileged “sys‑style” roles can start up or shut down databases:

- `SYSDBA`
- `SYSOPER`
- `SYSBACKUP`
- `SYSDG`

That is why, in the demo, the connection looked like:

```bash
sqlplus / as sysdba
```

Normal users do **not** get to bounce instances or reopen PDBs just because they feel like it.

---

## 7. Putting It All Together

By now you should be able to:

- Explain the startup phases:
  - `NOMOUNT` – instance only
  - `MOUNT`   – control file, recovery checks
  - `OPEN`    – database available
- Choose the right open mode:
  - Normal read‑write
  - Read‑only
  - Restricted
- Pick the appropriate shutdown mode:
  - `NORMAL` for “everyone logs off on their own time” (almost never used)
  - `TRANSACTIONAL` when you must let transactions finish
  - `IMMEDIATE` as the standard, safe shutdown
  - `ABORT` only when the gentle methods fail
- Understand what **instance recovery** does and why SMON and PMON get busy after a crash
- Resist the urge to use `STARTUP FORCE` as your daily restart button
- Safely open and close individual PDBs with `ALTER PLUGGABLE DATABASE`, and use saved state so they come back up the way you expect

If you treat your database more like a carefully tuned machine and less like a light switch, you will avoid a whole class of “it was fine until we bounced it” horror stories.

---

## Lab 5 Link

When you are ready to practice this in the lab environment, use:

- [Lab 5 – Startup, Shutdown, and PDB Open Modes](labs/lab05-manage-startup-shutdown-and-pdbs.md)

