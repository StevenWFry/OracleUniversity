## Lesson 29 - Using PDB Snapshots (in which Oracle gives each `PDB` its own little snapshot carousel, because apparently one timeline per container was not enough)

And look, this topic sounds like Flashback Database with slightly different branding. It is not. PDB snapshots are part of the multitenant snapshot-carousel feature, and the source's "flash back the PDB using a snapshot" is really a clone-and-replace workflow, not a direct `FLASHBACK PLUGGABLE DATABASE` command.

By the end of this lesson, you should be able to:

- Explain what a PDB snapshot carousel is
- Create manual or automatic PDB snapshots
- Configure the snapshot mode and `MAX_PDB_SNAPSHOTS`
- Distinguish PDB snapshots from snapshot copy `PDB`s
- Clone a new `PDB` from an existing PDB snapshot
- Understand the practical "replace the old PDB with a clone from snapshot" workflow

---

## 1. What a PDB Snapshot Really Is

A PDB snapshot is:

- a point-in-time snapshot of one pluggable database
- stored as part of that PDB's snapshot carousel

Oracle's term for the collection of all snapshots of a given `PDB` is:

- **PDB snapshot carousel**

Important correction to the source:

- a PDB snapshot is **not** the same thing as a snapshot copy `PDB`
- a PDB snapshot is also **not** the same thing as `FLASHBACK PLUGGABLE DATABASE`

These are three different things:

### PDB snapshot

- created with `ALTER PLUGGABLE DATABASE ... SNAPSHOT`
- used later as a source for cloning a new `PDB`

### Snapshot copy `PDB`

- created with `CREATE PLUGGABLE DATABASE ... SNAPSHOT COPY`
- a sparse clone based on storage-level snapshot behavior

### Flashback `PDB`

- rewinds a `PDB` using flashback logs
- part of Flashback Database behavior, not the snapshot carousel

If you mix those up, Oracle will fix your education with syntax errors.

---

## 2. The Big 19c Prerequisite: Local Undo

Oracle documents an important requirement for PDB snapshots:

- the `CDB` must be in **local undo mode**

You can check it with:

```sql
SELECT property_value
FROM database_properties
WHERE property_name = 'LOCAL_UNDO_ENABLED';
```

If that returns `TRUE`, then you are in the right mode.

This is the biggest hidden assumption in the source material.
Without local undo, the snapshot-carousel feature is not simply "a thing you can turn on whenever you feel nostalgic."

---

## 3. Snapshot Modes

Oracle lets each `PDB` control how snapshots are created over its lifetime.

The key modes are:

- `NONE`
- `MANUAL`
- `EVERY interval`

### `NONE`

- no snapshots can be created for this `PDB`

### `MANUAL`

- snapshots can only be created manually

### `EVERY`

- snapshots are created automatically at the specified interval
- manual snapshots are still allowed

You configure this with `ALTER PLUGGABLE DATABASE`.

Examples:

```sql
ALTER PLUGGABLE DATABASE SNAPSHOT MODE MANUAL;
```

```sql
ALTER PLUGGABLE DATABASE SNAPSHOT MODE EVERY 24 HOURS;
```

Important correction to the source:

- to "disable" the feature, Oracle uses `SNAPSHOT MODE NONE`
- not some generic "snapshot mode is none" idea pulled from the air

And yes, Oracle's syntax is fussy enough that the exact wording matters.

---

## 4. Manual Snapshots

To create a manual snapshot, connect to the `PDB` and run:

```sql
ALTER PLUGGABLE DATABASE SNAPSHOT sales_snap;
```

You can create multiple manual snapshots over time:

```sql
ALTER PLUGGABLE DATABASE SNAPSHOT pre_patch;
ALTER PLUGGABLE DATABASE SNAPSHOT post_patch;
```

Oracle stores metadata about snapshots in:

- `DBA_PDB_SNAPSHOTS`

Example query:

```sql
SELECT con_name,
       snapshot_name,
       snapshot_scn,
       snapshot_time
FROM dba_pdb_snapshots
ORDER BY snapshot_time;
```

This is how you answer the very reasonable DBA question:

- "Which versions of this `PDB` did we actually preserve?"

instead of relying on your memory, which is not a supported Oracle feature.

---

## 5. Automatic Snapshots

You can also let Oracle create snapshots automatically at intervals.

Example:

```sql
ALTER PLUGGABLE DATABASE SNAPSHOT MODE EVERY 24 HOURS;
```

Oracle documents limits on the interval:

- if specified in minutes, it must be less than `3000`
- if specified in hours, it must be less than `2000`

So no, you cannot ask Oracle for "every 5000 minutes" and call it a plan.

Automatic snapshots are useful when:

- you want recurring safety points
- you expect frequent rollback/clone needs for one `PDB`
- you want the carousel to maintain itself

Manual snapshots are better when:

- you want precise named checkpoints around known risky changes

The source is correct that either model is possible.

---

## 6. The Carousel Limit and `MAX_PDB_SNAPSHOTS`

Each `PDB` snapshot carousel has a maximum number of retained snapshots.

In `19c`:

- the default maximum is `8`
- the maximum possible value is also `8`

This is one of those rare Oracle settings where the default and the ceiling are the same, which is almost suspiciously concise.

