## Lesson 9 - Administration, Networking, Storage, and Diagnostics (where the appliance politely reveals that "administration" is really a long list of things you must watch before they become expensive)

Welcome to the bare-metal administration chapter, where Oracle Database Appliance stops being the exciting new engineered box in the rack and becomes what it really is: a living system full of networks, disks, alerts, jobs, hardware sensors, and consequences.

By the end of this lesson, you should be able to:

- Describe the main bare-metal administration process on Oracle Database Appliance
- Distinguish between Browser User Interface, `odacli`, and `odaadmcli` administration tasks
- Explain how to manage networks, VLANs, and storage on bare metal ODA
- Identify the main hardware and storage diagnostic commands
- Explain how Oracle Auto Service Request (`ASR`) fits into monitoring and support

---

## 1. The Bare-Metal Administration Process (less wizard, more responsible adult with a checklist)

Bare metal administration on ODA is basically the ongoing process of checking that the appliance is:

- Configured the way you think it is
- Healthy the way Oracle expects it to be
- Connected to the right networks
- Using storage correctly
- Actually capable of calling for help when hardware starts misbehaving

You can do most of this from either:

- The Browser User Interface (`BUI`)
- `odacli`
- `odaadmcli`

The high-level split is simple:

- Use the `BUI` when you want guided configuration, visual status pages, and job tracking
- Use `odacli` for appliance lifecycle and configuration tasks
- Use `odaadmcli` for low-level hardware, storage, and classic VLAN administration

Representative status commands include:

```bash
odacli describe-system -d
odacli list-networks
odaadmcli show server
odaadmcli show storage
```

One version wrinkle worth knowing:

- Older ODA material often says `odacli describe-appliance`
- Current Oracle documentation emphasizes `odacli describe-system`

So if you see both in training material, that is not Oracle being mysterious. That is just release drift doing what release drift does.

---

## 2. What the BUI Gives You (for people who enjoy seeing the wreckage before opening a shell)

The Browser User Interface is the main administrative dashboard for bare metal ODA.

In current Oracle documentation, the BUI lets you view and manage:

- Appliance information
- System information
- Disk group usage and storage utilization
- Installed hardware and software component inventory
- RPM and RPM drift information
- Networks
- Oracle ASR configuration
- Jobs and activity history

The usual navigation pattern is:

- `Appliance` for appliance-wide status and inventory
- `Network` for network creation, updates, and review
- `Oracle ASR` for call-home configuration
- `Activity` for job status and troubleshooting

So yes, the BUI is doing a lot of work here. It is effectively the control room for people who prefer menus over memorizing twenty commands before breakfast.

---

## 3. `odacli` Versus `odaadmcli` (because Oracle gave you two toolboxes and expected you not to mix them up)

Oracle splits command-line administration into two major tools.

**`odacli`**

Use `odacli` for lifecycle and appliance-level administration, such as:

- Describing the system
- Listing and describing networks
- Creating, modifying, and deleting network objects
- Managing jobs
- Configuring or modifying Oracle ASR
- Validating storage topology

**`odaadmcli`**

Use `odaadmcli` for hardware and storage administration, such as:

- Showing server, processor, memory, power, cooling, and network details
- Showing disks, disk groups, controllers, and enclosures
- Expanding storage
- Running storage diagnostics
- Working with classic VLAN commands in release-specific workflows

This is the practical rule:

- If the task smells like lifecycle, configuration, or a managed ODA object, think `odacli`
- If the task smells like hardware, sensors, disks, or "why is this shelf being weird," think `odaadmcli`

---

## 4. Core Bare-Metal Checks (the administrative equivalent of taking the patient's temperature before announcing a diagnosis)

When you are checking the state of an ODA appliance, start with a few basic questions:

- Is the system up and recognized correctly?
- Are the nodes healthy?
- Is the hardware environment clean?
- Are the disks, controllers, and disk groups healthy?
- Are the networks present and configured the way you intended?
- Is ASR configured and able to report faults?

