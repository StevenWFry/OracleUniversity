## Lesson 12 - Managing Virtual Machines and Compute Resources (where the appliance becomes an application platform and politely reminds you that VMs also need storage, networks, and adult supervision)

Welcome to the compute-instance chapter, where Oracle Database Appliance graduates from "database appliance" to "solution in a box" and starts hosting application tiers alongside managed DB systems, which is efficient, powerful, and absolutely capable of becoming a mess if you skip the planning.

By the end of this lesson, you should be able to:

- Explain the difference between a managed DB system and a compute instance
- Prepare the resources required to deploy a compute instance
- Create VM CPU pools, VM storage, virtual disks, and virtual networks
- Create and modify a compute instance from the BUI or `odacli`
- Understand how compute instances fit into ODA virtualization and HA operations

---

## 1. DB System Versus Compute Instance (because Oracle very much wants you to know these are not the same animal)

In a virtualized ODA deployment:

- The bare metal host runs Oracle Linux plus Oracle Appliance Manager
- Managed Oracle databases run in `DB systems`
- Application workloads run in `compute instances`, also called application VMs

The difference is mostly about who is responsible for what.

### DB system

A DB system is the deeply managed Oracle database VM.

ODA manages:

- VM lifecycle
- Grid Infrastructure
- Oracle Database software
- Database lifecycle operations
- Patching and related platform integration

### Compute instance

A compute instance is the user-managed application VM.

ODA manages:

- The VM lifecycle
- Placement, startup, stop, and failover behavior
- Its relationship to CPU pools, storage, and networks

You manage:

- The guest operating system
- Middleware
- Applications
- Application patching
- Everything inside that guest once it exists

This is why Oracle keeps saying, in increasingly clear language:

- run databases in `DB systems`
- run applications in `compute instances`

Do not shove an Oracle database into a generic application VM and then act offended when the ODA tooling refuses to love that decision.

---

## 2. Why Compute Instances Matter (or, how to turn an ODA into "app plus database in one box")

Compute instances let ODA host application workloads next to managed database workloads.

That gives you a few practical advantages:

- Better total system utilization
- Fewer physical servers
- Less rack space, power, and cooling usage
- Better workload isolation than piling everything into one operating system
- A cleaner "solution in a box" pattern for packaged applications

Oracle and partners have long used this pattern for deployments that amount to:

- JD Edwards in a box
- E-Business Suite in a box
- WebLogic in a box
- Third-party application plus Oracle database in one appliance

This is one of the more sensible uses of virtualization: the database gets its managed lane, the application gets its own guest OS, and the hardware gets used more like an engineered platform instead of an expensive paperweight with fans.

---

## 3. What a Compute Instance Actually Is (an application KVM with ODA guardrails around it)

A compute instance is an application KVM guest running on the ODA bare metal host.

Current Oracle guidance treats it as:

- A fully supported application VM model on ODA
- Managed through the BUI and `odacli`
- Capable of auto-start and failover behavior on HA systems
- Eligible for fast cloning and efficient storage usage when designed correctly

But there is an important boundary:

- ODA manages the VM as a platform object
- You manage the application stack inside the VM

So the compute instance is not "managed" in the same way a DB system is managed.

It is more like ODA saying:

"I will create the guest, wire it up, restart it, fail it over, and keep its infrastructure sane. What you install in there afterward is your business, your patching plan, and your future emotional burden."

---

## 4. High-Level Deployment Sequence for a Compute Instance (because the VM does not spring fully formed from the UI)

Current Oracle documentation breaks application VM deployment into a practical sequence:

1. Deploy and validate the bare metal ODA
2. Create a VM CPU pool if you want dedicated CPU isolation
3. Create VM storage
4. Create a virtual network if you need more than the defaults
5. Create any additional virtual disks you want attached
6. Create the compute instance using an operating system ISO image
7. Complete the guest OS installation
8. Configure guest networking and install the application

Important current notes:

- VM CPU pool is optional
- VM storage is mandatory
- Additional virtual disks are optional
- Additional vnetworks are optional, but `pubnet` matters if the VM must reach the DB system on the public side
- For Windows guests, Oracle documents using a VirtIO driver ISO as an external source

This is the part where the transcript's "create the VM" simplicity turns out to mean "create several prerequisite objects first, then the VM, then the operating system, then the application," which is much less catchy but far more honest.

---

## 5. CPU Pools for Compute Instances (how ODA keeps VMs from eating each other's lunch)