When the maximum is exceeded:

- Oracle purges the oldest snapshot automatically

You can inspect or set the limit with:

```sql
ALTER PLUGGABLE DATABASE SET MAX_PDB_SNAPSHOTS 4;
```

And the current setting is visible in:

- `CDB_PROPERTIES`

Important correction to the source:

- this is not an initialization parameter in the traditional sense
- it is a database property controlled through the `ALTER PLUGGABLE DATABASE` or `CREATE PLUGGABLE DATABASE` syntax

So while the source gets the idea right, the implementation detail matters.

---

## 7. Sparse vs Full Snapshot Behavior

The source treats snapshots as generic copies, but Oracle's behavior depends on storage and on `CLONEDB`.

Oracle documents this in `19c`:

- if the file system supports sparse files, then all snapshots except the first can be sparse
- this can significantly reduce storage usage

And Oracle also notes that the behavior depends on:

- `CLONEDB=TRUE` or `FALSE`
- whether the source `PDB` is read/write or read-only
- whether the storage system supports sparse files or storage snapshots

So if you are planning with the phrase:

- "snapshots are cheap"

you should immediately add:

- "on the right storage, with the right settings"

because otherwise Oracle may decide each one is a nice full copy and your disk budget will learn humility.

---

## 8. Cloning a New PDB from a Snapshot

This is where the source starts calling the workflow "flashback the `PDB` using a snapshot."

That is not the best description.

What you actually do is create a **new** `PDB` from an existing snapshot:

```sql
CREATE PLUGGABLE DATABASE pdb1_restore
  FROM pdb1
  USING SNAPSHOT sales_snap;
```

Or by `SCN`:

```sql
CREATE PLUGGABLE DATABASE pdb1_restore
  FROM pdb1
  USING SNAPSHOT AT SCN 3817001;
```

Or by time:

```sql
CREATE PLUGGABLE DATABASE pdb1_restore
  FROM pdb1
  USING SNAPSHOT AT TIME
    TO_TIMESTAMP('2026-03-20 10:15:00','YYYY-MM-DD HH24:MI:SS');
```

That gives you a **new** `PDB` based on the chosen snapshot.

It does not rewind the existing one in place.

That is the most important correction in this whole topic.

---

## 9. Replacing the Original PDB with the Snapshot-Based Clone

The source's practical workflow is really this:

1. choose the desired snapshot
2. create a new `PDB` from that snapshot
3. drop the original `PDB`
4. rename the new `PDB` to the old name, if desired

This is not ordinary flashback.
It is replacement by clone.

So if you want to "go back to Wednesday's version," what you are actually doing is:

- build a new `PDB` from Wednesday's snapshot
- discard the old one
- optionally rename the clone so users think the original came back

And frankly, that is a perfectly valid pattern.
It just deserves to be described honestly.

---

## 10. Dropping Snapshots

If a snapshot is no longer needed, you can drop it explicitly:

```sql
ALTER PLUGGABLE DATABASE DROP SNAPSHOT sales_snap;
```

Oracle raises an error if a `PDB` is still using that snapshot.

And if you want to drop **all** snapshots for the `PDB`, Oracle documents the clean way:

```sql
ALTER PLUGGABLE DATABASE SET MAX_PDB_SNAPSHOTS 0;
```

That is the supported way to empty the carousel, not an emotional decision to pretend the snapshots were never there.

---

## 11. What This Feature Is Good For

PDB snapshots are useful when:

- you want multiple rewind checkpoints for one `PDB`
- you frequently create temporary recovery/test clones
- you want rollback options that outlive flashback-log or undo windows
- you want to preserve named `PDB` states around patching or data changes

They are especially attractive when:

- the `PDB` is the blast radius you care about
- you do not want to touch the rest of the `CDB`

This is why snapshot carousel can be so effective in multitenant environments.
It turns one `PDB` into a manageable little history project instead of dragging the whole container through your regret.

---

## 12. What This Feature Is Not

PDB snapshots are **not**:

- Flashback Query
- Flashback Table
- Flashback Database
- snapshot copy `PDB`s
- RMAN backups

They are a distinct multitenant cloning/snapshot feature.

The easiest way to stay sane is:

- Flashback rewinds in place
- snapshots preserve cloneable historical states
- backups are still backups

If you keep those three buckets separate, Oracle becomes much less likely to slap you with a syntax error and a moral lesson.

---

## 13. Practical Takeaways

- PDB snapshots belong to the snapshot carousel feature, not the normal Flashback Database engine.
- Local undo mode is a hard prerequisite.
- `SNAPSHOT MODE` can be `NONE`, `MANUAL`, or automatic with `EVERY interval`.
- The default and maximum `MAX_PDB_SNAPSHOTS` value in `19c` is `8`.
- You can clone from a snapshot by name, `SCN`, or timestamp using `USING SNAPSHOT`.
- "Flashing back" a `PDB` with a snapshot is really a clone-drop-rename workflow.
- Sparse behavior depends on storage and configuration, so do not assume every snapshot is cheap.

PDB snapshots are a very useful multitenant safety net.
They just have one recurring Oracle theme attached:

- the feature is excellent
- the naming is confusing
- and the storage consequences depend on details you are expected to know already.
