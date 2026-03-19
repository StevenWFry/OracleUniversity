## Lesson 11 - Provisioning and Managing the ODA Platform (where the DB system finally leaves the PowerPoint phase and becomes an actual thing you have to maintain)

Welcome to the DB system deployment chapter, where Oracle Database Appliance stops speaking in broad virtualization theory and starts demanding host names, IP addresses, passwords, images, networks, and the sort of careful sequencing that separates "clean deployment" from "support ticket with a haunted expression."

By the end of this lesson, you should be able to:

- Prepare the bare metal appliance for DB system deployment
- Create a KVM DB system on Oracle Database Appliance
- View and modify an existing DB system
- Create or attach a virtual network for DB system use
- Understand the main `odacli` commands involved in DB system lifecycle management

---

## 1. What Must Exist Before You Create a DB System (because the appliance does not create itself out of optimism)

Before you can deploy a DB system, the bare metal appliance must already be up, configured, and healthy.

At minimum, current Oracle documentation expects:

- Bare metal ODA is already deployed
- The software repository contains the required Oracle Grid Infrastructure and Oracle Database clone files
- The repository also contains the Oracle Database Appliance KVM DB System image
- The target release and supported versions have been checked with `odacli describe-dbsystem-image`
- ASM access control is enabled on bare metal
- File ownership on ASM objects is correct, or you are prepared to run `odacli modify-dbfileattributes` if prompted

This is the practical staging flow:

1. Download the required GI clone, DB clone, and DB system image from My Oracle Support
2. Copy the files to the appliance
3. Update the repository with the required files
4. Confirm what DB system, GI, and database versions are actually supported
5. Only then begin DB system creation

Representative commands:

```bash
odacli update-repository -f /tmp/odacli-dcs-release-ODAVM.zip
odacli describe-dbsystem-image
odacli modify-dbfileattributes
```

One very important current rule:

- The DB system release must match the bare metal ODA release

That said, the Oracle Grid Infrastructure and Oracle Database versions inside the DB system may be earlier supported versions for that DB system image. Oracle expects you to confirm that with `odacli describe-dbsystem-image`, not by guessing and hoping the appliance is in a forgiving mood.

---

## 2. The High-Level DB System Lifecycle (create, inspect, modify, repeat)

The DB system lifecycle on ODA is straightforward on paper:

- Create the DB system
- List and inspect it
- Modify resources or networks as needed
- Start and stop it as part of operations
- Patch it as part of the managed lifecycle
- Delete it when it is no longer needed

The Browser User Interface gives you the clean visual path.
`odacli` gives you the more explicit scripted path.

Either way, the appliance is managing a lot for you:

- The VM itself
- The database infrastructure in the VM
- The DB system metadata
- Its relationship to CPU pools, storage, and networks

This is why Oracle keeps nudging you to use DB systems for Oracle databases instead of dropping a random database into a generic KVM guest and hoping the tooling will still love you.

---

## 3. Information You Should Gather Before Creation (the part where system administrators become your favorite people for ten minutes)

Before you open the Create DB System page, gather the values you will actually need.

### System information

Typical inputs include:

- DB system name
- Domain name
- Region
- Time zone
- System passwords
- Multi-user access choice if you are using passwordless role separation

### Network information

Typical inputs include:

- Host name
- Public IP address
- Subnet mask
- Gateway
- Public network or vnetwork selection

For HA configurations or extra networks, you may also need:

- Per-node public IPs
- VIP addresses
- SCAN name
- SCAN IPs

### User and group information

Oracle also lets you define:

- Operating system users
- Oracle and Grid ownership model
- User and group separation choices

### Optional database creation information

You can also create a database during DB system creation if you already know:

- Database name
- Database version
- Database class or workload type
- Database shape
- Storage option
- Character set and related database settings

This is the difference between a deployment that flows smoothly and one that stalls because somebody forgot to ask networking for the actual IP plan until the form was already open.

---

## 4. Creating a DB System in the BUI (the guided path for people who prefer forms over JSON)

Current Oracle documentation uses this general BUI flow:

1. Log in to the Browser User Interface
2. Open `Appliance`
3. Click `DB Systems`
4. Click `Create DB System`
5. Enter the required values for the DB system
6. Submit the job and monitor it in `Activity`

The form is usually organized around a few major sections.

### System Information

Here you define the overall identity of the DB system, including:

- DB system name
- Domain name
- Region
- Time zone
- Optional multi-user access settings

Current docs also note naming constraints, such as the DB system name not ending with a dash and usually being kept to 15 characters or fewer.

### Network Information

Here you define how the DB system talks to the outside world.

You specify items such as:

- Host name
- IP address
- Subnet mask
- Gateway
- Default or selected virtual network

If you want the DB system to use a vnetwork instead of the default `pubnet`, that vnetwork must already exist.

### User and Group Selection

This is where Oracle lets you define user/group ownership patterns and role separation choices for the DB system environment.

### Optional Database Creation

You can create a database as part of DB system provisioning.

This is useful if you already know the database shape and workload characteristics and do not want to come back later to build the database as a second job.

After creation:

- Use the DB Systems page to list systems
- Click the DB system name to view its details
- Track the create job in `Activity`

Because if anything goes wrong, Oracle would much rather you read the job details than stare at the screen like the BUI has personally betrayed you.

---

## 5. Creating a DB System with `odacli` (for people who enjoy structured files and repeatable deployments)

The CLI path for DB system creation is based on a JSON payload.

Useful commands include:

