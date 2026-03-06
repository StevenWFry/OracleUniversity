## Lesson 19 - ASM Directories, Aliases, Templates, ACLs (in which storage paths get civilized and permissions stop being vibes)

This chapter is the practical file-management layer in ASM: directory layout, alias indirection, template-driven file behavior, and ACL-based access control.

By the end of this lesson, you should be able to:

- Create, rename, and drop ASM directories
- Use aliases as symbolic-link-style indirection for ASM files
- Create and manage templates for striping/redundancy defaults
- Apply template-based file creation patterns
- Configure ACL-backed ownership and permissions using OS users/groups

---

## 1. ASM Directories

ASM directories form a hierarchical namespace under each disk group:

- Disk group root (`+DATA`, `+FRA`, ...)
- Directories
- Subdirectories
- Files at leaf levels

SQL management patterns:

```sql
ALTER DISKGROUP DATA ADD DIRECTORY '+DATA/mydir';
ALTER DISKGROUP DATA RENAME DIRECTORY '+DATA/mydir' TO '+DATA/myotherdir';
ALTER DISKGROUP DATA DROP DIRECTORY '+DATA/myotherdir';
```

Same mental model as OS directories, just with ASM metadata behind it.

---

## 2. ASM Aliases

An ASM alias is symbolic-link-like indirection:

- Cleaner path/name for humans
- Can hide complex fully qualified internal name
- Useful when real file identity is long or intentionally not exposed

SQL patterns:

```sql
ALTER DISKGROUP DATA
  ADD ALIAS '+DATA/mydir/system.dbf'
  FOR '+DATA/ORCL/DATAFILE/system.262.123456789';

ALTER DISKGROUP DATA
  RENAME ALIAS '+DATA/mydir/system.dbf'
  TO '+DATA/mydir/payroll_compensation.dbf';

ALTER DISKGROUP DATA DROP ALIAS '+DATA/mydir/payroll_compensation.dbf';
```

Aliases are for access convenience, not new physical file copies.

---

## 3. Templates: Policy Reuse for New Files

Templates control default file-creation behavior in a disk group.

Common settings:

- Striping:
  - `COARSE` (AU-sized)
  - `FINE` (small stripe unit behavior)
- Redundancy:
  - unprotected
  - mirror
  - high

Disk groups get default templates at creation, based on disk-group redundancy mode.

---

## 4. Viewing and Managing Templates

Visibility:

- `V$ASM_TEMPLATE`
- ASMCMD template listing commands

SQL management patterns:

```sql
ALTER DISKGROUP DATA
  ADD TEMPLATE reliable ATTRIBUTES (MIRROR HIGH FINE);

ALTER DISKGROUP DATA
  MODIFY TEMPLATE reliable ATTRIBUTES (MIRROR HIGH COARSE);

ALTER DISKGROUP DATA
  DROP TEMPLATE reliable;
```

ASMCMD provides equivalent create/change/list/remove flows.

---

## 5. Using Templates During File Creation

Templates can be referenced with:

- Explicit alias-style file names
- Incomplete names
- OMF-style creation paths

They apply to new file creation behavior, so you do not have to re-specify striping/redundancy intent every single time.

In short: fewer repetitive clauses, fewer fat-fingered policy mismatches.

---

## 6. ACL Model in ASM

ASM ACLs map access to OS users/groups (not ASM-local users stored in data dictionary tables).

Why:

- ASM does not maintain traditional user-data dictionary storage the way a normal database does
- Ownership and auth model is OS-integrated

Applies across:

- Unix/Linux role/group models
- Windows environments (with platform-specific operational context)

---

## 7. ACL Prerequisites and Attributes

From this lesson:

- Compatibility baseline for ACL feature set: `11.2+` (ASM/RDBMS context)
- Access control should be enabled
- `umask`-style behavior influences effective permissions

Referenced attribute model includes:

- access control enablement
- access-control mask behavior (`umask`) for permission filtering

Permission shorthand called out in this chapter:

- `0` none
- `4` read
- `6` read/write

Use the environment's exact policy mapping and test before production rollout.

---

## 8. Managing ACL Users/Groups and Permissions

SQL-side operations include:

- Add/drop user
- Add/drop group
- Add/remove members
- Set ownership
- Set permissions
- Query user/group views (`V$ASM_USER`, `V$ASM_USERGROUP`)

ASMCMD-side operations include command families for:

- group/user create/modify/remove/list
- ownership and permission change
- password/set operations where supported by your security model

This is where ASM starts feeling very Unix-like on purpose.

---

## 9. Operational Guidance

When managing directories/aliases/templates/ACLs:

1. Define namespace structure first
2. Apply template defaults second
3. Enforce ACL ownership and permissions third
4. Validate in views/ASMCMD before handing off to application teams

Because path hygiene plus policy consistency prevents most avoidable file-management disasters.

---

## 10. Key Takeaways

- Directories and aliases make ASM namespace manageable for humans.
- Templates standardize striping/redundancy defaults for new files.
- ACLs in ASM rely on OS users/groups and platform auth models.
- SQL and ASMCMD both support full lifecycle operations for these objects.
- Good namespace + template + ACL discipline saves future-you from forensic archaeology.
