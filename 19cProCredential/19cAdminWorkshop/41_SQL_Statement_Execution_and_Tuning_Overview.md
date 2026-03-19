# 41 - SQL Statement Execution and Tuning Overview

Chapter 4 is where performance tuning stops being a noble abstract principle
and becomes a direct interrogation of SQL itself: how it parses, how Oracle
chooses a plan, where the data goes, why statements wait, and which advisors are
there to help when the optimizer has clearly had a difficult day.

This lesson gives the big-picture map of SQL execution and tuning in Oracle
Database 19c. It is not yet the full forensic war room, but it is the part
where you learn which levers actually exist before you start yanking on them
like a panicked train conductor.

---

## 1. SQL Execution Starts With a Question, Not a Plan

A session submits a SQL statement.

Oracle does **not** execute that statement literally in the same form you wrote
it.

SQL is written to be human-readable.
Execution is about machine efficiency.

So the database must determine:

- what objects are involved
- which access paths are possible
- how joins should happen
- how much data is likely to be returned
- what plan is cheapest and most appropriate for the statement

That process is optimization.

The result is an **execution plan**, which is the set of steps Oracle will use
to get the data back to the session.

If you ever forget this, remember:

- SQL text is the request
- execution plan is the method
- performance pain usually lives in the method

---

## 2. What Happens During SQL Execution

At a high level, SQL execution looks like this:

1. session submits SQL
2. Oracle parses and optimizes the statement
3. Oracle selects or reuses an execution plan
4. the server process executes the plan
5. required blocks are read into the buffer cache if not already there
6. rows are processed and returned to the session

Important memory areas involved:

- **shared pool / SQL area**
  This is where parsed SQL and execution plan information can be reused.

- **buffer cache**
  This is where Oracle keeps database blocks in memory after reading them from
  data files.

Why the buffer cache matters:

- data is stored in Oracle blocks, not isolated single rows
- if Oracle brings one required block into memory, other rows in that same block
  may already be available for later access
- that reuse is one of the reasons memory matters so much to SQL response time

So when a statement runs, Oracle is not just "fetching a row."
It is navigating plans, blocks, cache, memory, I/O, and concurrency, all while
trying to look calm about it.

---

## 3. What the Optimizer Is Actually Doing

The optimizer is Oracle's plan-selection engine.

Its job is to come up with the most efficient execution path based on available
information.

That includes choices such as:

- full table scan versus index access
- rowid access after index lookup
- join methods
- join order
- predicate handling
- whether query transformations should be applied

The optimizer relies on:

- object statistics
- system statistics
- metadata
- available memory and resource context
- applicable optimizer features and parameters

If that information is accurate, the optimizer has a fighting chance.
If that information is bad, stale, or incomplete, the optimizer starts making
very confident mistakes.

---

## 4. Query Transformation Comes First

Before Oracle settles on a final plan, it may transform the query.

This means Oracle can rewrite the statement into a form that is easier or more
efficient to optimize.

Examples include:

- simplifying expressions
- rewriting subqueries
- changing join structures
- rewriting access to use a materialized view instead of base tables when
  appropriate

Classic example:

- your SQL joins tables `A`, `B`, and `C`
- Oracle sees a materialized view that already contains that joined result
- Oracle rewrites the query to use the materialized view instead

That is not Oracle being sneaky.
That is Oracle doing exactly what you pay it to do: less unnecessary work.

---

## 5. Costing, Estimation, and Plan Choice

Once the statement is in a workable form, the optimizer estimates possible
plans.

This involves two big ideas:

### Estimation

Oracle estimates things like:

- cardinality
- selectivity
- row counts
- bytes returned
- join sizes

### Costing

Oracle assigns a cost to candidate plans based on expected resource use.

Then it compares plans and chooses the best available option according to the
optimizer model.

This is why good object statistics matter so much:

- row counts
- number of distinct values
- number of nulls
- column distribution
- index structure and size
- correlated column behavior

Bad stats mean bad estimates.
Bad estimates mean bad cost calculations.
Bad cost calculations mean ugly plans wearing the badge of official approval.

---

## 6. Hard Parse, Soft Parse, and Why Reuse Matters

One of the most expensive parts of SQL lifecycle is generating a plan from
scratch.

### Hard parse

A hard parse happens when Oracle must fully parse and optimize a statement and
generate a new execution plan.

This can be expensive because Oracle must:

- validate syntax and objects
- check privileges
- transform the query
- estimate alternatives
- cost multiple plans
- choose and store the final plan

### Soft parse

A soft parse happens when Oracle can reuse an existing cursor and execution
plan.

Soft parse is much cheaper.

That is why shared SQL and plan reuse matter so much. You want Oracle reusing
good plans, not constantly rebuilding the same plan while pretending this is an
efficient lifestyle.

---

## 7. SQL Plan Baselines, SQL Profiles, and Optimizer Goals Are Not the Same Thing

