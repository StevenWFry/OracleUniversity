# 9 - Configuring a Shared Server Architecture (One Kitchen, Many Tables)

Chapter 4 is where we decide whether every connection gets its own private
worker, or whether we run a coordinated shared-service model that handles lots
of similar requests more efficiently.

---

## 1. Dedicated vs Shared: The Real Decision

Oracle can serve client connections in two broad ways:

- Dedicated server:
 - one connection maps to one server process/session path
 - each session gets its own full PGA context

- Shared server:
 - many client connections are funneled through dispatchers/shared servers
 - session state is shared through SGA structures, not duplicated everywhere

When shared server is a good fit:

- high-volume, similar request patterns (web/OLTP style interactions)
- predictable SQL behavior across sessions
- many users doing similar work, not wildly mixed workloads

When dedicated is a better fit:

- administration tasks
- backup/recovery operations
- heavy reporting and mixed SQL patterns
- batch/warehouse jobs with long, varied execution behavior

---

## 2. Why Shared Server Can Save Memory

In dedicated mode:

- PGA includes user session state (UGA) + work areas
- this repeats per connection

In shared mode:

- UGA moves into the SGA large pool
- server process PGA mostly keeps work areas
- multiple connections can share session-state structures

Result:

- less redundant per-connection memory
- capacity to support more concurrent connections
- or ability to allocate more work area where needed

---

## 3. Restaurant Analogy (Because Oracle Networking Loves Metaphors)

Dedicated model:

- one worker handles one customer flow directly

Shared model:

- listener = host/hostess routing incoming clients
- dispatcher = waiter coordinating requests
- shared server processes = staff that actually execute tasks

The important part is load distribution:

- listener/dispatchers steer requests to less busy paths
- work is processed from shared queues, not strict one-client/one-process pairing

---

## 4. Core Shared Server Parameters and Controls

Key knobs discussed:

- `SHARED_SERVERS`
 - minimum number of shared server processes to start

- `MAX_SHARED_SERVERS`
 - maximum shared servers allowed

- `DISPATCHERS`
 - defines dispatcher protocols/services for shared connections

- `SHARED_SERVER_SESSIONS`
 - optional cap on shared server sessions

- `CIRCUITS`
 - virtual circuit capacity for shared server infrastructure

Operational point:

- dispatchers and shared servers are related but not 1:1.
- one dispatcher can handle multiple client connections.

---

## 5. Service-Based Connection Design

Shared vs dedicated is often controlled per service.

That means a single database can expose:

- one service tuned for shared server application traffic
- another service using dedicated servers for reporting/admin tasks

This is why service design is central to connection architecture, not just
"what host and port do we use."

---

## 6. Demo Notes from the Lesson

The transcript demo highlighted:

- shared server is already in use for EM Express-style service patterns
- parameter checks showed shared-server-related values and defaults
- dispatcher config showed a TCP dispatcher tied to an XDB-related service

Example checks used in spirit:

```sql
SHOW PARAMETER shared;
SHOW PARAMETER dispatchers;
```

Takeaway:

- shared server behavior is active only where configured services/dispatchers
 are defined.
- other services can still run dedicated.

---

## 7. Practical Rule of Thumb

Use shared server when:

- many concurrent sessions do short, similar, repeatable work

Use dedicated when:

- session behavior is long-running, varied, or operationally sensitive

Hybrid service strategy is usually the grown-up answer in real systems.