Useful commands for that first-pass health check include:

```bash
odacli describe-system -d
odacli list-networks
odaadmcli show server
odaadmcli show env_hw
odaadmcli show network
odaadmcli show storage
odaadmcli show disk
odaadmcli show diskgroup
odaadmcli show controller
```

If you are troubleshooting cabling or storage layout on HA systems, add:

```bash
odacli validate-storagetopology
```

That command is Oracle's way of asking, with increasing firmness, whether the cables are actually where you think they are.

---

## 5. Managing Networks and VLANs (where one wrong IP can turn "administration" into "field archaeology")

ODA supports network and VLAN management for workloads such as:

- Database traffic
- Backup traffic
- Management traffic
- Data Guard traffic
- Other customer-defined uses

### BUI network management

The current BUI flow is straightforward:

1. Open the `Appliance` or `Network` area in the BUI
2. Review existing physical and virtual networks
3. Create, update, or delete network entries as needed
4. Monitor the resulting job in `Activity`

For HA systems, the BUI can help you create a network for both nodes at the same time.

### CLI network management

Current Oracle docs use `odacli` for managed network objects:

```bash
odacli list-networks
odacli describe-network -i NETWORK_ID
odacli create-network ...
odacli delete-network -i NETWORK_ID
```

You will also see classic VLAN administration through `odaadmcli` in some Oracle materials and release-specific command references:

```bash
odaadmcli show vlan
odaadmcli create vlan ...
odaadmcli delete vlan ...
```

That is not a contradiction so much as Oracle documenting both the higher-level network model and the lower-level VLAN plumbing.

### Important network constraints

Current Oracle documentation is stricter than the friendly tone of many training slides, so keep these rules in your head:

- The public VLAN is established during `odacli configure-firstnet`
- Only one public VLAN is supported
- On current docs, VLAN planning should be done before deployment
- DHCP is not supported for supplying the appliance with its network configuration
- On HA systems, Node 0 and Node 1 cannot share the same IP addresses
- On HA additional networks, you may also need SCAN and VIP details
- Public and private interface choices are not something you casually swap later without consequences

This is one of those areas where ODA is not being difficult for sport. It is trying to stop you from redesigning the airplane while it is already in the air.

---

## 6. Managing Storage (because databases become surprisingly emotional when storage goes sideways)

Storage administration on ODA includes:

- Viewing disks, disk groups, enclosures, and controllers
- Checking storage health
- Validating the storage topology
- Expanding storage on supported HA platforms
- Collecting diagnostics for failed or suspicious disks

Useful commands include:

```bash
odaadmcli show storage
odaadmcli show disk
odaadmcli show diskgroup
odaadmcli show controller
odacli list-dgdisks
odacli validate-storagetopology
```

For expansion-shelf workflows on HA systems, Oracle documents additional checks such as:

```bash
odaadmcli show enclosure
odaadmcli expand storage ...
odaadmcli show validation storage errors
odaadmcli show validation storage failures
```

The basic rhythm is:

1. Validate the current storage topology
2. Confirm the new shelf or disks are visible and healthy
3. Expand storage using the supported ODA workflow
4. Recheck disk state, ASM membership, and firmware/component status

This is not the place for improvisation. Improvisation in storage administration is how otherwise calm people end up talking to hardware with the intensity of a Victorian exorcist.

---

## 7. Diagnostics and Monitoring (the art of noticing trouble before users do)

ODA includes both built-in hardware visibility and external monitoring options.

### Hardware and environmental checks

Use `odaadmcli` for fast hardware inspection:

```bash
odaadmcli show env_hw
odaadmcli show power
odaadmcli show cooling
odaadmcli show processor
odaadmcli show memory
odaadmcli show network
odaadmcli show server
```

These commands help surface problems with temperature, fans, power, processors, memory, and overall server condition before the appliance decides to make the problem much more memorable.

### Storage diagnostics

For storage-specific investigation, Oracle documents:

```bash
odaadmcli stordiag pd_00
```