This is where people mash three different concepts into one unpleasant blob.

Do not do that.

### SQL plan baseline

A SQL plan baseline is part of SQL Plan Management.

It preserves accepted plans that the optimizer is allowed to use for a given SQL
statement, helping avoid regressions when new statistics, upgrades, or parameter
changes would otherwise lead Oracle into a worse plan.

Use this concept when the problem is:

- "Oracle keeps changing to a bad plan"

### SQL profile

A SQL profile is auxiliary information created through SQL Tuning Advisor that
helps the optimizer improve its estimates and choose a better plan.

Use this concept when the problem is:

- "Oracle is estimating badly and picking the wrong path"

Important correction:

- a SQL profile does **not** mean "return 100 rows at a time so the web page can
  paint faster"

That idea belongs much more to optimizer goals like `FIRST_ROWS(n)` versus
`ALL_ROWS`, not to the basic definition of a SQL profile.

### Optimizer goal

Optimizer goals influence whether Oracle should favor:

- fast return of the first rows, such as `FIRST_ROWS(n)`
- best total throughput for the whole result set, such as `ALL_ROWS`

This is where the transcript's web-page example actually belongs.

If you want the first page to show quickly, that is an optimizer-goal discussion
far more than a SQL-profile definition.

---

## 8. What SQL Tuning Actually Means

SQL tuning is not one thing.
It is several related investigations around one question:

- why did this SQL behave the way it did?

The main tuning areas discussed here are:

### Execution plan quality

- did Oracle choose the right plan?
- did it use the right access path?
- did statistics lead it somewhere foolish?

### Resource availability

- was memory insufficient?
- was the instance under pressure?
- were SGA or PGA limitations contributing?

### Concurrency and locking

- was the statement waiting because data was locked?
- was the plan fine but the statement stalled behind another session?

That last point matters because sometimes the SQL is innocent and the real
problem is that someone else is sitting on the row like a dragon on a pile of
transactional gold.

---

## 9. Object Statistics: The Foundation Under the Optimizer

Optimizer statistics describe the data and objects Oracle is trying to access.

They include things such as:

- number of rows in a table
- number of blocks
- column selectivity
- repeated values
- null counts
- index details
- multi-column correlations when extended stats are used

You manage them with `DBMS_STATS`.

Typical manual collection looks like:

```sql
BEGIN
  DBMS_STATS.GATHER_TABLE_STATS(
    ownname          => 'HR',
    tabname          => 'EMPLOYEES',
    cascade          => TRUE
  );
END;
/
```

Why `CASCADE` matters:

- table stats may have changed
- related index stats may also need updating

Useful views include:

- `DBA_TAB_STATISTICS`
- `DBA_TAB_STAT_PREFS`

And yes, if the table changed a lot and stats were not refreshed, the optimizer
can absolutely choose a plan that made sense for yesterday and is absurd for
today.

---

## 10. Optimizer Statistics Advisor

Optimizer Statistics Advisor helps evaluate how statistics are being gathered and
whether the current strategy follows Oracle best practices.

It can help answer questions such as:

- Is the stale percentage sensible?
- Should extended or multi-column stats be considered?
- Is sample size appropriate?
- Are stats preferences set badly?
- Is the collection approach mismatched to the workload?

Important point:

- Optimizer Statistics Advisor does **not** gather a replacement set of stats
  itself
- it analyzes your current stats strategy and recommends improvements

This is useful because "collect stats" is not the same as "collect the right
stats in the right way."

Sometimes Oracle is not telling you the data is bad.
It is telling you your **method** for measuring the data is bad, which is much
more insulting and often correct.

---

## 11. SQL Tuning Advisor

SQL Tuning Advisor focuses on **individual SQL statements**.

It can recommend things such as:

- gather better statistics
- accept a SQL profile
- consider an index
- evaluate an alternative plan
- restructure the SQL when the statement is too complex or inefficient

Important caution:

When SQL Tuning Advisor says "restructure the SQL," that does not automatically
mean the statement is stupid.

Sometimes it means:

- the statement is too complex
- the nested views and subqueries are too layered
- the optimizer cannot reason about it effectively as written

That is your cue to tune inside-out:

1. tune inner subquery or view
2. tune the next layer
3. then tune the outer statement

Because trying to tune a monstrous all-in-one statement in one pass is a little
like trying to debug a cathedral by staring at the roof.

---

## 12. SQL Access Advisor

SQL Tuning Advisor is about improving a statement.
SQL Access Advisor is about improving a **workload**.

That difference matters.

Example:

- SQL Tuning Advisor says an index would make one query faster
- you create the index
- the query improves
- but the wider workload now suffers because that column is updated constantly

That is where SQL Access Advisor comes in.

It evaluates workload-level implications and can recommend:

- indexes
- materialized views
- materialized view logs
- partitioning strategies

This is how you avoid the classic tuning disaster:

