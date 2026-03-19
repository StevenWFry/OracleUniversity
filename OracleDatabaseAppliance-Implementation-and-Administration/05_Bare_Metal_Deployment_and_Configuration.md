## Lesson 5 - Bare Metal Deployment and Configuration (in which the appliance finally becomes a real system instead of an expensive promise with LEDs)

Welcome to the bare metal deployment chapter, where ODA drops the virtualization layers, rolls up its sleeves, and asks you to provision the platform directly on the hardware. This is the part where Oracle says, "we made deployment easier," and then still hands you clones, repositories, passwords, patch levels, and enough moving parts to keep things interesting.

By the end of this lesson, you should be able to:

- Define what a bare metal Oracle Database Appliance deployment is
- Distinguish the single-node and multi-node bare metal deployment patterns
- Identify the base software and clone files used in bare metal deployment
- Summarize the predeployment tasks required before creating the appliance
- Explain how the Browser User Interface and `odacli` JSON workflow are used to deploy bare metal ODA
- Recognize the key security and password settings that matter during and immediately after deployment

---

## 1. What Bare Metal Means on ODA (no hypervisor, no abstraction, no excuses)

A bare metal ODA deployment is a non-virtualized Oracle Database Appliance configuration.

That means:

- The appliance is deployed directly on the hardware
- Oracle Linux, Oracle Grid Infrastructure, and Oracle Database components run without a VM layer in between
- The platform behaves as a direct appliance deployment rather than a DB system or application-VM environment

This is the most literal version of ODA: just the hardware, the Oracle stack, and your decisions for how much future trouble you would like to invite.

---

## 2. Single-Node and Multi-Node Bare Metal Options (small box versus clustered box, same stress different topology)

Bare metal deployment maps to the hardware model you are working with.

**Single-node bare metal**

- `X11-S` and `X11-L` are the single-node hardware models
- A bare metal deployment on these systems creates a direct, non-virtualized appliance on one server

**Multi-node bare metal**

- `X11-HA` is the two-node HA hardware model
- A bare metal deployment here creates a clustered bare metal appliance across both nodes

The useful mental model is:

- Single-node bare metal is the simpler path
- HA bare metal is the clustered path with all the extra availability, coordination, and networking consequences that phrase quietly implies

---

## 3. Deployment Methods (BUI for sanity, JSON for control)

You can deploy ODA bare metal in two main ways:

- Browser User Interface (`BUI`)
- `odacli` with a JSON configuration file

**Browser User Interface**

- This is the most straightforward deployment method for most people
- It exposes the main system, network, user, database, and ASR fields in one guided workflow
- On an unconfigured appliance, the BUI explicitly tells you the appliance is not configured and provides the link to start creating it

The BUI is not magic, but it does at least reduce the number of ways you can ruin a deployment with a typo and confidence.

**JSON plus `odacli`**

- This is the command-line path for teams that want repeatability or finer control
- You use `odacli create-appliance` with a JSON file that captures the deployment settings
- Oracle provides sample files and a readme under `/opt/oracle/dcs/sample`

This is the right choice when:

- You want a scripted deployment
- You need to version control your appliance configuration
- You trust yourself with JSON more than you trust point-and-click wizardry

It is also the right choice if you enjoy learning exactly how bad a wrong Oracle ILOM entry can make your day.

---

## 4. The Base Software Stack (the stuff that makes the hardware more than just a heated rectangle)

ODA bare metal starts from a factory or reimaged base state and then layers in the appliance software stack.

The relevant software pieces include:

- Oracle Linux
- Hardware drivers and firmware-aware management components
- Oracle Appliance Manager and the `odacli` tooling
- Oracle Grid Infrastructure components
- Oracle Database components
- Monitoring and management support such as Oracle ILOM and ASR integration

Current Oracle documentation describes the `Grid Infrastructure/Database Clone` bundle as the package used to perform the initial deployment on an appliance in factory state.

That bundle includes:

- Grid Infrastructure components
- Database components
- Oracle Database Appliance Manager software
- Oracle Linux and hardware drivers needed for deployment

This is Oracle's preferred version of convenience: one bundle that still contains several very meaningful ways to ruin an afternoon if handled carelessly.

---

## 5. Clones and Why Oracle Uses Them (because downloading random bits from the internet is not an appliance strategy)

ODA uses appliance-specific clone files instead of treating deployment like a generic Oracle download exercise.

The important clone types are:

- Grid Infrastructure / Database Clone bundle
- Oracle Database Clone files

Why clones matter:

- They are tested specifically for Oracle Database Appliance
- They are aligned with the appliance release and patching model
- They reduce deployment time compared with assembling components manually
- They preserve Oracle's supported and validated configuration path

The key distinction is:

- The GI/DB clone bundle is used for the initial appliance deployment
- The database clone files are used to create database homes and databases for supported releases

This is Oracle's way of saying, "please use the engineered path we gave you instead of inventing your own unsupported masterpiece."

---

## 6. Server Patches, Firmware, and Component Versions (because "factory state" is not the same thing as "current")

Before creating the appliance, you need to know whether the base system firmware and software are current.

Oracle's current bare metal workflow has you:

1. Download the latest server patch
2. Check the current component versions
3. Update server and storage components before deployment if they are out of date

The key command for that version check is:

```bash
/opt/oracle/dcs/bin/odacli describe-component
```

For HA systems, Oracle documents checking the system version on both nodes using:

```bash
/opt/oracle/dcs/bin/odacli describe-component -v
```

