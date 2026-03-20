## Lesson 18 - Diagnosing Failures (in which Oracle gives you logs, incidents, trace files, and a whole directory tree so you can stop guessing)

And look, diagnosing failures is mostly the art of finding the right clue before panic persuades you that all clues are equally meaningful. They are not. Oracle is noisy, but it is not silent. The trick is knowing where it puts the useful noise.

By the end of this lesson, you should be able to:

- Use ADR-based tools and views to locate diagnostic data
- Explain the difference between alerts, problems, and incidents
- Use `ADRCI` and `V$DIAG_INFO` to find failure details quickly
- Understand how Support Workbench and Incident Packaging Service help with Oracle Support cases
- Place Data Recovery Advisor in the right historical context for `19c`

---

## 1. Start with the Goal

When something fails, the real objective is not:

- stare at the error
- feel betrayed
- restart things until the database develops opinions

The real objective is:

1. identify the failure type
2. find the evidence Oracle already recorded
3. determine whether this is corruption, media loss, configuration trouble, or a
   known bug
4. collect the right diagnostics fast enough to fix it or open a support case

Short repair time starts with short diagnosis time.

Which is why rummaging randomly through directories like a raccoon in a bin is
not an actual troubleshooting framework.

---

## 2. The Core Diagnostic Stack

The source material names the right big pieces:

- `ADRCI`
- `V$DIAG_INFO`
- alert log
- incident directories
- trace files
- Support Workbench
- My Oracle Support

Those are the real places you look first.

If the database detects a serious internal error or corruption-related problem,
Oracle usually records:

- an error in the alert log
- one or more trace files
- an incident in the Automatic Diagnostic Repository (`ADR`)

So if you got an alert, Oracle probably already left a breadcrumb trail.
You just need to follow it before someone helpfully rotates, deletes, or ignores
the evidence.

---

## 3. Data Recovery Advisor: Know It, But Do Not Marry It

Older Oracle training presents Data Recovery Advisor (`DRA`) as a front-line
tool for:

- diagnosing failures
- listing failures
- advising repairs
- generating RMAN repair scripts

That was a real feature.

Important `19c` correction:

- `DRA` is deprecated in Oracle Database `19c`
- the RMAN commands `LIST FAILURE`, `ADVISE FAILURE`, `REPAIR FAILURE`, and
  `CHANGE FAILURE` are deprecated
- Oracle documents no replacement feature

So in modern `19c` study notes, the sensible stance is:

- understand what DRA was designed to do
- do not treat it as the center of your operational future

It is now one of those Oracle features that still lingers in the documentation
like a retired official who keeps turning up at ceremonies.

---

## 4. The Automatic Diagnostic Repository (`ADR`)

The `ADR` is Oracle's structured diagnostic storage area.

Its root is controlled by:

```sql
SHOW PARAMETER DIAGNOSTIC_DEST;
```

Under that base location, Oracle builds the diagnostic directory structure for
database components.

For a database instance, the ADR home typically looks like:

```text
<diagnostic_dest>/diag/rdbms/<db_name>/<instance_name>
```

Important distinction:

- `<db_name>` is the database name
- `<instance_name>` is the SID / instance name

That is why the path can look like it repeats itself when names are similar.
It is not Oracle being playful.
It is Oracle storing two different identity layers in one path.

---

## 5. What Lives Under the ADR Home

Key subdirectories include:

- `alert`
- `trace`
- `incident`
- `cdump`
- `hm`

What they are for:

- `alert`
  XML-formatted alert log
- `trace`
  text trace files and text-form alert log
- `incident`
  incident-specific diagnostic dumps
- `cdump`
  core-dump-related data
- `hm`
  Health Monitor output

So when the source says "there is an incident directory with lots of
information," yes, that is real. Oracle is separating serious incident data from
general trace clutter so you do not have to excavate the entire filesystem with
a shovel.

---

## 6. Alert Log: XML, Text, and Why Both Exist

Oracle's alert log in the ADR is fundamentally XML-based.

The XML file lives under the ADR `alert` directory.

A text-format alert log is also available under the `trace` directory for
backward compatibility and human readability.

That means:

- XML alert log = structured source
- text alert log = easier human reading

Oracle's own guidance is that machine parsing should prefer the XML version.

Because the text version is convenient for eyeballs, not for pretending plain
text is a stable data interchange standard.

---

## 7. `V$DIAG_INFO`: Your Shortcut to the Paths

The nicest part of Oracle diagnostics is that you do **not** have to memorize
the path structure.

Use:

```sql
SELECT name, value
FROM   V$DIAG_INFO
ORDER BY name;
```

Useful rows include:

- `Diag Enabled`
- `ADR Base`
- `ADR Home`
- `Diag Trace`
- `Diag Alert`
- `Incident`
- `Health Monitor`
- `Active Problem Count`
- `Active Incident Count`

This is the view that tells you where everything actually is.

So if someone is spelunking through random directories by instinct while
`V$DIAG_INFO` exists, that is not troubleshooting. That is performance art.

