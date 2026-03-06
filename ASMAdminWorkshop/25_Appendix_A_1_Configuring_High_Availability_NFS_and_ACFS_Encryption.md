## Appendix A-1 - Configuring High Availability NFS and ACFS Encryption (in which your file system gets a VIP and a bodyguard)

And look, this appendix covers two advanced ACFS topics:

- exporting ACFS as highly available NFS service in a cluster
- encrypting ACFS content for at-rest protection

By the end of this appendix, you should be able to:

- Describe how HA NFS is layered on top of ACFS in clustered environments
- Register ACFS/NFS resources with cluster control tooling
- Explain how ACFS encryption keys are structured and managed
- Enable ACFS encryption for target directories
- Locate encryption diagnostics and validate encryption status

---

## 1. HA NFS on Top of ACFS: Architecture Summary

Flow:

1. ACFS file system exists on ADVM/ASM storage
2. Cluster creates HA network endpoint/resource for NFS service
3. ACFS path is exported as NFS share resource
4. External clients mount via HA endpoint
5. Cluster migrates service on node failure

Important separation of responsibility:

- ASM provides storage
- Cluster resource stack provides HA NFS service orchestration

So ASM is the warehouse. Clusterware is the delivery company.

---

## 2. HA NFS Configuration Pattern

High-level command flow (conceptual):

1. Ensure NFS service prerequisites on nodes
2. Register HA endpoint/resource (VIP-style service resource)
3. Register ACFS file system as cluster resource
4. Add export definition for ACFS path
5. Start/enable export resource

Tooling:

- `srvctl` for add/start/status resource operations
- `crsctl` for deeper resource inspection/adjustment where needed

The exact CLI syntax can vary by release/platform packaging, but the orchestration model stays the same.

---

## 3. Why HA NFS Matters

Without HA layer:

- node hosting export fails -> export disappears

With HA layer:

- export service fails over to surviving node(s)
- clients keep service continuity with less interruption

This is especially useful when ACFS must be consumed outside the cluster boundary.

---

## 4. ACFS Encryption Overview

ACFS encryption protects file data at rest within ACFS paths.

Scope:

- can be enabled at directory level
- files created under encrypted directories inherit encryption behavior
- encrypted and unencrypted data can coexist on same volume/mount

Use this when ACFS carries sensitive user/application content and "trust the mount point" is not enough.

---

## 5. Key Model (Dual-Key Pattern)

This appendix describes two key layers:

- File encryption key (encrypts file data)
- Volume encryption key (protects file-encryption keys)

This mirrors standard layered key-protection design used in other Oracle security features.

Key management integration can involve Oracle key-management systems/vault workflows depending on environment policy.

---

## 6. Enabling ACFS Encryption

Operational sequence from the lesson:

1. Initialize encryption on target ACFS mount
2. Configure encryption parameters (algorithm/key size policy)
3. Enable encryption on directory (optionally recursive)
4. Verify status

Command-style pattern:

```bash
acfsutil encrypt init /acfs/mountpoint
acfsutil encrypt set -a <algorithm> -k <key_bits> /acfs/mountpoint
acfsutil encrypt on -r /acfs/mountpoint/sensitive_dir
acfsutil encrypt info /acfs/mountpoint/sensitive_dir
```

(Option flags vary by version/platform; validate with local help before execution.)

---

## 7. Logging and Diagnostics

Encryption diagnostics are written under ACFS security/log paths and cluster log locations.

Use these when:

- encryption enablement fails
- key operations are rejected
- policy validation is needed for auditing/support

Practical note from this appendix:

- after enabling encryption, ensure cluster-critical metadata backup policy captures the updated security state

Because losing key/metadata linkage is an expensive way to test your incident-response plan.

---

## 8. Compatibility Notes

From this appendix context:

- ACFS/ADVM feature baseline is tied to modern compatibility levels (historically 11.2+ for initial introduction)
- always align with current release support matrix and target platform documentation

In other words, do not copy/paste old runbooks into new clusters and call it architecture.

---

## 9. Operational Guidance

For production rollout:

1. Build ACFS first, validate mount and permissions
2. Register HA NFS resources in cluster and test failover
3. Enable encryption on scoped directories, not blindly everywhere
4. Validate export + encryption behavior from client perspective
5. Capture logs, resource state, and key-management references for ops handoff

Do rehearsals. Do not "first-test-in-production" this stack.

---

## 10. Key Takeaways

- HA NFS for ACFS is a cluster resource pattern, not a standalone ASM feature.
- ACFS encryption supports directory-scoped, inheritable at-rest protection.
- Encryption uses layered keys and requires disciplined key/metadata management.
- `srvctl`/`crsctl` handle HA resource lifecycle; `acfsutil` handles ACFS encryption operations.
- Validate failover, export behavior, and encryption status together before calling the deployment done.
