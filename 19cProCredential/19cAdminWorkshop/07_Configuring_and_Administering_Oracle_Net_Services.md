# 7 - Configuring and Administering Oracle Net Services (How Sessions Actually Get To Your Database)

And look, you can build the most beautifully tuned database instance on earth,
but if nobody can connect to it, you have essentially created a very expensive
paperweight with redo logs.

This chapter is about the plumbing between the user process on the client and
the session on the instance: **Oracle Net Services**.

We will cover:

- How clients find listeners and services
- Naming methods: Easy Connect, `tnsnames.ora`, and directory naming
- What `sqlnet.ora` really controls
- Listener placement and security
- Dedicated vs shared server connections
- The zoo of network tools (Net Manager, NETCA, EM, `lsnrctl`)

By the end of this module, you should be able to:

- Explain how clients connect to database services through listeners
- Compare naming methods (Easy Connect, local, directory)
- Describe Oracle Net configuration files and tools
- Differentiate dedicated vs shared server architecture

---

## 1. Big Picture: From Client Click To Database Session

When a user "connects to Oracle," several things happen under the hood:

1. A **client application** runs on a user machine (SQL Developer, app server, etc.).
2. That application starts a **user process** on the client host.
3. The user process needs to find a **listener** for the target database service.
4. The listener spawns or attaches a **server process** on the database host.
5. The server process creates/uses a **session** inside the instance.

Terminology:

- **Connection**: the network path between client and listener/server.
- **Session**: the logical conversation inside the instance.

In dedicated server mode, one connection = one session. In shared server mode,
multiple connections may share a smaller number of sessions.

---

## 2. Services in a Multitenant World (CDB/PDB)

In 19c multitenant:

- You still have one **instance** and one **CDB** (root database).
- Inside the CDB you have multiple **PDBs** (pluggable databases).
- Each PDB gets its own **service name** that clients can connect to.

Common service concepts:

- The **database name** is exposed as a service.
- The **instance name (SID)**, e.g. `ORCL`, is also exposed as a service.
- Each **PDB name** (e.g. `ORCLPDB1`) is or can be a service.
- You can create extra services with `DBMS_SERVICE`.
 - In RAC you usually use `srvctl` (Server Control) to create and manage
 services; `srvctl` calls `DBMS_SERVICE` behind the scenes and keeps the
 cluster configuration in sync.

Clients connect to **services**, not directly to SIDs or file names. This
matters a lot for load balancing, failover, and PDB access.

---

## 3. Listeners and Registration: Who Talks To Whom

The **listener** is a separate process that listens for incoming connections:

- Default host: the database host
- Default port: `1521`
- Default protocol: TCP

When the instance starts, a background process (LREG) periodically:

- Registers itself and its services with:
 - The **default listener** on the local host/port
 - Any additional listeners in:
 - `LOCAL_LISTENER` (listeners on the same host, non-default ports)
 - `REMOTE_LISTENER` (listeners on remote hosts, useful in RAC and HA)

Key points:

- One instance can register with **many** listeners.
- A client must connect to **one** of those listeners that knows about the
 desired service.
- Listeners can live on hosts other than the database host (for security,
 DMZs, connection managers, etc.).

Once a connection is established:

- The listener hands it off to a server process.
- The listener is no longer in the data path for that session.

---

## 4. Naming Methods: How Clients Find Listeners

The user process needs:

- Hostname (where the listener lives)
- Port (which TCP port to talk to)
- Protocol (usually TCP)
- Service name (which database/PDB to hit)

Oracle Net provides several **naming methods** to supply that information.

### 4.1 Easy Connect

The all-in-one connect string:

```text
username/password@host:port/service_name
```

Example:

```text
system/cloud_4U@dbhost.example.com:1521/orclpdb1
```

Pros:

- Simple, no client-side files needed.

Cons:

- Cannot be fully secured via Oracle Net configuration (no SSL config, etc.).
- You lean entirely on the network infrastructure for security.
- Very limited syntax: you cannot easily express complex failover or load
 balancing rules in a single Easy Connect string.

This is why many sites historically used Easy Connect with JDBC thin drivers:

- Apps and DB lived inside the same firewall.
- The network itself was assumed to be "trusted."
- No one had to understand Oracle Net files on the app side.

It works, but the limitations matter:

- You give up richer security and connection policies you get with full
 Oracle Net configuration.

### 4.2 Local Naming (`tnsnames.ora`)

Connection details are stored in a **text file** on the client host:

- Default location: `$ORACLE_HOME/network/admin/tnsnames.ora`
- Or in a directory pointed to by `TNS_ADMIN`

Clients refer to a **net service name**, which maps to a connect descriptor.

Example `tnsnames.ora` entry:

