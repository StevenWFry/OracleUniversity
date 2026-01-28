## Lesson 3 - Understanding Instance Recovery (in which SCNs, checkpoints, and log writers keep the database from panic spiraling)

This lesson digs into how Oracle survives a crash, how instance recovery actually works, and how you can tune recovery time without turning checkpointing into a full-time job.

By the end of this lesson, you should be able to:

- Explain the checkpoint and log writer processes
- Describe redo, undo, and archive logging flow
- Compare archivelog vs noarchivelog modes
- Summarize automatic instance recovery steps
- Use MTTR guidance to balance recovery time and workload
- Distinguish complete vs point-in-time recovery requirements

---

## 1. Checkpoints (the heartbeat of consistency)

A checkpoint records progress so recovery knows where to start.

When a commit happens:

- Control files record the SCN and redo byte address
- Data file headers record the same SCN
- A full checkpoint ensures those records are synchronized

No checkpoint, no map. No map, no fast recovery.

---

## 2. Log Writer (LGWR) (the flush that saves your data)

Redo is written to memory first, then flushed to redo logs by LGWR.

LGWR fires when:

- You commit
- The redo buffer is 1/3 full
- Every ~3 seconds
- Before DBWn writes
- Before a clean shutdown

The goal is simple: never let the redo buffer get too full, or your consistency story collapses.

---

## 3. Undo, Redo, and Archive (the three-layer safety net)

When you change data:

- Old values go to undo
- New changes go to redo buffer on commit
- LGWR writes redo to online redo logs
- ARCn copies full redo logs to archive logs

Online redo is circular. Archive logs make history permanent.

---

## 4. Archivelog vs Noarchivelog (the availability fork)

- **Noarchivelog:** recovery only to last backup; database must be closed
- **Archivelog:** recovery to last committed transaction; backups while open

Production systems almost always require archivelog mode. Downtime budgets are not a suggestion.

---

## 5. Automatic Instance Recovery (crash recovery, but polite)

When the instance crashes, Oracle:

1. Rolls forward all changes using redo (committed and uncommitted)
2. Rolls back uncommitted changes using undo

This is fast because it does not compare redo and undo line-by-line. It trusts redo to bring the data files current, then cleans up.

---

## 6. SCN Example (why recovery happens at all)

After a crash:

- Control files might show SCN 143
- Redo logs contain changes up to SCN 143
- Data file headers only show SCN 140

Instance recovery applies redo to close the SCN gap, then rolls back uncommitted changes so users see a consistent database.

---

## 7. MTTR Advisor (how to not over-checkpoint yourself)

Recovery speed depends on the distance between the last checkpoint and the end of redo.

- Short distance = faster recovery
- Too frequent checkpoints = the database does nothing but checkpoint

Oracle’s MTTR Advisor lets you set a target recovery time:

- `0` means disabled (Oracle decides)
- Up to 3600 seconds (1 hour)

Choose a realistic value. “Every millisecond” is not a strategy, it is a performance incident.

---

## 8. Restore vs Recover (two steps, not one)

Recovery has two phases:

- **Restore:** bring back files from backup
- **Recover:** apply redo/undo to reach the desired point

Skipping restore is like skipping the ambulance and trying to heal with vibes.

---

## 9. Complete vs Point-in-Time Recovery (how far back do you want to live)

**Complete recovery**

- Needs data files plus redo/archived redo
- Recovers to the point of failure

**Point-in-time recovery**

- Recovers to a chosen past time
- Discards changes after that point
- Often uses archived logs instead of current online redo

Both are valid. One preserves everything, the other preserves sanity.

---

## 10. Protection Options by Recovery Window (choose your weapon)

If you need:

- **Seconds to minutes:** Data Guard / Active Data Guard
- **Minutes to hours:** Flashback
- **Hours to days:** RMAN and tape (Oracle Secure Backup)
- **Deep diagnostics:** Data Recovery Advisor

Pick based on your SLA, not your optimism.

---

## 11. Wrap-Up (the crash course is over)

You now understand instance recovery mechanics, checkpoint and redo flow, MTTR tuning, and the difference between complete and point-in-time recovery. Next up: more on recovery choices and how to apply them in practice.