- one statement got better
- everything else got worse

Which is technically a change, but not one anyone should celebrate.

---

## 13. SQL Tuning Sets and Workload Analysis

When you want to analyze a SQL workload rather than one statement, Oracle lets
you capture that workload in a **SQL tuning set**.

That workload can then be handed to the right tools for broader analysis.

This is useful when you need to understand:

- repeated SQL
- duplicate patterns
- common access paths
- workload-wide effects of design changes

A SQL tuning set is basically Oracle's way of saying:

- stop tuning in isolation
- bring the whole crowd into the room

---

## 14. Adaptive Behavior, Statistics Feedback, and Dynamic Statistics

In newer Oracle releases, the optimizer is not completely frozen once the first
plan is generated.

Relevant adaptive ideas include:

### Adaptive plans

Oracle may choose between subplans during execution based on what it actually
sees at runtime.

### Statistics feedback

If execution reveals that estimates were materially wrong, Oracle can use that
feedback for later executions.

### Dynamic statistics

Formerly called dynamic sampling, this is when Oracle samples blocks during
optimization because existing stats are missing, stale, or insufficient.

### SQL plan directives

These are optimizer notes about repeated estimation problems that can influence
future optimization and statistics gathering behavior.

Important 19c nuance:

- adaptive statistics are more limited in 19c and disabled by default through
  `OPTIMIZER_ADAPTIVE_STATISTICS = FALSE`
- SQL plan directives can still exist, but their role is not the same as the
  older "optimizer writes a permanent magical hint and saves the day forever"
  classroom story

So yes, Oracle can react to feedback.
No, it is not literally scribbling little rescue hints in the margins like a
desperate student during finals.

---

## 15. ADDM and Automatic SQL Tuning

ADDM can identify expensive SQL based on observed resource consumption and
elapsed time patterns.

From there:

- SQL Tuning Advisor can analyze statements in detail
- Automatic SQL Tuning can review recurring high-load SQL during maintenance
  windows

This is part of the proactive model:

- detect problematic SQL
- analyze it
- recommend targeted changes

In other words, Oracle tries not to wait until you personally notice that one
query has turned into a small geological era.

---

## 16. SQL Performance Analyzer Is Different, and Licensed Separately

SQL Performance Analyzer is part of **Real Application Testing**, not the basic
everyday advisor set.

It is used to compare SQL behavior before and after major changes such as:

- database upgrades
- patching
- operating system changes
- hardware changes
- major application rollout changes

It runs SQL in both environments and compares the results and performance.

Important note:

- visible in tooling does **not** mean automatically licensed for use

Oracle is extremely fond of installing things you still need separate permission
to touch, which is a marvelous business model and a dreadful licensing trap.

---

## 17. What You Can See in `V$SQL` and the GUI Tools

At the command line, `V$SQL` is one of the key views for current SQL in memory.

It gives you information such as:

- SQL text
- executions
- parse/load information
- versions
- elapsed and resource usage statistics

Basic example:

```sql
DESC V$SQL
```

That view is where you stop arguing in generalities and start looking at actual
statement behavior.

### EM Express

EM Express gives you useful entry points for:

- SQL Tuning Advisor
- tuning task results
- some automatic tuning configuration

But it is not the full advisor universe.

### Cloud Control

Cloud Control provides the broader advisor home, including:

- ADDM
- SQL Tuning Advisor
- SQL Access Advisor
- Segment Advisor
- Memory Advisors
- Undo and recovery-related advisors
- SQL Performance Analyzer where licensed

So if EM Express is the local dashboard, Cloud Control is the bigger command
center with more buttons, more reach, and more ways to get yourself into
interesting decisions.

---

## 18. Key Takeaways

- SQL execution starts with optimization, not immediate row retrieval.
- The optimizer chooses a plan based on query transformation, estimation, and
  cost calculations.
- Good object statistics are essential to good plan selection.
- Hard parsing is expensive; plan reuse matters.
- SQL plan baselines, SQL profiles, and optimizer goals solve different
  problems and should not be confused with one another.
- SQL Tuning Advisor works at the statement level.
- SQL Access Advisor works at the workload level.
- Optimizer Statistics Advisor helps improve how statistics are gathered and
  configured.
- Adaptive plans, statistics feedback, dynamic statistics, and SQL plan
  directives help Oracle react when estimates are wrong.
- SQL Performance Analyzer is powerful, but separately licensed.

---

## 19. Wrap-Up

This chapter gives you the operational map of SQL tuning in Oracle: how a
statement becomes a plan, where statistics and memory fit into the story, why
hard parsing is expensive, how Oracle adapts when its assumptions are wrong, and
which advisors are responsible for which kind of tuning problem. The next step
is to take these ideas out of the whiteboard universe and into actual tools,
actual reports, and actual evidence, which is where SQL tuning becomes much less
theoretical and much more fun in the same way surgery is "fun" when you have the
right instruments.
