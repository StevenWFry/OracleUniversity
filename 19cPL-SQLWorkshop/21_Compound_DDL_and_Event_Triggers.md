## Lesson 19 - Compound, DDL, and Event Triggers

OK, my friends, welcome to Lesson 19. This is the part where triggers go from "helpful assistant" to "Swiss Army knife with a tiny law degree."

After completing this lesson, you should be able to:

- Describe compound triggers
- Describe mutating tables
- Create triggers on DDL statements
- Create triggers on system events
- Display trigger information

---

## Compound DML Triggers

Want a BEFORE statement trigger, AFTER statement trigger, BEFORE row trigger, and AFTER row trigger -- all tied to the same event -- without creating four separate triggers? That is a compound trigger. One trigger, four possible timing points, shared state, fewer headaches.

Key facts:

- A compound trigger can include any combination of timing points.
- Each timing point has its own executable section and optional exception handler.
- All timing points can access a shared PL/SQL state.
- The shared state is created when the triggering statement starts and is destroyed when it ends.
- Old/New qualifiers are only valid in row-level sections.

Why use it?

- Share data across timing points (collections, counters, cached values).
- Accumulate rows for batch processing.
- Avoid mutating table errors.

---

## Mutating Tables (aka the "No, You May Not Query Me While I'm Changing" Rule)

A mutating table is one being modified by an INSERT, UPDATE, or DELETE. Row-level triggers cannot query or modify the same table because Oracle refuses to let you see a table mid-change and pretend it is stable.

Classic failure:

- Row-level trigger queries the same table it is updating.
- Boom: mutating table error.

Classic fix:

- Use a compound trigger.
- Gather data in BEFORE STATEMENT.
- Validate in AFTER EACH ROW using cached data.

---

## Compound Trigger Pattern (High-Level)

1. BEFORE STATEMENT:
   - Load min/max salaries per department into collections.
2. AFTER EACH ROW:
   - Validate :NEW.salary against the cached ranges.
3. Raise error if out of range.

This pattern avoids mutating table errors because you are not querying the table during row changes -- you are using the cached data instead.

---

## DDL Triggers (Schema Level)

DDL triggers fire on CREATE, ALTER, or DROP. They can live on a schema or the database.

Example use:

- Block dropping objects in a schema:
  - BEFORE DROP ON schema
  - RAISE_APPLICATION_ERROR if someone tries to drop.

---

## Database Event Triggers

These fire on system events like LOGON, LOGOFF, STARTUP, SHUTDOWN, or SERVERERROR.

Example:

- After logon, write to an audit table.
- Before logoff, log the event.

These are great for auditing, bad for pretending logs do not exist.

---

## Trigger Design Guidelines

- Do not duplicate built-in constraints with triggers.
- Avoid recursive trigger chains.
- If logic is large, call a procedure from the trigger.
- Keep row-level trigger logic short and deterministic.
- Test everything, including non-triggering cases.

---

## Quick Quiz

Which are true for triggers?

- Created with CREATE TRIGGER -- true
- Stored in USER_TRIGGERS -- true
- Explicitly invoked -- false
- Implicitly invoked by events -- true
- COMMIT/ROLLBACK inside trigger -- not allowed

So yes, triggers are the database equivalent of automatic doors. You do not "call" them -- you just trigger them.

---

## Wrap-Up

You should now be able to:

- Explain compound triggers and shared state
- Describe mutating tables and how to avoid the error
- Create DDL triggers
- Create system event triggers
- Inspect trigger metadata in data dictionary views

Next up: Practice 19, where you create advanced triggers, intentionally hit mutating table errors, and then fix them like responsible adults with caffeine.
