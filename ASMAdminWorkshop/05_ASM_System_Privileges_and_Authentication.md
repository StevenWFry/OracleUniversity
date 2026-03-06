## Lesson 5 - ASM System Privileges and Authentication (in which everyone wants `SYSASM` and almost nobody should have it)

This is the security and access-control chapter: who can connect to ASM, how they authenticate, and which tools they can use without accidentally setting fire to shared storage.

By the end of this lesson, you should be able to:

- Explain why ASM user/security behavior differs from normal database instances
- Distinguish `SYSASM`, `SYSDBA`, and `SYSOPER` scope in ASM
- Map ASM administrative privileges to OS groups
- Compare local OS authentication, local password authentication, and remote password authentication
- Manage and validate ASM password-file users and supporting tools

---

## 1. Why ASM Security Is Different

ASM is not a normal data-serving database instance:

- It has no user datafiles
- It does not maintain a traditional data dictionary for regular schema users

So authentication/authorization centers on:

- Operating system group membership
- Password-file-based administrative users

This is less "open a user account and grant table access" and more "prove you are an admin before touching the storage control plane."

---

## 2. ASM System Privileges

Core privilege levels in ASM:

- `SYSASM`
  - Full ASM administration
  - Intended ASM superuser privilege
  - Typically mapped to an OSASM group

- `SYSDBA`
  - Database-admin-style privilege with limited ASM scope
  - Useful for visibility and some operations, but not full ASM control
  - Typically mapped to an OSDBA-style group

- `SYSOPER`
  - Limited operational privilege set
  - Designed for basic/non-destructive operations
  - Typically mapped to an OSOPER-style group

Default behavior note:

- `SYS` in ASM is granted `SYSASM`
- Connecting `AS SYSDBA` to ASM is allowed, but capabilities are narrower than `SYSASM`

---

## 3. OS Group Design and Account Separation

You can technically install everything with one OS user in all groups. You can also technically cut your own hair with hedge clippers. Both are legal; neither is ideal.

Recommended pattern:

- Separate Grid Infrastructure owner (commonly `grid`)
- Separate database software owner (commonly `oracle`)
- Use group-based privilege boundaries for OSASM/OSDBA/OSOPER responsibilities

For higher security environments:

- Consider separate OS ownership models per database/application domain as needed

---

## 4. Authentication Paths

### 4.1 Local OS authentication

When client and ASM instance are on the same host:

- Authenticate through OS group membership
- Common usage:
  - `sqlplus / as sysasm`
  - `sqlplus / as sysdba`

### 4.2 Local password-file authentication

You can also authenticate locally with explicit username/password:

- Useful when multiple admins need individual credentials
- Users are granted ASM system privileges and validated against password-file entries

### 4.3 Remote authentication

Remote ASM connections require password-file authentication:

- Connect with `username/password@net_alias AS SYSASM` (or appropriate privilege)
- TNS alias/service points to the target ASM instance

No password file, no remote admin session. End of negotiation.

---

## 5. ASM Password File Essentials

Password-file behavior for ASM:

- Created during Grid Infrastructure/ASM setup
- Can be recreated or migrated with `orapwd` when needed
- Must support password-file usage (for example, `remote_login_passwordfile` not set to `NONE`)

Typical managed accounts include:

- `SYS`
- `ASMSNMP`
- Additional admin principals granted ASM system privileges

How users get effective access:

1. Create user principal as needed
2. Grant ASM administrative privilege (`SYSASM`, `SYSDBA`, or `SYSOPER`)
3. Granting privilege updates effective password-file authorization for admin operations

Validation tools:

- `V$PWFILE_USERS`
- `asmcmd lspwusr`
- `asmcmd orapwusr` (user management helper)

Cluster best practice:

- Keep password files in shared storage to avoid node drift and synchronization mess
- ASM compatibility requirements apply for some password-file-in-ASM scenarios (12.1+ baseline in the notes)

---

## 6. `ASMCMD` and Privilege Behavior

`ASMCMD` authentication is OS-based by default, then privilege-scoped.

Examples:

- `asmcmd --privilege sysasm`
- `asmcmd --privilege sysdba`

Privilege impact:

- `SYSASM`: full ASM administration
- `SYSDBA`: limited operational/visibility scope

If ASM instance is down:

- Only a subset of commands are usable (help, startup/shutdown and a few basic utilities)

So if `asmcmd` feels strangely unhelpful, check whether ASM is actually running before blaming the command-line interface.

---

## 7. Tooling You Will Use

Main tooling stack for ASM operations:

- Oracle Universal Installer (software install layer)
- ASMCA (disk groups, clients, volumes, ACFS-related actions)
- SQL*Plus (full SQL-based admin control)
- ASMCMD (file-system-like ASM operations)
- `srvctl` (Clusterware-managed ASM/disk group service control)
- Enterprise Manager / Cloud Control (GUI-driven monitoring and administration)

If SQL*Plus is the scalpel, ASMCMD is the utility knife, and `srvctl` is the person with master keys to the building.

---

## 8. Key Takeaways

- ASM security is admin-centric: OS groups plus password-file authorization.
- `SYSASM` is the true ASM superuser; `SYSDBA`/`SYSOPER` are narrower.
- Separate Grid and database OS ownership whenever possible.
- Remote ASM administration requires password-file authentication.
- `V$PWFILE_USERS`, `lspwusr`, and `orapwusr` are core visibility/management tools.