VM CPU pools are the main way to isolate compute-instance CPU consumption.

Oracle documents VM CPU pools with these rules:

- They are optional but useful
- Resources in CPU pools cannot overlap
- VM CPU pools can be attached to one or more VMs
- The core count must be an even number
- You can optionally pin to a socket for special licensing or placement needs

Representative command:

```bash
odacli create-cpupool -n apppool1 -c 2 -vm
```

Useful related commands:

```bash
odacli list-cpupools
odacli describe-cpupool -n apppool1
odacli modify-cpupool -n apppool1 ...
odacli delete-cpupool -n apppool1
```

This is one of the cleaner ODA ideas:

- DB systems get DB-system CPU pools
- compute instances get VM CPU pools
- everyone stays in their lane

Which is more than can be said for many data centers.

---

## 6. VM Storage and Virtual Disks (because a VM without storage is just a strangely ambitious thought)

### VM storage

VM storage is mandatory for compute instances.

It is the storage container Oracle uses for VM resources such as:

- VM files
- snapshots or clone structures
- base storage for guest disks

Representative command:

```bash
odacli create-vmstorage -n vmstor1 -s 100G -dg DATA -r MIRROR
```

Useful related commands:

```bash
odacli list-vmstorages
odacli describe-vmstorage -n vmstor1
odacli modify-vmstorage -n vmstor1 ...
odacli delete-vmstorage -n vmstor1
```

### Virtual disks

When you create a compute instance, ODA creates the VM operating-system disk for that guest.

Additional virtual disks are optional, but useful when you want:

- Separate application data storage
- Better storage layout control
- Shared extra disks for specific designs

Representative command:

```bash
odacli create-vdisk -n appdisk1 -vms vmstor1 -s 49G
```

Useful related commands:

```bash
odacli list-vdisks
odacli describe-vdisk -n appdisk1
odacli modify-vdisk -n appdisk1 ...
odacli clone-vdisk -n appdisk1 ...
odacli delete-vdisk -n appdisk1
```

The transcript frames this as "create VM storage and virtual disk."
That is correct, but the real operational distinction is:

- VM storage is the container
- virtual disk is the extra block device
- the guest OS disk still gets created as part of the VM itself

If you need a disk to be shareable between multiple VMs, Oracle supports that too with the optional `-sh` flag. Most ordinary application-VM examples do not need it.

If you blur those together, the UI starts feeling much more magical than it actually is, and magical infrastructure is usually just misunderstood infrastructure with better marketing.

---

## 7. Virtual Networks for Compute Instances (including the special politics of `pubnet`)

Compute instances can use:

- the default public virtual network `pubnet`
- additional KVM virtual networks you create

Oracle documents virtual network types such as:

- `bridged`
- `bridged-vlan`

Representative command:

```bash
odacli create-vnetwork --name appvlan --bridge appvlan --type bridged-vlan --vlan-id 120 --interface btbond1 --ip 192.168.120.10 --gateway 192.168.120.1 --netmask 255.255.255.0
```

Useful related commands:

```bash
odacli list-vnetworks
odacli describe-vnetwork -n appvlan
odacli modify-vnetwork -n appvlan ...
odacli delete-vnetwork -n appvlan
```

Important current rule from Oracle's solution guidance:

- Any VM that must access the DB system on the public network must be attached to `pubnet`

Additional current-network nuance:

- No extra plain bridged network is allowed on the already-used public network interface other than the default `pubnet` bridge
- Additional `bridged-vlan` networks can be created on available public interfaces, including the interface that already carries the public network
- If you assign bridge IPs on HA, the first IP goes to node0 and the second to node1

So yes, you can create extra virtual networks.
No, that does not mean every interface is an all-you-can-eat buffet of networking creativity.

---

## 8. Creating a Compute Instance in the BUI (the guided path for building an application VM)

The Browser User Interface flow is broadly:

1. Open `Appliance`
2. Open `VM Instances` or the compute-instances area, depending on release wording
3. Create or verify the supporting objects:
   - CPU Pool, if needed
   - VM Storage
   - Virtual Disk, if needed
   - Virtual Network, if needed
4. Click `Create VM`
5. Enter the compute-instance details
6. Submit the job and monitor it in `Activity`

Typical inputs for the compute instance include:

- VM name
- VM storage name
- Source installation path to the guest OS ISO
- vCPU count
- memory size
- OS disk size
- CPU pool, if using one
- virtual disk, if using one
- virtual networks, including `pubnet` if needed