Use `stordiag` when you want detailed diagnostic information for a disk or NVMe device.

You can also use:

- `odaadmcli power disk` for NVMe power operations in supported cases
- `odaadmcli show validation storage errors`
- `odaadmcli show validation storage failures`

### Centralized monitoring

If you want a broader data-center view, Oracle still publishes release notes for the Oracle Database Appliance Enterprise Manager plug-in.

That means ODA monitoring does not have to live entirely inside one appliance at a time. You can use Enterprise Manager Cloud Control as the bigger control tower if your environment is large enough to justify one more enterprise console in your life.

---

## 8. Oracle Auto Service Request (`ASR`) (Oracle's call-home feature, or the appliance tattling to support on purpose)

Oracle ASR is the automated support and call-home mechanism for hardware faults.

The basic idea is:

1. A hardware or monitored fault is detected
2. Diagnostic information is collected
3. Oracle Support is notified automatically
4. Service activity can begin faster, including parts dispatch where appropriate

### ASR configuration types

Current Oracle documentation distinguishes between:

- **Internal ASR**: Oracle ASR Manager runs on the same appliance
- **External ASR**: Oracle ASR Manager runs on another server or appliance

The tradeoff is simple:

- Internal ASR is simpler because everything lives together
- External ASR is more resilient if the local appliance is the thing having a terrible day

### How to manage ASR

You can configure ASR:

- During deployment
- From the BUI after deployment
- From the CLI after deployment

Key `odacli` commands include:

```bash
odacli configure-asr
odacli describe-asr
odacli modify-asr
odacli test-asr
odacli delete-asr
odacli export-asrconfig
```

### What ASR needs

Current Oracle docs require a few practical things:

- The hardware must be associated with a Support Identifier (`SI`) in My Oracle Support
- Internal ASR needs My Oracle Support credentials tied to the registered system
- If a proxy is required, proxy settings must be provided
- External ASR uses configuration exported from the remote ASR manager appliance

### Important current-version nuance

Older training material often talks about SNMP settings as if that is the whole ASR story.

Current Oracle guidance is more specific:

- Oracle ILOM and hardware monitoring can still generate alerts such as SNMP traps, IPMI PET events, and remote syslog events
- Starting with ODA release `19.21`, Oracle ASR itself is configured around HTTPS/XML payload behavior rather than the older pure-SNMP mental model

So if the training wording sounds older than the product, trust the current release docs over nostalgia.

### ASR logs and behavior

Oracle documents ASR logs in:

```text
/var/opt/asrmanager/log/
```

And there is one operational gotcha worth remembering:

- You can modify an internal ASR configuration with `odacli modify-asr`
- You cannot use that same shortcut to convert external ASR in place; external changes often require delete-and-reconfigure behavior

Because of course they do.

---

## 9. Key Takeaways (the part to remember when the appliance starts blinking at you)

- Bare-metal ODA administration lives across the BUI, `odacli`, and `odaadmcli`
- The BUI is the clearest place to review appliance inventory, networks, ASR, and job activity
- `odacli` handles appliance lifecycle and managed objects such as networks and ASR
- `odaadmcli` handles low-level hardware, storage, diagnostics, and release-specific VLAN tooling
- Network and VLAN work has real constraints: only one public VLAN, no DHCP provisioning, and careful HA address planning
- Storage checks should begin with visibility and validation, not wishful thinking
- Diagnostics start with `show env_hw`, `show server`, `show storage`, `show disk`, and `stordiag`
- Oracle ASR is the appliance's automated call-home path, and current releases distinguish clearly between internal and external ASR

---

## 10. Wrap-Up (administration is mostly prevention with better tools)

You now have the bare-metal administration picture: how to inspect the appliance, where to manage networks and storage, which commands to reach for when hardware starts behaving suspiciously, and how ASR turns hardware failure into an automated support event instead of a manually assembled panic ritual. This is the part of ODA where the appliance stops being a deployment project and becomes an operational relationship, which is a much more elegant way of saying "you now have to keep it healthy on purpose."
