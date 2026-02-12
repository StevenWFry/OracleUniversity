# 27 - Assigning Quotas and Basic User Privileges (Because Unlimited Is Not a Security Model)

This chapter closes the user-management basics by focusing on quotas, privilege
assignment flow, and why one badly controlled user can punish an entire
instance.

---

## 1. Why Quotas Matter

If users can create objects and no quota/resource limits are enforced, they can
consume far more than intended:

- permanent tablespace growth
- temp usage blowouts
- undo pressure
- overall workload disruption

Quota and profile settings are not paperwork. They are blast-radius controls.

---

## 2. "Unlimited by Accident" Is a Real Risk

Lecture warning:

- when constraints are not explicitly designed, users can end up effectively
 unconstrained for operations they are allowed to perform

With proper profile/resource configuration:

- Oracle can reject over-budget operations early instead of letting them run
 wild and then cleaning up afterward.

---

## 3. Basic Local User Creation Flow (Demo Pattern)

In a PDB context (`PDB1` in demo), creating a user creates a **local** user:

```sql
CREATE USER ron IDENTIFIED BY ron;
```

But user cannot log in yet without session privilege:

```sql
GRANT CREATE SESSION TO ron;
```

Then connect:

```sql
CONNECT ron/ron@pdb1
```

---

## 4. Role-Based Privilege Assignment Example

Demo sequence:

```sql
CREATE ROLE r1;
GRANT SELECT ON sys.emp TO r1;
GRANT r1 TO ron;
```

Result:

- `RON` can `SELECT` from `SYS.EMP`
- `RON` still cannot `UPDATE` without explicit update privilege

This is the point: grant exactly what is needed, not what is convenient.

---

## 5. Error Messages and Security Behavior

Lecture note:

- some Oracle error responses intentionally avoid revealing sensitive object
 existence/details when user lacks privilege

Example behavior:

- user may see generic "object does not exist" style output for inaccessible
 dictionary objects
- for objects where user already has partial visibility, privilege errors can be
 more explicit

Treat error messages as controlled disclosure, not always literal truth.

---

## 6. Profiles and Quotas in Practice

Profiles can enforce:

- password controls
- session/resource limits

Quotas should be planned for:

- permanent tablespace usage
- temp usage
- undo impact patterns

Workflow:

1. create profile/role model
2. create user
3. grant minimal privileges/roles
4. assign profile and quotas
5. monitor and adjust based on real workload behavior

---

## 7. Key Takeaways

- user creation without limits is operational debt.
- `CREATE SESSION` is intentionally separate from user creation.
- role-based grants are easier to audit and safer to evolve.
- quota/profile design prevents runaway usage from becoming shared outage.
- privilege and error behavior are designed to minimize information leakage.
