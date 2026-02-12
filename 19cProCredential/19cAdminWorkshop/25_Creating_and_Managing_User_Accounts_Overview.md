# 25 - Creating and Managing User Accounts Overview (Who Can Log In, Who Can Break Things)

This starts the user-account module: what database users are, why modern Oracle
separates user types more aggressively, and how user scope works in CDB/PDB and
application container models.

---

## 1. Why User Design Matters Now

Every database session must connect as a user. No user identity means no
session, no privilege context, no execution policy.

Modern security pressure changed old habits:

- app connecting as schema owner is risky (SQL injection blast radius)
- admins connecting as full-power accounts for routine tasks is risky
- principle of least privilege is now operationally mandatory, not optional

---

## 2. User Categories You Actually Need

Lecture categorization:

- superusers (Oracle-provided operational identities/roles)
- DBA operational users (named per-admin accounts, role-scoped)
- schema owners (object ownership accounts)
- regular application/report users (least-privilege usage accounts)

Best-practice direction:

- do not use one mega-account for everything
- create accounts by duty and traceability

---

## 3. Schema-Only Users

In newer releases, you can create schema-only users (no authentication).

Effect:

- user can own objects
- user cannot log in directly

This is useful for reducing attack surface on schema-owner identities.

---

## 4. User Definition Essentials

When creating users, key attributes include:

- unique username
- authentication method (or none for schema-only)
- default permanent tablespace
- default temporary tablespace
- profile assignment (password/session limits)

Profiles manage:

- password complexity/history/expiration
- session/resource limits (basic controls)

For deeper runtime resource control, Resource Manager and consumer groups are
generally preferred.

---

## 5. User vs Schema (Still Confused Everywhere)

User account:

- security identity/auth context

Schema:

- collection of objects owned by a user

They often share the same name, which is why people keep mixing them up. Same
label, different concept.

---

## 6. Built-In and Privileged Accounts/Roles

Core references from lecture:

- `SYS` (full internal superuser context)
- `SYSTEM` (historical DBA-style admin account, but not ideal as universal admin login)
- `SYSOPER` (operational startup/shutdown style scope)

Additional 12c+ admin identities:

- `SYSBACKUP` (backup/recovery scope)
- `SYSDG` (Data Guard scope)
- `SYSKM` (key management/TDE scope)
- `SYSRAC` (cluster/RAC scope)
- `SYSMAN` (EM repository ownership)
- `DBSNMP` (monitoring/agent data collection scope)

Theme:

- specialized admin roles reduce unnecessary data exposure.

---

## 7. CDB Common vs PDB Local Users

In multitenant:

- users created in CDB root are common users
- users created in a PDB are local users

Common users:

- exist across containers per common-user model
- typically used for metadata/administrative scope

Local users:

- exist only inside one PDB
- privileges apply only there

---

## 8. Application Container User Scope

Application container has its own root/PDB model:

- users in application root are common within that application container
- users in application PDB are local to that application PDB

Important distinction:

- app-root common user scope is not the same as CDB-root common scope.

---

## 9. Common User Prefix

Common users in CDB follow common-user prefix rules (default `C##`, subject to
parameter configuration).

Local users in PDBs use normal naming conventions without common prefix
requirements.

---

## 10. Practical Takeaways

- map accounts to duties, not convenience.
- avoid schema-owner logins for routine app traffic.
- use specialized admin identities (`SYSBACKUP`, `SYSDG`, etc.) for scoped tasks.
- understand root/common/local boundaries before granting privileges.
- track everything with named user accounts, not shared superuser habits from 2008.