```text
ORCLPDB1 =
 (DESCRIPTION =
 (ADDRESS = (PROTOCOL = TCP)(HOST = dbhost.example.com)(PORT = 1521))
 (CONNECT_DATA =
 (SERVICE_NAME = orclpdb1)
 )
 )
```

Then you connect with:

```text
system/cloud_4U@ORCLPDB1
```

On the host where Oracle client or database software is installed, the default
location for these files is:

- `$ORACLE_HOME/network/admin`

Unless you set `TNS_ADMIN` to point somewhere else.

When editing `tnsnames.ora`:

- Use a **simple** text editor (no hidden formatting characters).
- Be careful with copy/paste; trailing spaces and odd characters can break
 parsing.
- In the demo, we:
 - Copied the `ORCL` entry.
 - Renamed the alias to `RON1`.
 - Connected with:

 ```text
 sqlplus system/oracle_4U@RON1
 ```

 - Oracle resolved `RON1` via `tnsnames.ora` and routed us to the correct
 service.

### 4.3 Directory Naming (LDAP / Active Directory)

Instead of a local `tnsnames.ora` on every client:

- Store net service names in a **directory service** (e.g. Active Directory).
- Clients query the directory to resolve the name into a connect descriptor.

Advantages:

- Centralized management of service names.
- No `tnsnames.ora` deployment headache across fleets of machines.

Which naming methods are enabled and in what order? That is controlled by
`sqlnet.ora`.

---

## 5. `sqlnet.ora`: The Network Rules Engine

The `sqlnet.ora` file defines Oracle Net **behaviour and requirements**:

- Which naming methods are used and in what order:

 ```text
 NAMES.DIRECTORY_PATH = (TNSNAMES, EZCONNECT, LDAP)
 ```

- Encryption, checksumming, and SSL requirements
- Timeouts, dead connection detection
- Tracing and logging options

Location:

- By default: `$ORACLE_HOME/network/admin/sqlnet.ora`
- Or in the directory pointed to by `TNS_ADMIN`

Important note:

- `sqlnet.ora` syntax is **very sensitive** to whitespace and formatting.
- One stray tab or misaligned parenthesis can break name resolution.
- This is why Oracle strongly prefers you use the GUI tools to generate it.

---

## 6. OCI vs Easy Connect: Why Network Files Still Matter

When you:

- Hard-code an Easy Connect string in your app, you talk directly to the listener.
- Use Oracle Net files (`tnsnames.ora`, `sqlnet.ora`), you are using the
 **Oracle Call Interface (OCI)** stack.

Advantages of using Oracle Net/OCI:

- You can configure:
 - SSL/TLS
 - Encryption and checksumming
 - Fallback/timeout policies
- Both **client** and **server** sides can enforce matching requirements.
- DB links and other server-side connections also use these local files.

Easy Connect is fine for labs and simple setups; Oracle Net configuration is
how you ship real systems without waking up at 3 a.m. over a mis-typed host.

---

## 7. Listener Placement and Security

That `(HOST=...)` in a TNS entry:

- Does **not** have to be the database host.
- It is the **listener host**.

Common patterns:

- Listener on the DB host (default, simple)
- Listener on a **separate** host in a DMZ or bastion tier
 - Clients connect to the "front" listener.
 - That listener forwards connections to the database side (or to Connection Manager).

Security reasons for splitting:

- Listeners accept connection requests from *anyone* who can reach them.
- If someone floods or probes the listener, you would rather it be an isolated
 listener box than your actual database server.

If there is **no** listener:

- No network connections, full stop.
- Only local OS-authenticated connections (e.g. `sqlplus / as sysdba` on the host).

---

## 8. Dedicated vs Shared Server Connections

On the database side, each connection can be handled in one of two broad modes.

### 8.1 Dedicated Server

- Each client connection gets its **own** server process and session.
- Session state (user global area) lives in the **PGA** of that process.
- Simple mental model, easiest to tune.

Good for:

- OLTP systems
- Heavy reporting where each session does substantial work

### 8.2 Shared Server

Designed for environments with large numbers of relatively lightweight connections.

Key differences:

- Multiple connections share a smaller pool of **server processes**.
- Session state (user global area) must be **shared**, so it moves to the
 **large pool** in the SGA.
- **Dispatchers** handle incoming requests and route them to available
 shared servers.

Pros:

- Lower per-connection memory footprint in the PGA.
- Potentially support more concurrent connections for similar memory.

Cons:

- Additional complexity and routing overhead.
- Not great for heavy, long-running reports that monopolize shared servers.

Rule of thumb:

- OLTP and mixed workloads often stick with dedicated server.
- Very large web-style environments with many short-lived requests may benefit
 from shared server.

---

## 9. Tools for Managing Oracle Net

Because nothing says "simple networking" like five different tools.

