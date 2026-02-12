# 26 - Creating Common Users in CDB and PDBs (And Not Accidentally Granting the Keys to Everything)

This chapter continues user management with multitenant scope rules, schema-only
accounts, authentication options, and why role-based privilege design is still
the difference between security architecture and chaos.

---

## 1. Scope Refresher: Common vs Local

In a CDB:

- users created in `CDB$ROOT` with `CONTAINER=ALL` are common users
- users created inside a PDB are local users for that PDB only

In an application container:

- users created in application root are common within that application container
- users created in application PDBs are local to those app PDBs

Common user prefix rules apply at CDB level (default `C##`, configurable).

---

## 2. Creating Common Users in CDB Root

Pattern:

```sql
CREATE USER c##app_admin IDENTIFIED BY "<password>" CONTAINER=ALL;
```

Effect:

- metadata/user entry exists across containers per common-user semantics
- privileges granted with container scope must be intentionally controlled

Because "it exists everywhere" is useful for operations and terrifying for mistakes.

---

## 3. Creating Local Users in PDBs

Inside target PDB:

```sql
CREATE USER app_user IDENTIFIED BY "<password>";
```

Local users:

- no common prefix required
- access only what is granted inside that PDB
- cannot magically cross to sibling PDBs

---

## 4. Application Root DDL Lifecycle Rules

For application root metadata changes (including common app-root users), lecture
emphasizes controlled lifecycle:

- perform changes in install/patch/upgrade mode flow
- finalize with corresponding end/finalization step and version advancement
- synchronize application PDBs so root changes are propagated

If you skip lifecycle discipline, "why didn’t this show up in app PDB2?" becomes your afternoon.

---

## 5. Schema-Only Accounts (No Authentication)

Schema-only user pattern:

- account owns objects
- account cannot log in directly

Use case:

- protect schema owner from direct session abuse
- reduce risk of privileged object renames/DDL bypassing layered security controls

When patching is needed:

- temporarily enable authentication
- perform maintenance
- remove authentication again

---

## 6. Authentication Models Mentioned

Primary options covered:

- password authentication (default for normal users)
- OS authentication (common for certain admin roles)
- password file authentication for privileged startup/down-state admin access
- advanced auth paths (LDAP/Kerberos/SSL/proxy/strong auth integrations)

Password behavior notes:

- case-sensitive in modern releases
- complexity/history/expiration policies should be profile-driven

---

## 7. Built-In Admin Identities and Separation of Duties

Specialized identities reduce blast radius:

- `SYSDBA`, `SYSOPER`
- `SYSBACKUP`
- `SYSDG`
- `SYSKM`
- `SYSRAC`
- supporting accounts like `SYSMAN`, `DBSNMP`

Operational message:

- stop doing every task as one universal super-account.

---

## 8. Roles vs Direct Privileges

Direct grants:

- harder to centrally govern at scale

Role-based grants:

- easier policy control and auditing
- cleaner change management for teams/jobs

Recommended flow:

1. define job functions
2. create roles per function
3. grant privileges to roles
4. grant roles to users

Least privilege with reusable governance beats ad-hoc grant sprawl every time.

---

## 9. Profiles and Account Lifecycle

Profiles control:

- password aging/expiration
- complexity/history
- basic session/resource constraints

Account controls:

- lock/unlock
- expire password
- enforce rotate-before-use behavior

Resource Manager can add deeper workload controls beyond profile basics.

---

## 10. Quotas: Who Gets Space

Quota logic from lecture:

- object owners need permanent tablespace quota
- read-only/report users generally do not need object-space quotas

Key governance areas to monitor:

- permanent data space
- undo generation pressure
- temp usage pressure

If someone asks for "UNLIMITED on everything," that is a conversation, not a checkbox.

---

## 11. Practical Takeaways

- treat user scope (`CDB` common vs `PDB` local) as a security boundary.
- use schema-only accounts where direct login is unnecessary.
- prefer role-based privilege architecture over direct one-off grants.
- align authentication type to risk and operational purpose.
- enforce profiles/quotas deliberately so growth and access stay controllable.
