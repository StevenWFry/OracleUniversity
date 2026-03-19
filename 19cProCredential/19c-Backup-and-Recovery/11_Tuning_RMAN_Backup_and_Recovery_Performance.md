## Lesson 11 - Tuning RMAN Backup and Recovery Performance (in which "it finished eventually" is not a performance strategy)

And look, RMAN performance tuning is mostly the art of figuring out which part of the pipeline is being slow, instead of angrily glaring at the whole backup like it personally betrayed you.

By the end of this lesson, you should be able to:

- Monitor RMAN job progress with the right dynamic views
- Use `BACKUP VALIDATE` and `RESTORE VALIDATE` to separate reading problems from writing problems
- Diagnose synchronous and asynchronous I/O bottlenecks
- Tune backup throughput with channels, multiplexing, and memory settings
- Recognize when the real problem is storage, tape, compression, or encryption rather than RMAN itself

---

## 1. What RMAN Tuning Actually Means

The goal is not "make everything faster by typing more commands."

The goal is:

- identify the bottleneck
- confirm which phase is slow
- tune the relevant part instead of randomly touching parameters

Oracle treats an RMAN job as a pipeline:

- read from source files
- copy/process through memory and CPU
- write to disk or tape

Whichever phase is slowest becomes the bottleneck.

That is the part you tune.
Not the whole universe.

---

## 2. Start by Asking Whether You Even Have a Problem

Before tuning anything, determine whether backup or restore performance is
actually below the throughput you should expect from the storage and media in
use.

Useful sanity checks:

```rman
BACKUP VALIDATE DATABASE;
RESTORE VALIDATE DATABASE;
```

What they do:

- `BACKUP VALIDATE` reads the same data a real backup would read, but does not
  write backup output
- `RESTORE VALIDATE` checks whether RMAN can restore from the chosen backups
  without actually restoring files into place

This is useful because it helps you isolate the phase that is slow.

If validation is almost as slow as the real job, the read side is suspicious.
If validation is much faster than the real job, the write or copy side is
usually where the trouble lives.

---

## 3. Reading RMAN Output Without Pretending It Is Poetry

RMAN output already tells you useful things if you stop skimming it like a legal
disclaimer.

During jobs, pay attention to:

- `Starting backup at ...`
- `Finished backup at ...`
- channel allocation and device type
- piece handles and set creation
- warnings about fallback behavior, skipped files, or validation-only activity

The simplest timing trick:

1. run `BACKUP VALIDATE`
2. note start and finish times
3. run the real `BACKUP`
4. compare durations

Interpretation:

- validate time approximately equals real backup time
  read phase is probably the bottleneck
- validate time is much lower than real backup time
  write, copy, tape, compression, or encryption is probably the bottleneck

It is gloriously low-tech, but it works.

---

## 4. Monitoring Progress with `V$SESSION_LONGOPS`

For live progress, use:

- `V$SESSION_LONGOPS`

Oracle records RMAN progress here using:

- **detail rows** for specific files or job steps
- **aggregate rows** for the broader RMAN command

Practical monitoring query:

```sql
SELECT sid,
       serial#,
       context,
       sofar,
       totalwork,
       ROUND(sofar/totalwork*100, 2) AS pct_complete,
       opname
FROM   v$session_longops
WHERE  opname LIKE 'RMAN%'
AND    totalwork <> 0
AND    sofar <> totalwork
ORDER BY sid, serial#;
```

What to look for:

- `SOFAR` increasing steadily means progress is happening
- `TOTALWORK` gives you the scale of the task
- `pct_complete` tells you how far through the current operation you are

If you run the query repeatedly and the percentage barely moves, that is your
hint that the job is stuck on something real rather than merely being dramatic.

---

## 5. Diagnosing I/O Bottlenecks with `V$BACKUP_ASYNC_IO` and `V$BACKUP_SYNC_IO`

These are the two views Oracle wants you to use for RMAN I/O diagnosis:

- `V$BACKUP_ASYNC_IO`
- `V$BACKUP_SYNC_IO`

They contain:

- one row per input file
- one aggregate row
- one row per output backup piece

These views are about ongoing and recently completed RMAN work.
They are not forever-history tables.

### Asynchronous I/O view

`V$BACKUP_ASYNC_IO` is the view to watch when asynchronous I/O is active.

Important columns:

- `TYPE`
- `EFFECTIVE_BYTES_PER_SECOND`
- `IO_COUNT`
- `READY`
- `SHORT_WAITS`
- `LONG_WAITS`
- `LONG_WAIT_TIME_TOTAL`

Representative query:

