## Lesson 22 - ASM ACFS Administration (in which your file system gets snapshots, utilities, and enough knobs to ruin a weekend)

And look, this chapter covers day-to-day ACFS administration after creation: utility commands, resource integration, mapping views, snapshots, backup strategy, and performance tuning context.

By the end of this lesson, you should be able to:

- Use key `acfsutil` and ADVM commands for ACFS operations
- Configure database-side prerequisites for database files on ACFS
- Generate and query storage mapping views for ACFS-backed files
- Register ACFS resources with cluster tooling and manage dependencies
- Create, convert, list, and delete ACFS snapshots
- Apply backup and performance guidelines for ACFS environments

---

## 1. Core ACFS Utility Surface

Main admin tools in this lesson:

- `acfsutil` for ACFS operations
- ASMCMD ADVM commands for volume operations
- `srvctl`/`crsctl` for cluster resource integration

Typical `acfsutil` use cases include:

- File-system info/status
- Mount registration
- Snapshot create/delete/convert/list
- Size/space operations
- Tunable and parameter inspection
- ADVM-related helper operations

If ASM is the storage engine, `acfsutil` is your dashboard full of buttons you should press carefully.

---

## 2. Database File Support on ACFS (and Why It Needs Care)

If database files are placed on ACFS (supported, but with extra overhead), this lesson emphasizes:

- Keep compatibility at modern levels (use current release feature set)
- Use efficient ADVM volume settings (stripe columns/width aligned with current best practice)
- Set database parameter:

```sql
ALTER SYSTEM SET FILESYSTEMIO_OPTIONS=SETALL;
```

This parameter is required for proper database I/O behavior through file-system paths in this context.

---

## 3. File Mapping Views and Refresh Workflow

To inspect file-to-storage mapping:

- Enable mapping:

```sql
ALTER SYSTEM SET FILE_MAPPING=TRUE;
```

- Generate/refresh map data:

```sql
BEGIN
  DBMS_STORAGE_MAP.MAP_ALL(<extent_count>);
END;
/
```

- Query map views (for example `V$MAP`) for extent/file mapping details.

This is useful for visibility into layout and fragmentation behavior in ACFS-backed database file workflows.

---

## 4. Cluster Resource Management for ACFS

ACFS file systems can be registered as cluster resources so startup/shutdown/dependency behavior is controlled centrally.

Operational pattern:

- Use `srvctl` to add/manage ACFS resource objects
- Use `crsctl` for deeper resource-level adjustments when needed

Why this matters:

- Resource dependencies are explicit
- Cluster can coordinate mount availability across nodes/services
- Lifecycle is consistent during node restarts/failover sequences

Because "someone mounted it manually once" is not HA architecture.

---

## 5. Snapshot Administration

ACFS snapshot behavior in this lesson:

- Up to 1023 snapshots per file system (read-only/read-write combinations)
- Snapshots can be file-system-level or targeted file-level workflows
- Snapshot metadata and changed-block references are stored efficiently

Common commands:

```bash
acfsutil snap create snap_name /mount/point
acfsutil snap delete snap_name /mount/point
acfsutil snap info /mount/point
acfsutil snap convert -r|-w snap_name /mount/point
```

Snapshots are stored under hidden ACFS snapshot structures associated with the mount.

Use cases:

- Rapid rollback/recovery
- Consistent point-in-time backup set
- Recovering deleted/incorrectly changed files

---

## 6. Backup Guidance

ACFS backup strategy must use tools that understand file-system semantics.

Supported style:

- OS/file-system-aware backup tools
- Oracle Secure Backup where applicable
- Compatible third-party tools with proper OS/Oracle integration

Not recommended:

- Tools that treat ACFS paths like unsupported raw-device blobs without proper integration

Snapshots often provide the cleanest operational checkpoint mechanism before external backup reads.

---

## 7. Performance Considerations

From this lesson's guidance:

- Use modern stripe column defaults (commonly 8) and sane stripe width planning
- Maintain good load distribution across ADVM volume space
- Leverage caching behavior (metadata/log activity) with awareness of periodic persistence
- Validate scheduler/I/O policy alignment at OS layer where applicable

Also monitor ACFS/volume-related views for:

- Volume status
- Snapshot inventory
- Tag/security/encryption context where configured

Because ACFS performance is not one setting. It is a stack decision.

---

## 8. Useful View Families (as referenced in this chapter context)

Depending on enabled features and release, inspect relevant ACFS/ADVM-related views for:

- snapshots
- tags
- volumes
- file system metadata
- volume statistics
- encryption/security realms

Always validate view availability in your exact release, since naming/coverage can vary by version and feature enablement.

---

## 9. Key Takeaways

- `acfsutil` is the primary ACFS operations tool after initial creation.
- Database-on-ACFS requires proper parameter and I/O alignment (`FILESYSTEMIO_OPTIONS=SETALL` in this workflow).
- Mapping views need explicit enablement and refresh calls (`FILE_MAPPING`, `DBMS_STORAGE_MAP`).
- Register ACFS with cluster resources for predictable HA behavior.
- Snapshot lifecycle (`create/delete/info/convert`) is central to ACFS safety and recovery operations.