```bash
odacli describe-dbsystem-image
odacli create-dbsystem -ta
odacli create-dbsystem -t /tmp/dbsystem_template.json
odacli create-dbsystem -p /tmp/example_dbsystem.json
odacli list-dbsystems
odacli describe-dbsystem -n dbsystem_name
```

The logic is:

- Use `describe-dbsystem-image` to verify supported versions
- Generate or review a template if needed
- Populate the JSON with system, network, and optional database values
- Run `create-dbsystem`
- Track the resulting job

This makes the CLI particularly useful for:

- Repeatable builds
- Standardized deployments
- Version-controlled provisioning inputs
- Situations where "I clicked around and I think I filled it in correctly" is not an acceptable operational method

---

## 6. Viewing and Modifying a DB System (because no deployed system stays perfectly sized forever)

Once the DB system exists, ODA lets you review and modify it from either the BUI or the CLI.

### In the BUI

The normal path is:

1. Open `Appliance -> DB Systems`
2. Click the `Actions` menu for the DB system
3. Choose `Modify`
4. Change the supported attributes
5. Submit the modification job

The transcript highlights changing things such as:

- CPU pool
- Memory
- Network

Current Oracle docs expand that story further. Depending on release, supported modifications can include:

- DB system shape
- CPU core allocation
- Memory allocation
- CPU pool attachment or detachment
- NUMA enablement
- Attach or detach vnetworks
- Attach or detach virtual disks

### In the CLI

Representative commands:

```bash
odacli list-dbsystems
odacli describe-dbsystem -n dbsystem_name
odacli modify-dbsystem -n dbsystem_name -m 64G
odacli modify-dbsystem --name dbsystem_name -cp shared_dbs_pool
odacli modify-dbsystem -n dbsystem_name -avn backupvlan --vnetwork-type Backup -ip 192.168.100.21,192.168.100.22 -nm 255.255.255.0 -gw 192.168.100.1
```

Important current-version nuance:

- Starting with ODA release `19.23`, changing the DB system shape no longer changes the database shape inside the DB system automatically
- Starting with ODA release `19.29`, Oracle also supports scaling DB system CPU cores up and down with `odacli modify-dbsystem --core`, and reducing DB system memory with `odacli modify-dbsystem --memory`

So on newer releases, changing the DB system does **not** magically retune every database inside it.

You must adjust the databases deliberately, in the correct order, unless your preferred operating style is "discover memory mismatch during startup and learn from the fire."

---

## 7. Virtual Networks for DB Systems (because a DB system with no network is just an expensive thought experiment)

DB systems can use the default public virtual network, `pubnet`, or additional virtual networks you create for specific purposes.

Oracle documents KVM virtual networks such as:

- `bridged`
- `bridged-vlan`

These vnetworks are useful for:

- Public connectivity
- Backup traffic
- Data Guard traffic
- Segregated application/database traffic

### Current rules that matter

- The vnetwork must exist before you attach it to a DB system
- By default, a DB system uses `pubnet`
- You cannot detach the original public network used by the DB system after creation
- You can attach an additional public network later and detach that additional one if needed
- Every DB system needs its own correct IP, VIP, and SCAN planning where applicable

This is Oracle's way of saying network planning is not a decorative activity.

---

## 8. Creating a Virtual Network (or VLAN-backed vnetwork) for DB System Use

The training material calls this "creating a virtual network or VLAN for the DB system."

That is directionally correct, but the cleaner current phrasing is:

- create the KVM virtual network first
- then attach it to the DB system

### BUI concept

In the BUI, create the virtual network in the virtualization/network area first, then attach it while creating or modifying the DB system.

Typical inputs include:

- Virtual network name
- Bridge name
- Network type
- Interface
- VLAN ID if using a `bridged-vlan`
- IP address
- Netmask
- Gateway

### CLI examples

Representative commands:

```bash
odacli create-vnetwork --name backupvlan --bridge backupvlan --type bridged-vlan --vlan-id 12 --interface btbond1 --ip 192.168.100.10 --gateway 192.168.100.1 --netmask 255.255.255.0
odacli list-vnetworks
odacli describe-vnetwork -n backupvlan
```

Once the vnetwork exists, you can attach it to the DB system:

```bash
odacli modify-dbsystem -n dbsystem1 -avn backupvlan --vnetwork-type Backup -ip 192.168.100.21,192.168.100.22 -nm 255.255.255.0 -gw 192.168.100.1
```

For `database` or `dataguard` network types, Oracle may also require:

- SCAN name
- SCAN IPs
- VIPs

So yes, this is "just networking," but only in the same way heart surgery is "just a procedure."

---

## 9. Key Takeaways (the part to remember before lesson 12 adds more moving pieces)

- A DB system can only be created after the bare metal appliance and repository are correctly prepared
- The DB system image must match the bare metal ODA release
- `odacli describe-dbsystem-image` is the authoritative way to check supported DB system, GI, and database versions
- DB system creation can include system info, network info, user/group settings, and optional database creation
- The BUI is the simplest guided path, while `odacli create-dbsystem` is the repeatable scripted path
- DB systems can be modified after creation, but newer releases separate DB system sizing changes from database sizing changes
- Virtual networks should be created first, then attached to the DB system
- `pubnet` is the default public network, and the original public network cannot simply be detached after the DB system is created

---

## 10. Wrap-Up (the platform is now becoming real infrastructure instead of a diagram)

You now have the practical deployment picture for ODA DB systems: prepare the repository, verify the image, gather the system and network details, build the DB system, inspect it, modify it when reality changes, and treat virtual networks as first-class planning inputs rather than afterthoughts. This is the point in the course where ODA virtualization stops being a nice idea and becomes an actual operating model with names, IPs, shapes, jobs, and very little tolerance for casual improvisation.