```sql
SELECT type,
       filename,
       effective_bytes_per_second,
       io_count,
       ready,
       short_waits,
       long_waits,
       long_wait_time_total
FROM   v$backup_async_io
ORDER BY long_waits DESC, io_count DESC;
```

How to interpret it:

- lots of `READY` is good
- low waits are good
- high `LONG_WAITS` relative to `IO_COUNT` usually means the file or device is
  making RMAN wait too often

This is where you find the problem child instead of blaming the entire storage
estate like a medieval villager blaming the weather.

### Synchronous I/O view

`V$BACKUP_SYNC_IO` matters when synchronous I/O is being used.

Important columns:

- `EFFECTIVE_BYTES_PER_SECOND`
- `DISCRETE_BYTES_PER_SECOND`
- `IO_TIME_TOTAL`
- `IO_TIME_MAX`

Representative query:

```sql
SELECT type,
       filename,
       effective_bytes_per_second,
       discrete_bytes_per_second,
       io_count,
       io_time_total,
       io_time_max
FROM   v$backup_sync_io;
```

With synchronous I/O, the main test is simple:

- compare the achieved rate to what the device should be capable of

If the rate is materially below device capability, you have tuning room.
If it is already at or near device throughput, RMAN is not the villain.

---

## 6. Read Bottlenecks vs Write Bottlenecks

### Read bottleneck pattern

If `BACKUP VALIDATE` takes about as long as the real backup, then reading is
probably the problem.

Things to investigate:

- files sitting on slow disks
- ASM layout or rebalance needs
- too little channel parallelism
- poor multiplexing choices
- storage contention from other workloads

Practical responses:

- move chronically slow files
- rebalance ASM if appropriate
- add disks or improve striping where the platform allows it
- tune channel count and multiplexing instead of just wishing harder

### Write or copy bottleneck pattern

If validation is much faster than the real backup, then writing or copying is
probably the problem.

Typical suspects:

- slow output device
- poor tape buffer settings
- too much RMAN compression CPU cost
- too much encryption CPU cost
- network or media-manager limitations

Practical responses:

- review output buffer and device settings
- reduce compression aggressiveness if CPU is the issue
- use `AES128` if encryption overhead is too high and policy allows it
- let the tape system handle tape compression instead of double-compressing with
  RMAN

Oracle is quite clear here:

- if media-manager tape compression is good, prefer that over RMAN binary
  compression for tape workflows

Because compressing data twice is not sophistication.
It is just a slower way to annoy the hardware.

---

## 7. Best-Practice Tuning Levers Oracle Actually Recommends

### Remove `RATE` limits unless you set them deliberately

The `RATE` channel parameter exists to throttle throughput, not improve it.

So if you set `RATE`, and now backups are slow, congratulations:
you successfully did the thing that parameter is for.

Oracle's first recommendation for slow backups is often:

- remove `RATE` from configured or allocated channels

### Use asynchronous I/O when possible

Asynchronous I/O lets a process begin I/O and do other work while waiting for
completion.

That is generally better than synchronous I/O, where one process waits around
for each operation like it has nowhere else to be.

If the operating system does not support native asynchronous I/O, Oracle
documents using:

- `DBWR_IO_SLAVES`

to simulate it.

Important caveat:

- this is only relevant when native async I/O is unavailable
- it is not a magical modern-speed button for every system

### Size `LARGE_POOL_SIZE` appropriately

Oracle recommends sizing `LARGE_POOL_SIZE` when RMAN I/O workers and related
memory consumers need space, especially if `DBWR_IO_SLAVES` is in play.

Why it matters:

- insufficient large pool can push the database into less efficient behavior
- Oracle may fall back toward synchronous I/O paths if it cannot get the memory
  it wants

So this parameter is not glamorous, but neither is discovering your "performance
tuning" accidentally converted the job into a slower I/O path.

---

## 8. Parallelism, Channels, and Tape Streaming

A channel is a stream of bytes to a device.

More channels can improve throughput, but only if the storage and destination
can actually support the extra concurrency.

Use more channels when:

- disk output is striped across multiple disks
- ASM has enough underlying disks to benefit
- compression can use additional CPU effectively
- tape or media-manager architecture can keep multiple streams busy

For tape especially, the goal is to keep the drives streaming.

If tape drives keep stopping and repositioning because they are not being fed
fast enough, performance falls apart in a deeply irritating way.

This is why Oracle recommends:

- adequate channels
- sensible multiplexing
- media-manager buffer tuning
- tape compression instead of RMAN compression when tape is the destination

---

## 9. Multiplexing Without Making It Worse

