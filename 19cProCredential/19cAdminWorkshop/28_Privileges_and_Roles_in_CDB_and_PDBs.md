# 28 - Privileges and Roles in CDB and PDBs (Who Can Do What, Where, and Why That Matters)

Chapter 2 of this module drills into privilege and role mechanics in a
multitenant database: system vs object privileges, common vs local scope, role
nesting, secure roles, and revoke behavior.

---

## 1. Two Privilege Types, One Common Mistake

Oracle privileges are broadly:

- **System privileges**: what operations a user can perform in a container
- **Object privileges**: what operations a user can perform on specific objects

Common mistake:

- granting directly to users everywhere

Better approach:

- grant privileges to roles, then roles to users

That gives you control levers. Direct grants give you future regret.

---

## 2. Roles: Containers for Privileges

Role lifecycle:

1. create role
2. grant system/object privileges to role
3. grant role to user (or role to role)

Role nesting is supported:

- grant role A to role B
- grantee of role B inherits privileges from both

This enables layered access models (`clerk` -> `manager`, etc.) without copy-paste privilege sprawl.

---

## 3. Password-Protected and Secure Roles

Roles can be password-protected:

- user must authenticate to enable role
- useful for temporary elevation workflows

Secure application roles:

- enabled only through authorized package logic
- commonly tied to controlled app/session initialization paths

This avoids constant grant/revoke churn and gives cleaner audit posture.

---

## 4. Common vs Local in Multitenant

Creation scope rules:

- CDB root users/roles intended across containers are common (`CONTAINER=ALL` flow)
- PDB-created users/roles are local to that PDB
- app-root users/roles are common within application container scope

Key point from lecture:

- common and local users/roles can be mixed in grants **within the same container context**
- local scope does not magically become global because the role name sounds important

---

## 5. `ANY` Privileges and Scope Risk

`ANY` system privileges are powerful and broad.

In a container architecture:

- scope still depends on where/ how granted
- CDB-level vs local-PDB behavior differs by privilege semantics

Translation:

- `ANY` is not "safe because multitenant"
- `ANY` still needs strict governance

---

## 6. Built-In Roles Mentioned

Examples from lecture:

- `DBA` (broad administrative role)
- `RESOURCE` (historically schema-owner style capability)
- `SCHEDULER_ADMIN`
- `SELECT_CATALOG_ROLE`

Use these intentionally and validate included privileges in your target release.

---

## 7. Revokes: System vs Object Cascade Behavior

Critical difference:

- **System privilege grants with admin option** do **not** cleanly cascade on revoke in the same way object grants do
- **Object privilege grants with grant option** do cascade from parent/child grant chain

So:

- revoke on object grants can automatically remove downstream grants
- revoke on system grants may leave downstream grantees still holding access

This is exactly why privilege lineage must be audited, not assumed.

---

## 8. Demo Pattern Recap (`RON` / `R1`)

Lecture demo reaffirmed:

- user created in `PDB1` is local
- cannot log in until `CREATE SESSION` is granted
- role `R1` with `SELECT` on `SYS.EMP` allows read
- `UPDATE` fails without explicit update privilege

Also highlighted:

- Oracle may return deliberately non-revealing errors for inaccessible objects
- some errors are security-hiding behavior, not literal object absence truth

---

## 9. Operational Caution: Powerful Built-In Accounts

Lecture warning on `SYSTEM` account behavior:

- avoid over-reliance on broad built-in admin accounts
- use named DBA accounts with role-based grants for auditability and least privilege

If everyone uses one high-power account, post-incident attribution turns into a detective novel.

---

## 10. Key Takeaways

- roles are the control plane; direct grants are the exception.
- design access in layers, then grant users the smallest required layer.
- understand common/local scope before granting in multitenant.
- treat `ANY` and admin/grant options as high-risk controls.
- test revoke behavior explicitly, especially for system privileges.
