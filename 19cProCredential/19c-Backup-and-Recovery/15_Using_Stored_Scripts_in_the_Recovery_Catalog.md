## Lesson 15 - Using Stored Scripts in the Recovery Catalog (in which RMAN finally lets you stop retyping the same commands like a medieval scribe)

And look, stored scripts are one of the saner parts of the recovery catalog. Instead of rebuilding the same `BACKUP DATABASE PLUS ARCHIVELOG` block from memory every time, you store it once, name it, and run it again later like a civilized adult.

By the end of this lesson, you should be able to:

- Explain the difference between local and global RMAN stored scripts
- Create stored scripts in the recovery catalog
- Execute stored scripts from RMAN
- Print, list, replace, and delete stored scripts
- Use file-based script editing when the RMAN prompt becomes too annoying to tolerate

---

## 1. What Stored Scripts Are

An RMAN stored script is a named collection of RMAN commands stored in the
recovery catalog for later execution.

That means:

- the script lives in the recovery catalog, not in the target control file
- you can call it by name
- you do not have to rebuild the command block every single time

Stored scripts are made of commands that are valid inside a `RUN` block.

Important Oracle rule:

- you can put RMAN commands and `SQL` commands inside a stored script
- you cannot put a `RUN` command inside a stored script

Because the stored script itself is effectively meant to become content inside a
`RUN` context, not recursively nest itself into madness.

---

## 2. Local Scripts vs Global Scripts

RMAN gives you two flavors:

- **local scripts**
- **global scripts**

### Local script

A local script is tied to the current target database.

That means:

- it belongs to one registered target
- it is useful for target-specific backup and maintenance logic

Example use:

- a script that knows the backup pattern for `orclcdb` only

### Global script

A global script can be used with any database registered in the recovery
catalog.

That means:

- it is shared
- it is good for standard procedures used across many databases

Example use:

- a standard full backup policy
- a common maintenance routine

So local is database-specific.
Global is reusable across registered targets.
One is custom tailoring.
The other is uniform management with fewer opportunities for improvisational
damage.

---

## 3. Why Bother Using Them

Stored scripts help because they give you:

- repeatability
- central storage
- easier reuse
- less typing
- less opportunity for "I swear I usually include that clause"

They are especially helpful when:

- you manage multiple databases
- you already use a recovery catalog
- you want standard backup routines stored centrally

This is one of those features that quietly reduces human error by removing the
need for heroic keyboard performances at 2 a.m.

---

## 4. Create a Local Script

To create a local script, connect to:

- the target database
- the recovery catalog

Then issue `CREATE SCRIPT` at the RMAN prompt:

```rman
CREATE SCRIPT db_arch_bkup
{
  BACKUP DATABASE PLUS ARCHIVELOG;
}
```

Important Oracle rule:

- `CREATE SCRIPT` is executed at the RMAN prompt
- not inside a `RUN` block

And yes, the braces matter.
Oracle is very particular about its punctuation, which is only fair given how
much misery punctuation has caused everyone else.

---

## 5. Create a Global Script

To create a global stored script:

```rman
CREATE GLOBAL SCRIPT global_arch_bkup
{
  BACKUP DATABASE PLUS ARCHIVELOG;
}
```

Use this when the script should be available across registered databases.

This is useful for common tasks, but it also means you should name your scripts
carefully so you do not create local and global scripts with confusingly similar
names and then act surprised when your future self suffers.

---

## 6. Execute Stored Scripts

To execute a stored script, you must be connected to:

- the recovery catalog
- and, for local scripts, the relevant target database

Execution happens inside a `RUN` block:

```rman
RUN {
  EXECUTE SCRIPT db_arch_bkup;
}
```

To execute a global script explicitly:

```rman
RUN {
  EXECUTE GLOBAL SCRIPT global_arch_bkup;
}
```

Important Oracle behavior:

- if you execute `SCRIPT name` and no local script with that name exists for the
  current target, RMAN can fall through to a global script of the same name

That is convenient.
It is also exactly how naming confusion turns into operational nonsense.

So in real life, clear naming conventions are not optional if you want your
blood pressure to remain indoors.

---

## 7. Print a Script or Write It to a File

To display a stored script:

```rman
PRINT SCRIPT db_arch_bkup;
```

To print a global script explicitly:

```rman
PRINT GLOBAL SCRIPT global_arch_bkup;
```

To write a script to a file:

```rman
PRINT SCRIPT db_arch_bkup TO FILE '/tmp/db_arch_bkup.rman';
```

To write a global script to a file:

```rman
PRINT GLOBAL SCRIPT global_arch_bkup TO FILE '/tmp/global_arch_bkup.rman';
```

This is one of the more practical features because it lets you:

