# 30 - Managing Audit with Unified Auditing (Because "Who Did What" Eventually Matters)

Chapter 4 introduces audit management in Oracle Database: what to audit, what
not to flood, and how unified auditing changes the game from "best effort" to
"this is now your problem if you ignore it."

---

## 1. Why Auditing Exists

Auditing is not prevention. It is accountability.

- Security controls try to stop bad behavior.
- Auditing records who did what, when, and how.

Yes, it is after the fact. No, that does not make it optional. It is still
critical for:

- private-data access traceability
- suspicious behavior investigation
- compliance evidence
- deterrence (people type more carefully when their name is attached)

---

## 2. Audit Inside the Security Stack

Audit is one layer among many database-side controls:

- VPD / row and column filtering
- data redaction
- data masking
- transparent data encryption (files, redo, undo, cache paths)

Core idea: protect data at the source, then audit the events that actually
matter.

---

## 3. Audit Types Covered

### Mandatory auditing

High-risk operations Oracle expects to be tracked:

- DBA/superuser activity
- DDL activity (especially structural changes)
- audit/security policy management operations

### Legacy/standard auditing

Older model using legacy parameters/targets. Still relevant in old estates, but
not the preferred modern model.

### Fine-grained/value-based auditing

For sensitive value changes and context-heavy capture where generic statement
logs are nowhere near enough.

---

## 4. Unified Auditing in 12c+

Unified auditing is the modern path.

- Early 12c deployments did not always have it enabled by default.
- You must verify status before designing policies.
- Unified records are managed in protected internal structures.

Quick check:

```sql
SELECT parameter, value
FROM v$option
WHERE parameter = 'Unified Auditing';
```

---

## 5. Role Separation (`AUDIT_ADMIN` vs `AUDIT_VIEWER`)

Two key privilege domains:

- `AUDIT_ADMIN`: create/enable/disable/manage policies and lifecycle
- `AUDIT_VIEWER`: read audit records

This gives you basic separation of duties, which is what adults call "not
letting everyone do everything because it is convenient."

---

## 6. Policy Lifecycle

1. verify unified audit is enabled
2. configure audit destination/storage
3. create policies
4. add meaningful conditions
5. enable policies
6. review and manage trail volume
7. purge/retain on schedule

---

## 7. Biggest Operational Rule: Avoid Audit Spam

If you log everything, you review nothing.

Good policy design:

- targets security-relevant events
- filters low-value noise
- focuses on high-risk success/failure paths

Otherwise you get a perfect archive of meaningless trivia and miss the one event
that mattered.

---

## 8. Context-Aware Conditions

Useful audit conditions include:

- user/role context
- operation type and outcome
- object sensitivity
- boundary violations vs normal expected activity

Failure-only auditing is often more useful for attack detection than logging
routine successful behavior all day long.

---

## 9. Key Takeaways

- auditing is accountability, not prevention.
- unified auditing is the preferred modern framework.
- mandatory and custom policies must be planned together.
- condition design determines whether audit is signal or noise.
- role separation around audit administration is not optional theater.