### 9.1 EM Express and EM Cloud Control

- **EM Express**
 - Lightweight web console, per-database.
 - Lets you inspect listeners and net services at a high level.

- **Enterprise Manager Cloud Control**
 - Full-blown enterprise management tool.
 - Can manage:
 - Databases
 - App servers
 - Hosts
 - Network components
 - Includes Net Services administration pages (listener status, services, etc.).

### 9.2 Net Manager (`netmgr`)

GUI utility for editing:

- `listener.ora`
- `tnsnames.ora`
- `sqlnet.ora`

You navigate a tree:

- Profiles -> `sqlnet.ora` settings
- Service Naming -> net service names (`tnsnames.ora`)
- Listeners -> `listener.ora`

Important: after making changes, use **File -> Save Network Configuration**
or your edits will not hit disk.

### 9.3 Net Configuration Assistant (NETCA)

Another GUI wizard, focused on:

- Creating and configuring listeners
- Creating TNS entries
- Naming method configuration (local, directory, etc.)

DBCA actually calls NETCA behind the scenes when you ask it to create a new
listener as part of database creation.

### 9.4 Listener Control (`lsnrctl`)

Command-line interface for the listener:

```bash
lsnrctl status
lsnrctl start
lsnrctl stop
```

Useful for:

- Verifying:
 - Which services are registered
 - Which instances they point to
 - Which addresses the listener is listening on

### 9.5 Manual Editing (When You Are Brave Or Desperate)

All Oracle Net files are plain text:

- `listener.ora`
- `tnsnames.ora`
- `sqlnet.ora`

You *can* edit them by hand, but:

- `sqlnet.ora` is very sensitive to whitespace and formatting.
- A stray tab or extra space in the wrong place can break resolution.
- Recommended practice: use tools (Net Manager, NETCA, EM) to generate/modify,
 and only hand-edit when you really know what you are doing.

---

## 10. TNS_ADMIN and Default Network Locations

Oracle needs to know where to find its network files.

Two main possibilities:

- **Default**: `$ORACLE_HOME/network/admin`
- **Custom**: directory pointed to by `TNS_ADMIN`

If `TNS_ADMIN` is set:

- All Oracle Net files are expected there:
 - `listener.ora`
 - `tnsnames.ora`
 - `sqlnet.ora`

If it is not set:

- Oracle falls back to the `network/admin` subdirectory of the current `ORACLE_HOME`.

This matters when you:

- Run multiple Oracle Homes on one host.
- Want app-specific network configs.
- Run DB links from the database host (server-side `tnsnames.ora`).

---

## 11. Dedicated vs Shared: Choosing What Fits Your Workload

Putting it together:

- Dedicated server:
 - Simple, predictable, more PGA usage per connection.
 - Great for OLTP and reporting-heavy systems.

- Shared server:
 - More complex, uses large pool for session state.
 - Great for thousands of lightweight connections doing small bits of work.

You can also mix modes:

- Use shared server for some services.
- Force specific services or modules to use dedicated connections.

Design the network + connection model **around your workload**, not the other
way around.

---

## 12. Designing Connection Policy: Naming Methods, Protocols, and Compression

Before you write a single `tnsnames.ora` entry in production, you should decide
what kind of connections you actually want.

Questions worth answering *up front*:

- Will clients connect over untrusted networks (Internet, partner networks)?
- Do we require encryption (SSL/TLS) or checksumming?
- Are we OK with Easy Connect only, or do we want full Oracle Net control?
- How will we handle failover and load balancing for HA?

Things you *can* control with Oracle Net:

- **Protocol** and security:
 - Plain TCP
 - TCP + SSL/TLS
 - Checksumming on or off
- **Packet size and compression**:
 - Larger packets and compression can push more data per round-trip.
 - But compression costs CPU, so usually you set a **threshold**: only compress
 when traffic is high enough to justify it.
- **HA behaviour**:
 - You can define load balancing and failover either
 - In the **service** definition in the database (recommended), or
 - In the **TNS connect descriptor** in `tnsnames.ora`.

This is not something you improvise at 3 a.m. in production. Align it with:

- Your DNS and IP plan
- Internal vs external network segments
- Where app servers and databases actually live

---

## 13. Service Names, Domains, and Aliases

Every Oracle service effectively has **two pieces**:

1. The **service name** itself:
 - Could be the instance name (`ORCL`)
 - Could be a PDB name (`ORCLPDB1`)
 - Could be an application service you created (`FINANCE`)
2. The **domain**:
 - For example: `.us.flowers.com`

Together they form the **global service name**:

```text
FINANCE.us.flowers.com
```

On the client side, you rarely expose that full name directly in application
code. Instead, you define a **net service name** (alias) in `tnsnames.ora`:

