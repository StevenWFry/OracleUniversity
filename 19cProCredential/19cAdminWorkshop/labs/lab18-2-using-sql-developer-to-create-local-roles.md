# Lab 18-2 - Using SQL Developer to Create Local Roles

Practice 2-2 uses SQL Developer to create local roles in `ORCLPDB1` and grant
object privileges on HR tables.

---

## 1. Practice Goal

Create local roles:

- `HR_CLERK`
- `HR_MANAGER`

Grant privileges:

- `HR_CLERK`: `SELECT`, `UPDATE` on `HR.EMPLOYEES`
- `HR_MANAGER`: `SELECT`, `INSERT`, `UPDATE`, `DELETE` on HR tables used in lab

Assumptions:

- logged in as `oracle` OS user
- Practice 2-1 completed (`PDBADMIN` has local `DBA`)

---

## 2. Create SQL Developer Connection (`PDB1-PDBADMIN`)

1. Launch SQL Developer:
 - `Applications` -> `Programming` -> `SQL Developer`
2. In Connections pane, click `+`.
3. Set:
 - Name: `PDB1-PDBADMIN`
 - Username: `pdbadmin`
 - Password: `cloud_4U`
 - Connection type: `TNS`
 - Network alias: `orclpdb1`
4. Click `Test` (expect success), then `Save`, then `Connect`.

---

## 3. Open DBA Navigator for This Connection

1. `View` -> `DBA`
2. In DBA pane, click `+`.
3. Select connection `PDB1-PDBADMIN`.
4. Confirm it appears in DBA tree.

---

## 4. Create Role `HR_CLERK`

In DBA pane:

1. Expand `PDB1-PDBADMIN` -> `Security` -> `Roles`.
2. In Roles tab: `Actions` -> `Create New`.
3. Role name: `HR_CLERK`.
4. Review SQL tab.
5. Click `Apply`, then `OK`, then `Close`.
6. Verify role is listed.

---

## 5. Grant Object Privileges to `HR_CLERK`

From Connections pane (not DBA pane):

1. Expand `PDB1-PDBADMIN` -> `Other Users` -> `HR` -> `Tables`.
2. Select table `EMPLOYEES`.
3. In table details, open `Grants` tab.
4. Add grant:
 - User/Role: `HR_CLERK`
 - Privileges: move `SELECT` and `UPDATE` to granted list
5. Review SQL:

```sql
GRANT SELECT, UPDATE ON hr.employees TO hr_clerk;
```

6. Apply and confirm success.

---

## 6. Create Role `HR_MANAGER`

In DBA pane:

1. `Security` -> `Roles`
2. `Actions` -> `Create New`
3. Role name: `HR_MANAGER`
4. Review SQL, apply, confirm, close
5. Verify role appears in role list

---

## 7. Grant HR Table Privileges to `HR_MANAGER`

Target privileges:

- `SELECT`, `INSERT`, `UPDATE`, `DELETE`

Use either SQL Developer grant UI or worksheet SQL.

Transcript flow used worksheet SQL after copying template. Example pattern:

```sql
GRANT DELETE, INSERT, UPDATE, SELECT ON hr.countries   TO hr_manager;
GRANT DELETE, INSERT, UPDATE, SELECT ON hr.employees   TO hr_manager;
GRANT DELETE, INSERT, UPDATE, SELECT ON hr.regions     TO hr_manager;
GRANT DELETE, INSERT, UPDATE, SELECT ON hr.locations   TO hr_manager;
GRANT DELETE, INSERT, UPDATE, SELECT ON hr.job_history TO hr_manager;
GRANT DELETE, INSERT, UPDATE, SELECT ON hr.jobs        TO hr_manager;
GRANT DELETE, INSERT, UPDATE, SELECT ON hr.departments TO hr_manager;
```

Run each statement and verify grant success.

---

## 8. Verify `HR_MANAGER` Object Privileges

In DBA pane:

1. `Security` -> `Roles`
2. Double-click `HR_MANAGER`
3. Open `Object Privileges` tab
4. Confirm listed HR tables and expected privileges

---

## 9. Exit

Close SQL Developer. If prompted to save unrelated UI changes, choose as
appropriate (`No` in transcript flow).

---

## 10. Lab Result

You created local roles `HR_CLERK` and `HR_MANAGER` in `ORCLPDB1` and assigned
object-level HR schema privileges needed for upcoming user/role assignment labs.
