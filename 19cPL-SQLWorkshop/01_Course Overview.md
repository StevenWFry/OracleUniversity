## Lesson 0 - Course Overview (in which PL/SQL learns to talk and also gets very chatty)

Welcome to **Oracle Database 19c: PL/SQL Workshop**, where SQL stops being a polite, declarative librarian and starts becoming a full-blown narrator with opinions, variables, and a questionable sense of timing.

In this course, you will:

- Learn the **three-part structure** of a PL/SQL block: **declaration**, **execution**, and **exception**.
- Declare variables, assign values, and write anonymous PL/SQL blocks without accidentally bricking your session.
- Embed your **SELECT, INSERT, UPDATE, and DELETE** statements inside PL/SQL like a civilized database citizen.

---

## 1. Programming with PL/SQL (where the loops live and occasionally escape)

You will write control structures and work with composite data types:

- **Loops**: basic, `WHILE`, and `FOR` loops.
- **Composite types**: PL/SQL records and arrays (a.k.a. PL/SQL tables).
- **Explicit cursors**: how to declare them, open them, and march through them with a loop like a tiny database parade.

---

## 2. Working with Your PL/SQL Code (error handling, but make it graceful)

Because errors will happen, and we should pretend we are prepared:

- Use the **exception section** to trap errors and exit with dignity.
- Build **subprograms**: procedures and functions, and understand why both exist.

---

## 3. Subprograms, Packages, and Debugging (the "grown-up" chapter)

You will:

- Create procedures and functions, and see how they differ.
- Debug with SQL Developer's debugger without crying.
- Build **packages**, understand their parts, and call subprograms inside them.
- Meet a few **Oracle-supplied packages**, because Oracle has already built half your homework.
- Use **dynamic SQL** with `EXECUTE IMMEDIATE` for those times when your SQL needs improv.

---

## 4. Triggers and Performance (where things fire, sometimes on purpose)

You will:

- Build triggers and connect them to **INSERT, UPDATE, DELETE** events.
- Distinguish **trigger headers** from **trigger bodies**, which is somehow both literal and necessary.
- Create system triggers (startup/shutdown) and **INSTEAD OF** triggers for views.
- Apply performance tricks like **bulk binding** and **NOCOPY** to keep your PL/SQL fast.
- Manage compiler warnings and dependencies so your code compiles without screaming.

---

## 5. Audience and Outcomes (who this is for, and what you'll actually be able to do)

This workshop is for developers and data analysts who already know SQL and want to actually **program** with it.

By the end, you should be able to:

- Build full PL/SQL blocks with variables, logic, and error handling.
- Use explicit cursors and loops without inventing new bugs.
- Create and call **procedures, functions, and packages** with proper parameter modes (`IN`, `OUT`, `IN OUT`).
- Implement database triggers, including system and INSTEAD OF triggers.
- Apply performance considerations and choose **native vs interpreted** compilation when it matters.

---

## 6. Prerequisites (what you should already know)

You should be comfortable with:

- SQL SELECT statements
- DML: `INSERT`, `UPDATE`, `DELETE`
- DDL: `CREATE`, `ALTER`, `DROP`
- DCL: `GRANT`, `REVOKE`

---

## 7. What's Next (because there's always another workshop)

After this course, consider:

- **Oracle Database 19c: SQL Tuning Workshop** (execution plans and performance tuning)
- **Oracle Database 19c: Advanced PL/SQL** (LOBs, collections, and application security)

Now take a deep breath, stretch your typing fingers, and let's begin.