If the firmware is not current, Oracle's deployment guidance has you update:

- Server components
- Storage components

before the appliance is fully deployed.

That is an important operational distinction:

- You are not just deploying database software
- You are making sure the underlying appliance stack is current enough not to sabotage the rest of the build

---

## 7. Preparing the Software for Deployment (download, copy, update repository, check the job, try not to improvise)

Before you create the appliance, download the required software from My Oracle Support and stage it on the ODA.

Common file movement options include:

- `scp`
- `sftp`
- USB storage formatted as `FAT32`, `ext3`, or `ext4`

For current X11 bare metal provisioning, Oracle documents a flow like this:

1. Download the GI and DB clone software from My Oracle Support
2. Copy the files to a temporary location on the appliance
3. Run `odacli update-repository`
4. Confirm the repository-update job completed successfully
5. Remove temporary zip files to save space

Representative command:

```bash
/opt/oracle/dcs/bin/odacli update-repository -f /tmp/GI_clone_file,/tmp/DB_clone_file
```

And then verify the job:

```bash
/opt/oracle/dcs/bin/odacli describe-job -i job_ID
```

The practical point is:

- The appliance is not really ready to deploy until the repository is ready

And yes, that is exactly as fun as it sounds.

---

## 8. Predeployment Checklist for Bare Metal (the chores before the wizard gets involved)

Before starting the actual bare metal deployment, make sure you have already completed the basics:

- Attach the network labels
- Complete the first power-on successfully
- Configure Oracle ILOM
- Configure the initial network connection
- Download the latest software
- Copy the required software to the appliance
- Verify firmware and component versions
- Open the BUI or prepare the JSON file for `odacli create-appliance`

For HA systems, also remember:

- Validate the storage topology and cabling on both nodes
- Treat both nodes as part of the deployment from the start instead of pretending Node1 is some optional sequel

---

## 9. Creating the Appliance (this is the actual "deploy the thing" step, not the 47 steps before it)

Once the repository is updated and the network is plumbed, you can create the bare metal appliance.

**Using the BUI**

- Open `https://ODA-host-ip-address:7093/mgmt/index.html`
- Ensure the required client-to-appliance ports are open
- Set the `oda-admin` password when prompted
- Log in and follow the guided Create Appliance flow

The BUI lets you populate:

- System information
- Network information
- Users and groups
- Optional initial database settings
- Optional Oracle ASR settings

**Using `odacli`**

- Build or adapt a JSON file
- Review the examples in `/opt/oracle/dcs/sample`
- Run `odacli create-appliance`

The CLI route is more powerful, more automatable, and more likely to punish sloppy inputs immediately.

---

## 10. Security Settings and Passwords (because every appliance ships with defaults, and defaults are how headlines happen)

Bare metal deployment includes several important password and ownership behaviors.

During deployment, Oracle documents that the system password becomes the password for:

- `root`
- `SYS`
- `SYSTEM`
- `PDBADMIN`

After deployment, that same password is also applied to:

- `oracle`
- `grid`

That is convenient for deployment and terrible as a long-term security philosophy, so Oracle recommends changing those passwords promptly after provisioning.

There is also the BUI administrator account:

- User name: `oda-admin`
- On an unconfigured appliance, you are prompted to set the password when you first open the BUI

Current ODA guidance also documents changing the `oda-admin` credential after deployment with:

```bash
odacli set-credential --username oda-admin
```

The security takeaway is brutally simple:

- Default and deployment-time passwords are for getting the system stood up
- They are not a respectable end state

---

## 11. Owners, Groups, and Role Separation (who owns what, and how much trouble that prevents later)

ODA bare metal supports Oracle's role-separation model for installation owners and groups.

The classic default split is:

- `oracle` as the Oracle Database installation owner
- `grid` as the Oracle Grid Infrastructure installation owner

Common default groups include:

- `oinstall`
- `dba`
- `dbaoper`
- `asmadmin`
- `asmoper`
- `asmdba`

Supported user/group patterns include:

- Two users with six groups
- Single user with six groups
- Single user with two groups

If you care about separation of duties, auditability, or limiting the blast radius of mistakes, decide that during deployment instead of after everyone has already started using the box like it belongs to them personally.

---

## 12. Key Takeaways (the part where the hardware becomes software, and the software becomes your problem)

- Bare metal ODA is a non-virtualized deployment directly on the appliance hardware
- `X11-S` and `X11-L` are the single-node bare metal models; `X11-HA` is the multi-node HA bare metal model
- The BUI is the simplest deployment path, while `odacli create-appliance` plus JSON provides more control
- ODA uses appliance-specific clone files rather than generic Oracle install media
- The GI/DB clone bundle is central to initial deployment, and DB clone files are used for database-home and database creation workflows
- Factory state is not the same thing as current state, so verify component versions and update server or storage components if needed before deployment
- Stage the software, update the repository, verify the job, and only then create the appliance
- Deployment-time passwords are not the same thing as acceptable long-term security
- Change the `oda-admin`, `root`, `oracle`, `grid`, and database administrative passwords after deployment

---

## 13. Wrap-Up (the appliance is finally becoming a platform, which is another word for "your responsibility now")

You now know what a bare metal ODA deployment is, how it differs across single-node and HA hardware, which software bundles and clone files matter, how the repository-update workflow fits in, and where the key security and password decisions show up. Next comes the part where the bare metal system is actually used and managed, which is where Oracle's elegant diagrams get handed over to real administrators with real change windows.
