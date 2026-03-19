## Lesson 4 - Deployment Concepts, Configuration, and Documentation (in which the appliance stops being a pile of hardware and starts demanding actual architectural decisions)

Welcome to the part of the course where Oracle Database Appliance quits pretending it is just a tidy rack install and starts asking the more dangerous questions: what are you deploying, where are you storing it, how are you naming it, and did anyone read the documentation before clicking through a wizard like it owed them money?

By the end of this lesson, you should be able to:

- Identify the key Oracle Database Appliance documents you need before deployment
- Describe the major Oracle Database deployment options available on ODA
- Explain what database shapes are and why Oracle wants you to use them
- Distinguish the main ODA data-storage options and where each one fits
- Summarize the most important configuration inputs and planning requirements before deployment

---

## 1. The Documents That Actually Matter (because "I thought the wizard would explain it" is not a deployment plan)

If you are working with ODA, there are a few document sources you should treat as primary references:

- Oracle Database Appliance documentation on `docs.oracle.com`
- Oracle Database Appliance release notes for the currently targeted software release
- The Oracle Database Appliance Information Center in My Oracle Support

Why these matter:

- The main documentation explains the supported workflows, options, and procedures
- The release notes tell you what the current release actually supports
- The Information Center in My Oracle Support helps you navigate known issues, patches, and Oracle's favorite category of surprise: "important notes you were expected to know already"

Practical rule:

- Use the deployment and user guide for process
- Use the release notes for version support and current caveats
- Use My Oracle Support when reality and documentation stop agreeing

Also, one important Oracleism deserves to be stated plainly:

- Do not use `DBCA` to create databases on ODA
- Use Oracle Appliance Manager tooling instead so the deployment stays supported and aligned with ODA best practices

---

## 2. Database Deployment Options (three ways to decide how much cluster behavior you want in your life)

ODA centers on three core database deployment types:

- Single-instance
- Oracle RAC One Node
- Oracle RAC

These show up in tooling as deployment types such as `SI`, `RACOne`, and `RAC`.

**Single-instance**

- One database instance runs on one node
- Operationally simpler than clustered options
- By itself, this is the least cluster-heavy deployment model

Important nuance:

- Historically, single-instance meant no automated cluster failover
- In newer ODA releases, Oracle also documents optional Standard Edition High Availability and Enterprise Edition High Availability features for certain 19c release levels on HA hardware

So the safe mental model is:

- Classic single-instance is the simplest option
- But on current ODA releases, single-instance does not automatically mean "zero HA features"

**Oracle RAC One Node**

- Oracle RAC software is installed on both nodes
- One node is the active home node at a time
- The database can relocate or fail over within the cluster

This is the "I want some cluster behavior without going fully double-everything" option.

**Oracle RAC**

- Oracle RAC software and RAC database homes are installed across both servers
- The clustered database can run as a true RAC deployment across the HA platform

This is the full cluster option, with the corresponding increase in capability, moving parts, and opportunities for very educational outages.

Version note:

- `19c` remains the broad baseline across ODA releases
- `23ai` support is release-dependent and is explicitly called out in current ODA release notes for DB systems, so always check the current support matrix before locking your deployment plan

---

## 3. Database Shapes (Oracle's way of saying "please stop hand-tuning everything from scratch")

Oracle strongly recommends using ODA database templates, also called shapes.

Shapes are preconfigured templates that encode Oracle's opinionated best practices for ODA.

Why shapes matter:

- They are built specifically for Oracle Database Appliance
- They size memory and CPU expectations in a repeatable way
- They reduce the amount of improvisation required during database creation
- They help align the database with workload-specific defaults

Workload classes typically include:

- `OLTP`
- `DSS`
- `IMDB` or in-memory

Shape behavior to remember:

- Shapes are designed around specific core counts
- Different workload classes have different sizing templates
- You can run multiple shapes at the same time on the same appliance

For example:

- One larger shape such as `odb16`
- Multiple smaller shapes such as several `odb4` databases

That lets you mix heavyweight and lighter-weight databases on the same ODA without treating every database like it deserves identical memory, CPU, and process settings.

Important modern nuance:

- The exact list of supported shapes is not one universal magical number
- It varies by hardware model, workload class, database edition, and deployment type
- Standard Edition deployments are more constrained and generally align with `OLTP`-style templates

So the takeaway is not "memorize every table." The takeaway is "use ODA shapes unless you have a very good reason not to, and know which workload class you are actually deploying."

---

## 4. Data Storage Options (where the appliance politely asks whether you want raw ASM muscle or ACFS flexibility)

ODA supports two primary storage approaches for database deployments:

- Oracle ASM
- Oracle ACFS

**Oracle ASM**

- Oracle Automatic Storage Management is the default storage choice in many ODA workflows
- It handles striping and mirroring across disks
- It protects against failures by spreading and mirroring data across the storage layout
- It is the most direct fit for classic Oracle-managed database storage on the appliance