Multiplexing is the number of input files simultaneously read and written into
the same RMAN backup piece.

The level of multiplexing is determined by the lesser of:

- `MAXOPENFILES`
- the number of files per backup set

And `FILESPERSET` controls the maximum number of input files per backup set.

Defaults that matter:

- `MAXOPENFILES` default is `8`
- `FILESPERSET` default is `64`

### Buffer sizing rules Oracle uses

Oracle documents the read-buffer sizing algorithm like this:

- multiplexing level `<= 4`
  16 buffers of `1 MB`, total `16 MB`
- level `> 4` and `<= 8`
  variable number of `512 KB` buffers, total less than `16 MB`
- level `> 8`
  4 buffers of `128 KB` per file, total `512 KB` per input file

This is why random multiplexing changes can backfire.

If you crank the level thoughtlessly, Oracle adjusts buffers accordingly, and
you may end up with less efficient behavior instead of more.

### Practical guidance

- if striped disk is the issue, increasing multiplexing may help
- if ASM is the source, Oracle often recommends keeping `MAXOPENFILES` low,
  often `1` or `2`
- if one channel is dragging because a single giant file dominates, consider
  multisection backup instead of just stuffing more files into the same piece

So yes, multiplexing is useful.
No, "more" is not automatically "better."

---

## 10. Compression and Encryption Are CPU Conversations

RMAN compression and encryption can both become bottlenecks.

Compression:

- `HIGH` saves more space
- `LOW` and `MEDIUM` usually hurt less
- `BASIC` is available without Advanced Compression but is still CPU work

Encryption:

- stronger algorithms cost more CPU
- `AES128` is the least CPU-intensive Oracle option in this area

If the backup is slow because CPU is saturated, lowering the compression level
or using a lighter encryption algorithm may solve the problem faster than
storage tuning.

Which is inconvenient, because "it is a CPU problem" is much less emotionally
satisfying than blaming storage.

---

## 11. Monitoring Query You Actually Use in Practice

If you just need one practical monitoring query during a running RMAN job, this
is the one that earns its lunch:

```sql
SELECT sid,
       serial#,
       opname,
       sofar,
       totalwork,
       ROUND(sofar/totalwork*100, 2) AS pct_complete
FROM   v$session_longops
WHERE  opname LIKE 'RMAN%'
AND    totalwork > 0
AND    sofar < totalwork
ORDER BY pct_complete;
```

How to use it:

1. start the RMAN job in one session
2. run this query in another session every minute or two
3. watch whether `pct_complete` and `SOFAR` are actually moving

If progress barely changes over repeated checks, you likely have a real
bottleneck instead of mere impatience.

---

## 12. Practice-Style Monitoring Flow

The practice version is intentionally simple:

1. open one session for SQL
2. open another for RMAN
3. start a backup job
4. run the monitoring query from `V$SESSION_LONGOPS`
5. repeat until the job finishes

Representative RMAN command:

```rman
BACKUP DATABASE;
```

Representative monitoring query:

```sql
SELECT sid,
       serial#,
       opname,
       sofar,
       totalwork,
       ROUND(sofar/totalwork*100, 2) AS pct_complete
FROM   v$session_longops
WHERE  opname LIKE 'RMAN%'
AND    totalwork > 0
AND    sofar < totalwork;
```

What the practice is trying to teach:

- not every backup that feels slow is actually stuck
- a real problem shows up as poor movement over time
- you diagnose first, then tune

Which is a vastly better habit than changing five parameters, creating three new
problems, and then announcing that Oracle is mysterious.

---

## 13. Practical Tuning Checklist

1. Compare `BACKUP VALIDATE` time to real backup time.
2. Check `V$SESSION_LONGOPS` for actual progress.
3. Use `V$BACKUP_ASYNC_IO` or `V$BACKUP_SYNC_IO` depending on the I/O mode.
4. Remove accidental `RATE` throttling.
5. Confirm async I/O is available or that the documented fallback path is sized properly.
6. Size `LARGE_POOL_SIZE` appropriately if RMAN worker memory needs it.
7. Tune channels and multiplexing based on storage layout, not optimism.
8. For tape, prefer tape compression over RMAN compression.
9. Revisit encryption and compression levels if CPU is the bottleneck.
10. Treat storage and media-manager tuning as part of the RMAN conversation, because they absolutely are.

---

## 14. Wrap-Up

RMAN tuning is mostly about proving where the slowdown is:

- read side
- write side
- tape or media-manager side
- CPU side
- storage layout side

Once you know that, the fix becomes much more boring and much more effective.

And in Oracle work, boring and effective is usually how you know you are finally
doing it correctly.