- inspect the script cleanly
- edit it in a real editor
- keep a file copy outside RMAN

Which is a wonderful improvement over squinting at a prompt and pretending this
is a pleasant editing experience.

---

## 8. List Stored Scripts

To see what scripts exist:

```rman
LIST SCRIPT NAMES;
```

To list only global scripts:

```rman
LIST GLOBAL SCRIPT NAMES;
```

This is the catalog equivalent of opening the cupboard and checking whether the
thing you think exists actually exists.

Always a good habit.

---

## 9. Replace or Update a Script

To replace a stored script interactively:

```rman
REPLACE SCRIPT db_arch_bkup
{
  BACKUP DATABASE PLUS ARCHIVELOG;
  DELETE OBSOLETE;
}
```

To replace a global script:

```rman
REPLACE GLOBAL SCRIPT global_arch_bkup
{
  BACKUP AS BACKUPSET DATABASE PLUS ARCHIVELOG;
}
```

Important Oracle behavior:

- `REPLACE SCRIPT` updates the script if it exists
- if it does not exist, RMAN creates it

So Oracle chose pragmatism here, which is rare enough that it deserves brief
respect.

---

## 10. Replace a Script from a File

If editing inside RMAN makes you want to throw the terminal into a river, use a
real editor and then load the script from a file.

Example:

```rman
REPLACE SCRIPT db_arch_bkup FROM FILE '/tmp/db_arch_bkup.rman';
```

Or for a global script:

```rman
REPLACE GLOBAL SCRIPT global_arch_bkup FROM FILE '/tmp/global_arch_bkup.rman';
```

Important Oracle formatting rule:

- the first line of the file must begin with `{`
- the last line must contain `}`
- the enclosed commands must be valid inside a `RUN` block

So yes, Oracle will happily let you manage scripts from files.
No, it will not tolerate you casually inventing your own format.

---

## 11. Delete a Script

To delete a script:

```rman
DELETE SCRIPT db_arch_bkup;
```

To delete a global script explicitly:

```rman
DELETE GLOBAL SCRIPT global_arch_bkup;
```

Important Oracle nuance:

- if you do **not** specify `GLOBAL`, RMAN first looks for a local script
- if no local script exists, RMAN may delete a global script with the same name

That is technically useful and emotionally dangerous.

So while the source makes it sound like "you don't have to use global," the
better operational advice is:

- use `GLOBAL` when you mean global
- use clear naming conventions
- do not leave this to Oracle's fallback behavior unless you enjoy surprise
  administrative vandalism

---

## 12. Demo-Style Script Lifecycle

The demo flow really boils down to this sequence:

1. connect to target and catalog
2. create a script
3. print it
4. replace it
5. print it again
6. execute it later from a `RUN` block

Representative example:

```text
rman target "/ as sysbackup" catalog rcatowner@rcatpdb
```

Then:

```rman
CREATE SCRIPT db_arch_bkup
{
  BACKUP DATABASE PLUS ARCHIVELOG;
}

PRINT SCRIPT db_arch_bkup;

REPLACE SCRIPT db_arch_bkup
{
  BACKUP DATABASE PLUS ARCHIVELOG;
  DELETE OBSOLETE;
}

PRINT SCRIPT db_arch_bkup;
```

Then later:

```rman
RUN {
  EXECUTE SCRIPT db_arch_bkup;
}
```

That is the essential stored-script lifecycle, minus the classroom pause where
someone inevitably mistypes the script name and then blames Oracle personally.

---

## 13. Practice-Style Notes

The practice shows two especially useful habits:

### First habit: check whether scripts already exist

```rman
LIST SCRIPT NAMES;
```

That avoids the classic "I thought I was creating this" moment when you are
actually colliding with something already in the catalog.

### Second habit: print after replace

```rman
PRINT SCRIPT db_arch_bkup;
```

That confirms the script contents are what you think they are before you execute
them against a real target.

This may sound embarrassingly basic.
It is also how you prevent "I meant to back up the database" from becoming
"apparently I also delete obsolete now, surprise."

---

## 14. Practical Takeaways

Stored scripts are best when:

- names are clear
- local and global usage is intentional
- common procedures are standardized
- file-based editing is used when scripts get longer

They are worst when:

- everything is named badly
- nobody remembers which scripts are local versus global
- people trust fallback behavior more than explicit commands

Which is to say, the tool is fine.
The species using it remains the variable.

---

## 15. Wrap-Up

Using stored scripts in the recovery catalog means you can:

- create local or global RMAN procedures
- execute them consistently
- print and export them
- replace them interactively or from files
- delete them cleanly when they are no longer needed

At this point, RMAN script management stops being a pile of repeated commands
and becomes something much closer to centralized operational sanity.
