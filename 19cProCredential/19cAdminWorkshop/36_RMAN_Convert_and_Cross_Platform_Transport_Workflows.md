# 36 - RMAN Convert and Cross-Platform Transport Workflows

Welcome back to the part where data movement gets serious: endianness,
conversion timing, transport scripts, and the very specific command order that
keeps your migration from becoming a forensic project.

---

## 1. Why `RMAN CONVERT` Exists

If source and target platforms write bytes differently (big-endian vs
little-endian), you cannot just copy datafiles and call it a day.

You must convert at some point:

- during source-side extract/backup
- during target-side restore/import

Oracle gives you both options. Your downtime window decides which one hurts
less.

---

## 2. Determine Platform Endianness First

Use platform metadata views before touching transport commands.

Key view discussed:

- `V$TRANSPORTABLE_PLATFORM`

Important detail:

- platform name in conversion commands must match exactly (case and spelling)

No exact match, no joy.

---

## 3. Tablespace Conversion Pattern (`CONVERT TABLESPACE`)

Typical source-side pattern:

1. set tablespace read-only
2. run RMAN convert with explicit target platform
3. output converted files to target staging format/path

Example concept:

```rman
CONVERT TABLESPACE ...
  TO PLATFORM 'Exact Platform Name'
  FORMAT '/path/%U';
```

`FORMAT` controls where converted files land and how names are generated.

---

## 4. Near-Minimum-Downtime Strategy (Incremental Apply)

For low-downtime migration pressure:

1. take level 0 baseline backup/copy
2. apply incrementals repeatedly to destination
3. reduce delta over time
4. perform final catch-up in controlled window
5. open systems at synchronized point

This is the "chip away at the gap" model instead of "all in one giant outage."

---

## 5. Image Copy Transport Logic

When using image copies:

- source tablespaces commonly switched to read-only for consistency
- Data Pump exports metadata
- datafiles are moved to target
- target imports metadata and opens tablespaces read/write

If endian differs, add conversion either source-side or target-side in the
workflow. Same flow, extra step, more coffee.

---

## 6. Backup Set Transport Logic

Backup-set transport gives flexibility and compression options.

Two key patterns:

### Convert at source

```rman
BACKUP TO PLATFORM 'Target Platform Name' ... DATABASE;
```

### Convert at target

```rman
BACKUP FOR TRANSPORT ... DATABASE;
```

Then restore on target with foreign-database semantics.

---

## 7. Database vs Tablespace Read-Only Rules

From the lecture:

- full database transport workflows require stricter read-only handling
- cross-platform tablespace workflows can allow read/write in some incremental
  approaches until final consistency stage

Rule of thumb: if moving the entire database image, consistency constraints are
more rigid than moving selected transportable tablespaces.

---

## 8. Foreign Restore Semantics on Target

On destination, restore commands explicitly indicate foreign source context.

Example pattern:

```rman
RESTORE FOREIGN DATABASE TO NEW
  FROM BACKUPSET '...';
```

For tablespaces:

```rman
RESTORE FOREIGN TABLESPACE ...;
```

You are not restoring a local backup lineage. You are importing a transported
one.

---

## 9. Convert/Transport Script Workflow

RMAN-generated scripts can split responsibilities:

- conversion script (data format transformation)
- transport/create script (database creation/attach flow)

Operationally common sequence:

1. run convert/transport generation on source
2. move datafiles + pfile + SQL scripts to target
3. run convert script on target (if target-side conversion chosen)
4. run create SQL script
5. finalize identity/config steps

---

## 10. Required Post-Transport Considerations

Do not skip these just because the restore finished:

- password file availability on target
- external file dependencies (`BFILE`, directory objects)
- DBID change workflow (`DBNEWID`) where required
- parameter review from generated pfile/spfile artifacts

A technically successful restore can still be operationally broken if these are
ignored.

---

## 11. Data Pump + RMAN Together

In transport workflows, RMAN and Data Pump are usually complementary:

- RMAN handles physical data movement/conversion
- Data Pump handles metadata movement/mapping

Use both where appropriate instead of forcing one tool to do everything badly.

---

## 12. Key Takeaways

- cross-platform transport starts with endianness verification.
- decide conversion location based on downtime and resource windows.
- use incremental apply to reduce cutover risk.
- foreign restore semantics and script order matter.
- post-transport cleanup (DBID, files, params) is not optional.