---

## 8. `ADRCI`: The Command-Line Tool for Diagnostic Work

`ADRCI` is Oracle's command-line interface for browsing the ADR.

Common commands:

```text
adrci
show base
show homes
show incident
show alert
```

Practical flow:

1. source the correct Oracle environment
2. start `adrci`
3. identify the ADR home
4. inspect incidents or alert output

Representative commands:

```text
adrci
show homes
set home diag/rdbms/orclcdb/orclcdb1
show incident
show alert
```

You can also change the editor used by `ADRCI`:

```text
set editor vi
```

Or set another editor if your system supports it.

So yes, the source is right that `ADRCI` can present alert information in a more
human-readable way. It is basically Oracle's way of saying, "here, have a tool,
because opening XML directly at 3 a.m. is an offense against morale."

---

## 9. Alerts, Problems, and Incidents

Oracle diagnostic language has a hierarchy:

### Alert

An alert is the visible message or notification that something happened.

### Problem

A problem is the underlying issue class.

### Incident

An incident is a single occurrence of a problem.

This matters because:

- one problem can have many incidents
- support packaging often groups incidents by problem

So if you see an `ORA-00600` or `ORA-07445`, Oracle is not just throwing a
number at you. It is usually creating an incident in the ADR, and that incident
can then be packaged for support.

---

## 10. What to Do with Internal Errors

For serious internal errors such as:

- `ORA-00600`
- `ORA-07445`

Oracle's own error-help pages point you toward:

- My Oracle Support lookup tools
- incident packaging via Support Workbench or `ADRCI`

That is because these are often:

- known bugs
- version-specific defects
- issues Oracle Support needs to inspect using incident data

So your job is not to stare heroically at the number and hope intuition blossoms.
Your job is to:

1. identify the exact error
2. gather the incident package
3. check MOS
4. open an SR if needed

Very adult. Very unglamorous. Very effective.

---

## 11. Support Workbench and Incident Packaging Service (`IPS`)

If you are using Enterprise Manager Cloud Control, Support Workbench makes it
easier to:

- inspect incidents
- gather the related files
- build a support package
- attach it to a service request

The packaging engine behind this is the Incident Packaging Service (`IPS`).

With `ADRCI`, you can also use `IPS` directly.

Representative commands:

```text
ips create package problem 1
ips generate package 1 in /tmp
```

The exact package flow depends on what incident or problem you are packaging,
but the big idea is simple:

- bundle the trace files
- include correlated logs and diagnostics
- send them to Oracle Support

This is much better than emailing Oracle a vague summary like:

- "database sad, please advise"

---

## 12. Practical Failure-Diagnosis Flow

When a failure alert arrives, a sane sequence looks like this:

1. check the alert message and exact Oracle error
2. query `V$DIAG_INFO` for paths
3. inspect the alert log
4. inspect the incident directory or relevant trace files
5. use `ADRCI` to list incidents
6. determine whether the issue is:
   corruption
   media failure
   internal error / bug
   logical failure
7. search My Oracle Support for the exact error and version context
8. if needed, package the incident data and open an SR

This is a much better sequence than:

1. restart database
2. panic
3. delete files
4. become the incident

---

## 13. The `HM` Directory and Health Monitor Context

The `hm` directory mentioned in the source refers to Health Monitor output.

This matters because Health Monitor checks can write diagnostic findings that may
support manual troubleshooting.

That directory is not just decorative.
It is one of the places Oracle stores structured diagnostic output when internal
checking tools run.

So if you are trying to understand a failure deeply, the `hm` location can be
part of the evidence trail, especially when the issue is more interesting than
"someone deleted the wrong file again."

---

## 14. Practical Query and Command Set

If you only remember a few commands from this lesson, remember these:

Find the ADR paths:

```sql
SELECT name, value
FROM   V$DIAG_INFO
ORDER BY name;
```

Start `ADRCI` and inspect homes:

```text
adrci
show homes
show incident
show alert
```

And when Oracle Support is needed:

- package incidents with Support Workbench or `ADRCI IPS`
- search the exact Oracle error in My Oracle Support

That is the backbone.
Everything else is elaboration.

---

## 15. Practical Takeaways

When diagnosing failures in `19c`:

- treat ADR as your evidence store
- use `V$DIAG_INFO` to find the right directories fast
- use `ADRCI` to inspect incidents and alert data
- treat DRA as legacy context, not your main plan
- use MOS and incident packaging for internal errors and suspected bugs

This is how you reduce diagnosis time:

- by using the tools Oracle already put in place
- instead of wandering around the filesystem hoping guilt will reveal the truth

---

## 16. Wrap-Up

Diagnosing failures is about getting from alert to evidence quickly:

- alert log
- ADR home
- incident directory
- trace files
- `V$DIAG_INFO`
- `ADRCI`
- MOS and Support Workbench when necessary

Once you know where Oracle hides the facts, failure diagnosis becomes much less
mystical and much more like what it actually is: organized digital detective
work with a lot more acronyms.
