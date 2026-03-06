## Lesson 21 - ASM ADVM and ACFS Creation (in which storage stops being abstract and becomes a mounted thing people can actually use)

This chapter is the practical build path for ADVM volumes and ACFS file systems: create volume, create file system, mount it, register it, and keep it manageable without summoning avoidable chaos.

By the end of this lesson, you should be able to:

- Create ADVM volumes with ASMCMD and SQL
- Build and mount ACFS on top of ADVM volumes
- Understand where root privilege is mandatory
- Use ASMCA session-level root credential behavior correctly
- Manage volume lifecycle (resize/enable/disable/delete) and basic file-system maintenance

---

## 1. End-to-End Build Flow

Typical sequence:

1. Create ADVM volume in a disk group
2. Inspect volume metadata
3. Create OS mount point
4. Create file system on the ADVM device
5. Mount file system
6. Register/manage with ACFS tooling

At that point, clients can use it like a regular file system path.

---

## 2. ASMCMD Volume Creation and Inspection

Core patterns:

```bash
asmcmd volcreate -G DATA -s 100G vol_appdata
asmcmd volinfo -G DATA vol_appdata
```

Where:

- `-G` chooses disk group
- `-s` sets size
- trailing value is the volume name

ASMCMD volume lifecycle also supports:

- resize
- delete
- enable/disable
- set attributes
- stats/info inspection

Because once a volume exists, the lifecycle work has only just begun.

---

## 3. File System Creation and Mount (OS Layer)

After ADVM volume exists, create file system and mount via OS commands.

Linux-style pattern:

```bash
mkdir -p /acfs/appdata
mkfs -t acfs /dev/asm/vol_appdata-###   # device naming depends on environment
mount -t acfs /dev/asm/vol_appdata-### /acfs/appdata
```

And for maintenance:

- `umount` to dismount
- `fsck`-style checks/repair for file-system consistency as appropriate

These steps require root privilege in normal Linux administration.

---

## 4. Root Privilege Reality Check

Volume creation in ASM scope can be done with ASM admin tooling, but file-system create/mount operations are OS-privileged.

Meaning:

- Database/ASM privileges are not enough for OS mount operations
- Root (or equivalent delegated privilege model) is required

So yes, this is another chapter where DBA + sysadmin cooperation is not optional.

---

## 5. ASMCA Assisted Workflow

ASMCA can orchestrate much of this interactively:

1. Select ADVM/volume manager area
2. Create volume with desired settings
3. Open settings and provide root credentials for the session
4. Define mount point and file-system options
5. Preview generated commands
6. Execute directly (if privileged session configured) or run commands manually as root

Important nuance:

- Root privilege in ASMCA is session-scoped
- Close ASMCA and you must re-establish privilege for next run

No persistent "root forever" switch, and that is a good thing.

---

## 6. SQL-Based Volume Management

SQL*Plus (ASM-connected) can manage ADVM volume operations, including:

- create
- resize
- drop
- enable/disable
- modify attributes

Use SQL when you need repeatable scripted control and change tracking in DBA-style workflows.

---

## 7. ACFS Registration and Utility Layer

After mount, `acfsutil` can be used for registration and operational management tasks.

This chapter context emphasizes:

- register volume/file-system integration points
- ensure system sees the mount as managed ACFS resource

From there, higher-level ACFS features (snapshots, replication options, etc.) can be layered according to policy and version support.

---

## 8. Basic Operational Checklist

Before handing an ACFS mount to app teams:

1. Verify volume exists and is enabled
2. Verify mount point exists and is mounted
3. Validate ownership/permission model
4. Confirm persistence/startup strategy if required by platform policy
5. Capture `volinfo`/mount/fs output for baseline

Because "it mounted once on Tuesday" is not an operations standard.

---

## 9. Key Takeaways

- ADVM volumes are the bridge between ASM disk groups and file-system consumers.
- ACFS creation/mount includes both ASM-side and OS-side operations.
- Root privilege is required for core file-system operations in Linux-style environments.
- ASMCA can simplify workflows, but root authorization is session-scoped.
- ASMCMD, SQL, and OS utilities together form the full lifecycle toolkit for ADVM/ACFS creation.
