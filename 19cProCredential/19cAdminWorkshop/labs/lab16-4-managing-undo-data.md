# Lab 16-4 - Managing Undo Data

Practice 4 is a short validation lab: inspect undo usage across containers and
confirm local undo mode is enabled for multitenant operations.

---

## 1. Practice Goal

You will:

- view undo tablespaces in the container database
- determine undo mode from observed container behavior
- verify `LOCAL_UNDO_ENABLED` directly from database properties

Lab estimate from guide:

- about 2 minutes

---

## 2. Connect to CDB Root

Open terminal and connect as `system` to CDB root:

```bash
sqlplus system/cloud_4U@orclcdb
```

---

## 3. Display Undo Tablespaces Across Containers

Run query to list undo tablespaces by container (guide query pattern):

```sql
SELECT con_id, tablespace_name
FROM   cdb_tablespaces
WHERE  contents = 'UNDO'
ORDER  BY con_id;
```

Interpretation from practice:

- if each container has its own undo tablespace, undo mode is local undo.

---

## 4. Why Local Undo Mode

Guide conclusion:

- local undo supports key multitenant operations such as:
 - hot clone
 - PDB relocation
 - proxy-related PDB workflows

---

## 5. Verify Local Undo Property

Query database properties:

```sql
SELECT property_name, property_value
FROM   database_properties
WHERE  property_name = 'LOCAL_UNDO_ENABLED';
```

Expected:

- `LOCAL_UNDO_ENABLED = TRUE`

---

## 6. Exit

```sql
EXIT
```

---

## 7. Lab Result

You confirmed that undo is configured in local mode and verified the setting via
`DATABASE_PROPERTIES`, matching expected multitenant best practice behavior.