```text
FINFLOWERS =
 (DESCRIPTION =
 (ADDRESS = (PROTOCOL=TCP)(HOST=listenerbox)(PORT=1521))
 (CONNECT_DATA =
 (SERVICE_NAME = FINANCE.us.flowers.com)
 )
 )
```

Then applications just use:

```text
system/cloud_4U@FINFLOWERS
```

Benefits:

- Small extra layer of indirection: if someone reverse-engineers the alias from
 the app (`FINFLOWERS`) they still do not know the actual global service name.
- You can repoint the alias to a different service or host without touching the app.

Remember:

- `tnsnames.ora` lives on **clients** and also on the **database host** for
 server-side connections like DB links.

---

## 14. Naming Methods in More Detail (Easy, Local, LDAP/Global, External)

We touched on naming methods earlier; here is the slightly deeper dive.

### 14.1 Easy Connect (EZCONNECT)

- Uses a bare connect string:

 ```text
 user/password@host:port/service
 ```

- Enabled by default even if you have no `sqlnet.ora`.
- Great for:
 - Labs
 - Quick tests
 - Scripts where you control both ends

Downside:

- You cannot describe rich connection policies (SSL, compression, failover)
 in Easy Connect alone. You rely heavily on the network perimeter.

### 14.2 Local Naming (TNSNAMES)

We already covered this, but the operational cost is worth repeating:

- The `tnsnames.ora` file must exist on:
 - Every client host that connects
 - Every database host that needs DB links or outgoing connections
- Any change in service or host location means:
 - Updating all relevant copies
 - Or centralizing via config management

Useful and very common, but know what you are signing up for.

### 14.3 LDAP / Directory Naming

Instead of `tnsnames.ora` everywhere:

- Place all connection definitions in an LDAP-compliant directory.
- Put an `ldap.ora` on clients and database hosts that points to that directory.
- All TNS lookups go there.

Advantages:

- Single source of truth for service definitions.
- Updating a connection means **one** LDAP change, no client edits.

When the directory is an **Oracle** directory service and deeply integrated,
you get true **global naming** where:

- Oracle can re-authenticate users against the directory on each connection.
- Roles/privileges can be managed centrally.

### 14.4 External Naming and Security Caveats

External naming is when the directory or OS is not fully Oracle-aware.

Key warning:

- With many third-party directories or pure OS authentication, Oracle treats
 the fact that you reached the database host as the real security barrier.
- Once you are "inside the neighborhood," the **front door may be open**.

The instructor demo showed this difference clearly:

- OS-authenticated connection:

 ```bash
 # On the database host as the oracle user
 sqlplus / as sysdba
 SHOW USER; -- SYS
 ```

- Still OS-authenticated, but with bogus credentials:

 ```bash
 sqlplus dummy/junk as sysdba
 SHOW USER; -- still SYS
 ```

 Over OS authentication, Oracle ignores the supplied username/password and
 trusts the OS user and group membership instead.

- But if you go through Oracle Net (network authentication):

 ```bash
 sqlplus dummy/junk@orcl as sysdba
 ```

 You now get an authentication failure, because the credentials matter.

Takeaways:

- For secure deployments:
 - Use Oracle-aware directory services (Oracle directory, Kerberos) when you
 want centralized auth.
 - Be very cautious with generic external naming where Oracle does not
 re-authenticate.

---

## 15. Load Balancing and Failover

High availability is not just a RAC or Data Guard problem; it is also a
**connectivity** problem.

You can define load balancing and failover:

- At the **service** level (preferred):
 - Use services that span multiple instances/databases.
 - Configure failover and load balancing in the service, and let the database
 and listener advertise it.
- In the **TNS connect descriptor**:
 - Configure multiple addresses and options like:

 ```text
 (LOAD_BALANCE=ON)
 (FAILOVER=ON)
 ```

 - List multiple listener addresses and let the client cycle through them.

This must be planned alongside:

- Which hosts run listeners
- Whether you have a primary/standby setup
- Whether you use RAC with multiple instances

---

## 16. Summary - From "It Works On My Laptop" To Real Connectivity

By now you should be able to:

- Explain how user processes, listeners, server processes, and sessions fit together
- Describe how CDB/PDB services are exposed and used
- Compare Easy Connect vs local/directory naming and when to use each
- Explain what `sqlnet.ora` controls and why TNS_ADMIN matters
- Understand why listener host does not always equal database host
- Contrast dedicated vs shared server connections and when each is appropriate
- Identify the main tools for managing Oracle Net:
 - EM Express / EM Cloud Control
 - Net Manager / NETCA
 - `lsnrctl`

Which means the next time a developer says "the database is down" because they
cannot connect, you can calmly check the listener, the services, the naming
method, and the directory configuration and reply, "No, the database is fine,
but your connect string is lying to you."
