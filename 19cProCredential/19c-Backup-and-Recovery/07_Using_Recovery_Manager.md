## Lesson 7 - Using Recovery Manager (in which RMAN is your most competent coworker and also your strict librarian)

And look, RMAN is one of those Oracle tools that sounds boring until the day you need it, at which point it instantly becomes the most important thing in your life.

By the end of this lesson, you should be able to:

- Explain what RMAN manages for backup and recovery workflows
- Connect to RMAN and run core backup/report commands
- Distinguish standalone RMAN commands from `RUN` blocks
- Configure and inspect persistent RMAN settings
- Set retention policy correctly using recovery window or redundancy

---

## 1. What RMAN Actually Does (more than "backup database;")

RMAN knows your database structure and backup metadata, including:

- data files and tablespaces
- control file/spfile backup and recovery needs
- backup/recovery state and history

RMAN can also integrate with:

- disk backups
- tape systems (for example, Oracle Secure Backup via SBT)
- cloud-oriented backup workflows

So yes, it is backup tooling, but it is also orchestration, policy, and reporting.

---

## 2. Connect and Run Basic Work

Before starting RMAN, source your environment:

```bash
. oraenv
```

Then connect:

```text
rman target "/ as sysbackup"
```

Common first commands:

```rman
BACKUP DATABASE;
LIST BACKUP;
DELETE OBSOLETE;
```

Why `DELETE OBSOLETE` matters:

- it removes backups outside retention policy
- it prevents FRA/disk from turning into a museum of old backups

And yes, RMAN command statements end with semicolons.

---

## 3. SQL from Inside RMAN (because context switching is overrated)

You can run SQL in RMAN, especially for backup/recovery-relevant inspection.

Example:

```rman
SQL "SELECT name, dbid, log_mode FROM v$database";
```

Older versions often required more explicit SQL prefix usage. Modern RMAN handling is friendlier, but the concept is the same: RMAN can execute SQL for operational checks.

---

## 4. Standalone Commands vs RUN Blocks

### Standalone RMAN command

Single command, executed directly:

```rman
BACKUP DATABASE;
```

### `RUN` block

Grouped commands inside braces, usually when you want explicit channel control and multi-step execution as one unit.

Example pattern:

```rman
RUN {
  ALLOCATE CHANNEL c1 DEVICE TYPE DISK;
  SQL "ALTER SYSTEM ARCHIVE LOG CURRENT";
  BACKUP DATABASE PLUS ARCHIVELOG;
  RELEASE CHANNEL c1;
}
```

Why this matters:

- standalone is simpler
- `RUN` blocks are better for controlled job flows

It gets worse if you forget channel logic in complex jobs and then wonder why throughput is weird.

---

## 5. Persistent RMAN Configuration (defaults with opinions)

RMAN lets you define persistent settings so every job does not need full manual redefinition.

Inspect current settings:

```rman
SHOW ALL;
```

Also available via database view:

```sql
SELECT * FROM v$rman_configuration;
```

Example persistent settings:

```rman
CONFIGURE DEVICE TYPE SBT PARALLELISM 3;
SHOW CONTROLFILE AUTOBACKUP FORMAT;
SHOW EXCLUDE;
```

Clear settings when needed:

```rman
CONFIGURE BACKUP OPTIMIZATION CLEAR;
CONFIGURE MAXSETSIZE CLEAR;
CONFIGURE DEFAULT DEVICE TYPE CLEAR;
```

So yes, RMAN supports both:

- long-term defaults (`CONFIGURE`)
- one-off job settings inside a specific run flow

---

## 6. Retention Policy (where storage budget meets recovery reality)

You can define retention in two mutually exclusive models:

1. **Recovery window**  
Keep backups needed to recover to any point in the last `N` days.

```rman
CONFIGURE RETENTION POLICY TO RECOVERY WINDOW OF 7 DAYS;
```

2. **Redundancy**  
Keep a fixed number of full backup copies.

```rman
CONFIGURE RETENTION POLICY TO REDUNDANCY 2;
```

You choose one model at a time, not both.

Why this matters:

- aggressive cleanup saves space
- aggressive cleanup can also erase your recovery options if policy is wrong

That is not a technical issue. That is a planning issue wearing a fake mustache.

---

## 7. Obsolete Backups: The Practical Meaning

With a 7-day recovery window, a very old backup can become obsolete if newer backup + archived log chain fully covers recovery needs inside that window.

In plain English:

- "old" is not determined by age alone
- "obsolete" is determined by whether it is still required to satisfy policy

So if backup A is outside recoverability requirements, it is a cleanup candidate. If it is still needed for policy compliance, it stays.

---

## 8. Practical Guidance Before You Touch Production

1. Run `SHOW ALL;` and save output before changing RMAN config.
2. Decide retention model based on RPO/RTO, not vibes.
3. Test `DELETE OBSOLETE` in a non-production environment first.
4. Use `RUN` blocks for multi-step jobs with explicit channels.
5. Periodically validate you can actually restore, not just backup.

Because "we have backups" and "we can recover" are famously not the same sentence.

---

## 9. Wrap-Up

You now have the RMAN foundation for:

- connecting and running core commands
- using SQL in RMAN for operational checks
- choosing between standalone commands and run blocks
- managing persistent config state
- applying retention policy without sabotaging recovery

RMAN is stable, mature, and very good at its job. Which means if recovery fails, it is usually not RMAN. It is configuration, policy, or process. And yes, that includes us.
