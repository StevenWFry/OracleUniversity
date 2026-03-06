## Appendix A-2 - ACFS Replication (in which snapshots become your long-distance disaster-recovery relationship)

And look, this appendix covers ACFS snapshot-based replication: how a primary ACFS file system ships snapshots to a standby file system, and how to keep that pipeline secure, stable, and not on fire.

By the end of this appendix, you should be able to:

- Explain how ACFS snapshot replication works
- Describe primary vs standby replication roles
- Configure the SSH/SSL prerequisites for secure replication
- Initialize replication on standby and primary sides
- Plan capacity so replication does not quietly implode at 2 AM
- Understand how encryption and replication interact

---

## 1. Replication Model: Base Copy + Delta Updates

ACFS replication is snapshot-based.

Operational pattern:

1. Create initial snapshot on primary
2. Transfer it to standby file system
3. Apply future snapshot deltas to that standby copy
4. Take a new recovery snapshot
5. Remove older backup snapshot after successful update

So this is not "copy everything forever." It is "copy baseline once, then send changes," which is exactly what your network team wants to hear.

---

## 2. Primary and Standby Roles

Terms used in this appendix:

- `Primary file system`: source ACFS where writes happen
- `Standby file system`: destination ACFS that receives replication

Important constraints:

- One cluster can host both primary and standby file systems
- One specific file system cannot be both primary and standby at the same time
- Primary and standby targets must be separate file systems

---

## 3. Security Prerequisites

Replication traffic must be secured.

Supported secure transport pattern in this lesson:

- SSL or SSH connectivity between primary and standby sides
- SSH key-based authentication for automated replication workflow

If secure connectivity is not configured, replication setup is dead on arrival.

---

## 4. SSH Setup Pattern (From the Lesson Flow)

High-level sequence:

1. Pick a non-root replication user with required ASM admin context
2. Generate keys if needed (for example, `ssh-keygen -t rsa`)
3. Distribute primary node public keys to standby-side authorized keys
4. Validate primary-to-standby SSH connectivity for each required node

Representative flow:

```bash
ssh-keygen -t rsa
cat /root/.ssh/id_rsa.pub >> /home/repluser/.ssh/authorized_keys
ssh repluser@standby1
ssh repluser@standby2
```

Windows note from this appendix:

- SSH dependencies may require explicit setup because they are not always present by default in older Windows deployment models.

---

## 5. VIP and Host-Key Considerations

When using VIP endpoints for standby access:

- Ensure host-key material includes VIP mappings
- Update key/host records so the replication user can authenticate via VIP path

This is mostly boring identity plumbing, but skipping it gives you "host key verification failed" at exactly the worst possible time.

---

## 6. Standby-Side Initialization

Before standby initialization:

- Create and size the standby ACFS file system correctly
- Mount it on one node for initial replication configuration

Then initialize standby replication metadata using `acfsutil` replication-initiate workflow.

Conceptual pattern (validate exact syntax in your environment):

```bash
acfsutil repl init standby -u repluser /acfs/standby_mount
```

After initialization, mount and expose on remaining nodes as required by your cluster design.

---

## 7. Primary-Side Initialization

On primary side, initiate replication toward the standby target and set cadence.

Conceptual pattern:

```bash
acfsutil repl init primary -i <interval_minutes> -u repluser -m /acfs/standby_mount /acfs/primary_mount
```

Parameter intent from this appendix:

- `-i`: interval between replication snapshots
- `-m`: standby mount mapping (when different/required)
- user/endpoint fields identify secure target path

Replication can run as near-real-time cadence or fixed interval scheduling, depending on setup.

---

## 8. Upgrade and Migration Notes

When moving to modern GI/ASM stacks:

- Ensure Flex ASM-compatible posture
- Keep SSH replication plumbing intact before and after upgrade
- Upgrade primary side and standby side in controlled sequence

Lesson callout:

- Primary and standby upgrade timing should be tight (for example within ~24 hours) so replication continuity is preserved during transition windows.

---

## 9. Capacity Planning: The Part Everyone Forgets

Standby needs enough room for:

- replicated data set
- snapshot delta activity
- backup snapshot lifecycle

If either side runs out of space, replication can halt and recovery becomes manual and unpleasant.

Recommended guardrail from the course flow:

- Enable ACFS auto-resize where appropriate
- Monitor growth trends, not just current free space

---

## 10. Encryption + Replication Together

ACFS replication can operate with ACFS security/encryption policies.

Practical behavior summary:

- Security/encryption configuration applied in source realm can be propagated with replicated content context
- Standby side should be validated for equivalent security policy handling

Do not treat "replicated" and "secured" as separate checkboxes. They are one operational story.

---

## 11. Operational Checklist

1. Build and validate secure SSH/SSL connectivity first
2. Create properly sized standby file system
3. Initialize standby replication
4. Initialize primary replication with interval and target mapping
5. Validate snapshot transfer and apply cycle
6. Verify backup snapshot lifecycle
7. Monitor capacity and auto-resize behavior
8. Rehearse failover/recovery usage of standby copy

Because replication that only works during daytime demos is not replication. It is theater.

---

## 12. Key Takeaways

- ACFS replication is snapshot-and-delta based, not full-copy every cycle.
- Primary and standby must be separate file systems, even if in same cluster.
- Secure transport (SSH/SSL) is mandatory for replication operations.
- `acfsutil` handles standby and primary replication initiation workflows.
- Capacity planning and auto-resize controls are critical to avoid replication stalls.
- Encryption and replication should be designed together, not bolted on later.