Think of ASM as the storage engine that wants less debate and more discipline.

**Oracle ACFS**

- Oracle ASM Cluster File System (`ACFS`) is a portable, high-performance cluster file system
- It is created on top of Oracle ASM storage
- It is useful when you want file-system semantics in addition to ASM-backed storage

On ODA, ACFS is commonly used for:

- Shared repositories
- Database-related file systems
- Oracle homes and related managed storage in newer releases
- General-purpose shared storage use cases

ODA storage administration is integrated into the platform, which helps with:

- Database files
- Oracle Grid Infrastructure components
- Oracle Database homes
- Operational files and tools
- Mirroring and load distribution across disks

In short:

- ASM is the core storage foundation
- ACFS adds file-system flexibility on top of that foundation

---

## 5. Configuration Inputs You Need Before Deployment (the checklist that decides whether deployment is smooth or absurd)

Before deployment, Oracle expects you to gather a non-trivial pile of configuration information.

Use the checklists in the deployment and user guide. They exist for a reason, and that reason is preventing the kind of mistakes people only make once if they are lucky.

Core information includes:

- Hardware model
- Deployment mode: bare metal or virtualized platform
- Region
- Time zone
- Host or system name
- Domain name
- Root or master password
- Network details
- Optional initial database details

For clustered or HA planning, networking matters a lot:

- Multi-node deployments require public host addresses for each node
- Virtual IP addresses are required for each node
- SCAN-related addressing must be planned if you are using clustered access naming
- Oracle's classic HA checklist expects at least six public addresses on the same subnet for the clustered node and SCAN layout

That is not networking trivia. It is the difference between "cluster" and "angry collection of half-configured interfaces."

---

## 6. System Naming and Derived Defaults (one name becomes many names, because Oracle loves templates)

During deployment, you choose the appliance system name.

That system name becomes the default root for several other generated names, such as:

- Node public names
- Node VIP names
- SCAN name
- Oracle ILOM names

Practical guidance:

- Keep the name RFC-friendly
- Avoid underscores
- Keep it short, clean, and lower-case where possible

This is not just cosmetic. A sloppy system name tends to breed sloppy derived names, and then everybody gets to read those names in logs for the next several years.

---

## 7. Users, Groups, and OS Role Separation (the part where you decide how much separation of duties you actually believe in)

ODA lets you choose how to configure operating-system users, groups, and role separation.

Oracle supports several patterns, including:

- Two users with six groups
- Single user with six groups
- Single user with two groups

The default Oracle-style split is:

- `grid` for Oracle Grid Infrastructure ownership
- `oracle` for Oracle Database ownership

With separate groups such as:

- `oinstall`
- `dba`
- `dbaoper`
- `asmadmin`
- `asmoper`
- `asmdba`

This gives you the option to keep Grid Infrastructure and database ownership separated instead of merging everything into one giant "please do not log in as this unless necessary" user.

If you want OS role separation, decide that early. It is much easier to configure cleanly at deployment than to explain later why everything ended up owned by one account with a suspicious amount of authority.

---

## 8. Optional ASR and Other Deployment-Time Decisions (because Oracle always has one more form for you)

Oracle Auto Service Request (`ASR`) is an optional deployment-time feature.

You may need information such as:

- Proxy server name
- Proxy port
- Proxy user and password
- My Oracle Support user credentials for internal ASR
- External ASR Manager details for external ASR

ODA supports internal and external ASR approaches, and the choice affects how fault events are reported and where the ASR manager lives.

Other deployment-time decisions may also include:

- NTP settings
- Disk-group redundancy choices
- Initial database creation options
- Language and territory preferences
- Storage layout choices

This is the administrative equivalent of mise en place. Get it ready before deployment, or spend the next hour pretending the wizard is the problem.

---

## 9. Key Takeaways (the architecture decisions before the architecture decisions)

- The key ODA references are the main Oracle docs, the release notes, and the My Oracle Support Information Center
- ODA database deployment options center on `SI`, `RACOne`, and `RAC`
- Single-instance is the simplest path, but modern ODA releases also add HA options for certain single-instance deployments
- Database shapes are ODA-specific templates that encode workload-focused best practices
- The main workload classes are `OLTP`, `DSS`, and `IMDB`
- ODA storage centers on Oracle `ASM` and Oracle `ACFS`, with ACFS layered on top of ASM for shared file-system use cases
- Deployment planning requires much more than a host name: network, storage, naming, users, groups, passwords, and optional ASR details all matter
- For HA deployments, network planning includes clustered public addresses, VIPs, and SCAN-related naming and addressing

---

## 10. Wrap-Up (the paperwork phase is over, and now the real provisioning decisions begin)

You now know where the important ODA references live, what the main database deployment choices are, how shapes and storage options fit into the design, and what configuration data has to be gathered before deployment. Next comes the part where those decisions stop being theoretical and become actual deployed infrastructure, which is where architecture finally meets consequences.
