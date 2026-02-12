# 35 - Transporting Data with Data Pump and RMAN

This chapter covers transport workflows: Data Pump export/import patterns,
transportable options, network links, and RMAN cross-platform support.

---

## 1. Data Pump Transport Flow

High-level pattern:

1. run `expdp` on source
2. produce dump file(s)
3. move/share dump location
4. run `impdp` on target
5. rebuild data/metadata per mode and remap settings

Data Pump jobs are server-managed and tracked with master-table metadata.

---

## 2. Interfaces and Scope Modes

You can execute via:

- command line
- parameter file (`parfile`)
- management GUI/orchestration

Scope options include:

- full database
- schema
- table
- tablespace
- transportable-style movement

---

## 3. Remap Capabilities

Import remap options include:

- datafile remap
- tablespace remap
- schema remap
- table remap
- directory remap

Useful when target design is intentionally different from source.

---

## 4. Non-CDB and PDB Movement Patterns

Supported patterns in lecture:

- non-CDB -> PDB
- PDB -> PDB across CDBs
- PDB -> non-CDB

Common prep sequence:

1. ensure target exists and is open
2. create Data Pump directory on target
3. copy/share dump files
4. create required target tablespaces (if needed)
5. run import with appropriate remap settings

---

## 5. Full Transportable Export/Import

Transportable workflows use export/import options such as:

- full mode
- `TRANSPORTABLE=ALWAYS`

Import then references transport artifacts and datafiles as required.

Operationally, user tablespaces are often set read-only during the transport
window for consistency.

---

## 6. Endian Conversion

Cross-platform movement may require endian conversion (byte order differences).

You choose where conversion occurs:

- source-side processing
- target-side import path

No conversion planning = no successful cross-platform day.

---

## 7. Network Link Transport

Database links can reduce manual staging overhead.

Pattern:

1. define DB link
2. mark relevant tablespaces read-only when required
3. transport over network path
4. import on target using link-aware options

---

## 8. RMAN-Assisted Transport

RMAN supports cross-platform transport workflows using:

- image copies
- backup sets

Benefits highlighted:

- reduced downtime potential
- compression support
- multisection support
- incremental-apply style migration workflows

Transport artifacts here are not ordinary "restore history" in the same way as
standard backup/restore tracking.

---

## 9. Key Takeaways

- Data Pump remains the core logical transport engine.
- transportable methods reduce manual rebuild effort.
- PDB/non-CDB movement is flexible when prechecks are done correctly.
- endian strategy is mandatory for cross-platform moves.
- RMAN is a strong option for large, time-sensitive platform migrations.