For Windows guests, Oracle also documents an external source ISO for VirtIO drivers, because apparently installing one operating system image was not enough adventure for the day.

After creation:

- Use `odacli describe-vm -n vm_name` to find the VNC display port if needed
- Complete the guest OS install
- Configure the guest network settings inside the VM
- Install the application stack

One useful networking nuance from Oracle's KVM guidance:

- the VM gets a default host-only interface on `virbr0`
- that interface is for convenient host-to-guest communication
- it is not the interface you rely on for normal external application traffic
- attach `pubnet` or another vnetwork for real network connectivity

That last part matters.

ODA creates the compute instance.
It does not finish the guest OS and application setup for you, because that would apparently be too generous.

---

## 9. Creating a Compute Instance with `odacli` (for people who like explicit commands more than clicking through tabs)

Representative Linux example:

```bash
odacli create-vm -n appvm1 -vc 2 -m 8G -vms vmstor1 -s 49G -cp apppool1 -vd appdisk1 -vn pubnet,appvlan -src /u01/software/ol9.iso
```

Representative Windows-style example with VirtIO external source:

```bash
odacli create-vm -n winvm1 -vc 2 -m 8G -vms vmstor1 -s 49G -cp apppool1 -vn pubnet -src /u01/software/win2022.iso -esrc /u01/software/winvirtio.iso
```

Useful lifecycle commands:

```bash
odacli list-vms
odacli describe-vm -n appvm1
odacli start-vm -n appvm1
odacli stop-vm -n appvm1
odacli clone-vm -n appvm1 ...
odacli migrate-vm -n appvm1 -to node2
odacli delete-vm -n appvm1
```

This is the CLI equivalent of what the BUI is doing for you, just with fewer buttons and a much higher chance that you will remember exactly which options you forgot.

---

## 10. Modifying a Compute Instance (because the first sizing choice is rarely the last one)

Once a compute instance exists, ODA lets you modify it through the BUI or `odacli`.

Common changes include:

- CPU count
- memory
- CPU pool association
- attached vdisks
- attached virtual networks
- autostart behavior
- preferred node
- NUMA enablement

Representative command examples:

```bash
odacli modify-vm -n appvm1 -vc 6 -mm 6G --live --config
odacli modify-vm -n appvm1 -avn appvlan
odacli modify-vm -n appvm1 -avd appdisk1
odacli modify-vm --name appvm1 --enable-numa
```

Two practical notes matter here:

- `--live` applies the change to the running VM immediately
- `--config` makes the change persistent for future boots

If you do not understand that distinction, you can produce one of infrastructure's classic party tricks:

- "it worked right now"
- "it vanished after reboot"

Which is rarely the outcome anyone was hoping for.

---

## 11. Compute Instances on HA Systems (because application VMs also enjoy not dying in place)

On ODA HA systems, Oracle documents automatic restart and failover support for compute instances.

That means a compute instance can be part of a broader HA design where:

- the VM is started on a preferred node
- it can auto-start after host events
- it can fail over to the surviving node when appropriate

Oracle also documents VM migration and cloning support, which helps with:

- maintenance
- testing
- template-based deployment
- reducing rebuild effort

This is part of the real appeal of application VMs on ODA:

- you get isolation
- you get consolidation
- you still get platform-managed HA behavior

Which is a much better arrangement than manually rebuilding an application server at 3 AM because "it was only a small VM."

---

## 12. Key Takeaways (the bit to remember before the final module starts putting databases inside DB systems in more detail)

- `DB systems` and `compute instances` are different by design: DB systems are deeply managed database VMs, while compute instances are user-managed application VMs
- A compute instance is useful for the application tier in a "solution in a box" design
- VM CPU pools are optional but valuable for isolation and controlled resource allocation
- VM storage is mandatory; additional virtual disks and virtual networks are optional
- Any application VM that needs to reach the DB system on the public network must use `pubnet`
- Creating the VM is only part of the job; you still must install and configure the guest OS and application
- `odacli` supports the full compute-instance lifecycle, including create, modify, start, stop, clone, migrate, and delete

---

## 13. Wrap-Up (the application tier has entered the building)

You now have the compute-instance side of ODA virtualization: how it differs from DB systems, which supporting resources must exist first, how to create and attach CPU, storage, disk, and network resources, and how to build and modify the VM itself. This is the point where ODA stops being just a database platform and starts looking suspiciously like a compact application stack factory, provided you respect the sequencing and do not confuse "Oracle manages the VM" with "Oracle manages everything you put inside it."
