# 29 - Configuring Profiles and User Resource Limits

Chapter 3 focuses on profiles: how to control user resource usage, password
behavior, and authentication policy in CDB/PDB environments.

---

## 1. What a Profile Is

A profile is a policy object assigned to users that controls two big areas:

- resource limits (session/instance usage boundaries)
- password/account management rules

Profiles work only when resource limiting is enabled (`RESOURCE_LIMIT=TRUE`).
Oracle also creates a default profile automatically, which can be tuned for your
baseline policy.

---

## 2. Common vs Local Profiles in Multitenant

Profile scope follows multitenant object rules:

- common profile in CDB root: available across containers
- local profile in a PDB: usable only in that PDB

You can assign a common profile to local users in a PDB, or assign a local PDB
profile when that PDB needs stricter or different behavior.

---

## 3. What You Can Limit with Profiles

Typical resource controls include:

- connect time and idle time
- concurrent session limits
- per-session resource boundaries (including PGA-related controls)
- additional execution/consumption constraints

Profiles are coarse controls. For detailed workload governance, Oracle Resource
Manager is usually the better tool.

---

## 4. Password and Account Controls

Profiles can enforce:

- failed-login lock threshold
- lock duration/unlock behavior
- password lifetime (expiry)
- password history/reuse rules
- password verification function

Design these as a combined policy, not isolated checkboxes. Example from the
lecture: password history without time constraints can be bypassed immediately
by rapid password rotations.

---

## 5. Verify Functions and Complexity Levels

Oracle ships built-in password verification functions, including:

- `verify_function_11G`
- `verify_function_12C`
- stronger variants such as `verify_function_12C_strong`
- STIG-oriented options in hardened environments

Stronger functions can enforce stricter length, character mix, repetition
controls, and dictionary/word-based restrictions.

---

## 6. Assigning Profiles

Profiles are assigned through `ALTER USER`:

```sql
ALTER USER app_user PROFILE profile_name;
```

In CDB root for common users/profiles:

```sql
ALTER USER c##app_admin PROFILE c##profile_dev CONTAINER=ALL;
```

A common user can still have different effective profile behavior per container
when local profile assignments are used in specific PDBs.

---

## 7. Privileged User and Password File Note

Historically, profile behavior for superuser-style accounts had caveats because
authentication came from the password file path. In newer password file formats
(12.2+), profile compatibility improved for privileged-account management.

Still validate behavior in your exact release/auth path, especially for
complexity-function expectations.

---

## 8. EM Express Demo Highlights

From `EM Express -> Security -> Users`, you can:

- inspect users
- create users (or create-like)
- assign profiles and review profile options

SQL behavior from the demo still matters:

- in CDB root, names must satisfy common-user rules
- `CONTAINER=ALL` and common naming both need to be correct
- Oracle error text may not always reveal the exact root cause directly

---

## 9. Key Takeaways

- profiles are baseline guardrails for user risk control.
- use common profiles for broad policy, local profiles for PDB-specific needs.
- password policy strength comes from rule combinations, not one setting.
- use Resource Manager for fine-grained runtime control beyond profile basics.