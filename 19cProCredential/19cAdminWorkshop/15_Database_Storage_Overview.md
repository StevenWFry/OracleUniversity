# 15 - Database Storage Overview (Where Your Data Lives, And Why It Sometimes Sets Your Weekend On Fire)

This starts the storage module: physical file placement, logical storage
structures, and why tuning storage design is often the difference between
"database is fine" and "why is this query melting the building."

---

## 1. Physical vs Logical Storage

Oracle is physically stored in files but logically managed through database
structures.

You need both perspectives:

- Physical: file locations, redundancy, capacity, recoverability
- Logical: tablespaces, segments, extents, blocks, and access behavior

If physical design is wrong, recovery suffers. If logical design is wrong,
performance suffers. If both are wrong, congratulations: you built a panic simulator.

---

## 2. Core Database Files (Inside the Database)

Main files required for operation:

- control files
- online redo logs
- datafiles (CDB + PDB content)
- `PDB$SEED` files for empty PDB provisioning

Control files:

- contain structural metadata and operational tracking
- should be multiplexed across different storage locations

Redo logs:

- capture change vectors for commit durability
- support instance recovery and transactional consistency, aka "how Oracle avoids
 turning your commits into interpretive dance"

PDB seed/datafiles:

- seed is the template basis for empty PDB creation
- regular PDB files hold user/application data structures

---

## 3. Supporting Files (Outside Core Datafiles)

Important external/support storage areas:

- archive logs
- Fast Recovery Area (FRA) contents
 - backups
 - flashback logs
 - block change tracking
 - diagnostic traces / alert logs
- parameter file and password file
- wallet files (encryption keys/certs)
- Oracle Net config files (`tnsnames.ora`, `sqlnet.ora`, `listener.ora`)

Security note from lecture:

- do not keep wallet/password-file backups in the same place as ordinary
 backup payloads. Because if one location gets compromised, you have gifted the
 attacker both the vault and the key.

---

## 4. Location Rules and Parameters

Key placement conventions:

- network config default: `$ORACLE_HOME/network/admin` (or `TNS_ADMIN`)
- password/parameter file defaults depend on platform conventions under
 `ORACLE_HOME`
- wallet location historically via `sqlnet.ora`, with newer parameterized
 options in newer releases

RAC note:

- shared password-file handling improved in newer releases for easier cluster
 admin consistency. Fewer "which node has the right file?" arguments at 2 a.m.

---

## 5. Built-In Tablespaces Created With the Database

At database creation, Oracle provisions core logical containers such as:

- `SYSTEM`
- `SYSAUX`
- `UNDO`
- `TEMP`
- `USERS` (and optional sample-schema tablespaces)

These are foundational; application storage design extends from here, like a
house built on concrete instead of hope.

---

## 6. Designing Application Tablespaces

Practical design rule used in the lecture:

- create tablespaces based on object behavior and workload profile.

Examples:

- separate user data vs index data
- separate small vs large index workloads
- dedicated structures for large objects (LOB/video/text) where block strategy differs
- temp tablespace groups for heavy parallel/reporting temp usage

In other words: do not throw every object into one giant tablespace and call it
"architecture."

---

## 7. Block Size and File Strategy Choices

For each tablespace, decide:

1. block size
2. file destination
3. bigfile vs smallfile style

Bigfile tablespace:

- one very large datafile (can scale extremely high)

Smallfile tablespace:

- multiple datafiles
- Oracle manages object extent placement across available files, because you
 have better things to do than hand-place every block like decorative tiles

Choose based on storage platform characteristics, manageability, and workload
behavior, not vibes.

---

## 8. Logical Storage Chain: Tablespace -> Segment -> Extent -> Block

Storage hierarchy in use:

- tablespace holds segments (objects)
- segment grows by extents
- extents are made of Oracle blocks

Not all objects are single-segment:

- partitioned objects can map to multiple segments

Growth behavior:

- as segments grow, extents are allocated
- as extent demand grows, tablespace/file growth follows

Fragmentation concern:

- heavily fragmented extent layout can increase maintenance overhead and reduce
 IO efficiency, which is Oracle's way of charging you interest on past storage decisions.

---

## 9. Practical Takeaways

- plan physical file placement for resilience and recovery.
- design logical storage around workload patterns, not guesswork.
- separate storage classes by behavior (OLTP/index/LOB/temp/reporting).
- capacity and layout choices should be made before growth pressure arrives and
 everyone starts saying "just add disk" like it is a strategy.

Next lesson focus from transcript:

- deeper dive into segments, extents, and block-level behavior.
