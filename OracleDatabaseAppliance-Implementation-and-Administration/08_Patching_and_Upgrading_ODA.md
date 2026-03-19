## Lesson 8 - Patching and Upgrading ODA (in which Oracle politely asks you to patch the whole appliance as one system instead of freelancing your own maintenance disaster)

Welcome to the patching chapter, where Oracle Database Appliance reminds you that an engineered system only stays engineered if you resist the very human urge to patch random components individually and then act surprised when the metadata, inventory, and future upgrades all burst into flames.

By the end of this lesson, you should be able to:

- Explain the Oracle Database Appliance patching model for bare metal systems
- Identify what is included in the Oracle Database Appliance patch bundle
- Understand why Oracle requires appliance-level patching instead of component-by-component patching
- Describe the Browser User Interface and `odacli` options for patching bare metal ODA
- Explain how Oracle's upgrade-path rules and minimum supported releases affect patch planning
- Summarize the main prechecks and patching sequence used to update a bare metal appliance

---

## 1. The ODA Patching Model (one bundle to rule the maintenance window)

Oracle Database Appliance uses an appliance-level patching model.

That means patching is performed with the regular Oracle Database Appliance patch bundle rather than by patching individual components one at a time.

Current ODA documentation describes the patch bundle as covering the appliance as a whole, including items such as:

- Oracle Database Appliance server software
- DCS Admin
- DCS components
- DCS Agent and related management services
- Oracle Appliance Manager
- Oracle Linux
- Oracle ILOM
- BIOS
- Hardware drivers
- Firmware and Hardware Management Pack components
- Oracle Auto Service Request (`ASR`)
- Oracle IPMI
- Network-card-related fixes for relevant hardware

This is Oracle's way of saying the appliance is a system, not a buffet.

---

## 2. What You Must Not Do (the forbidden patching arts)

Oracle is extremely direct about what not to do.

Do **not** patch ODA using:

- Generic Oracle Grid Infrastructure patches
- Generic Oracle Database home patches applied outside the appliance workflow
- Individual Oracle Linux patching as a substitute for ODA patching
- Individual firmware patching as a substitute for ODA patching
- `OPatch` or similar tools as your main appliance patching method

Why this matters:

- ODA maintains its own metadata and inventory for lifecycle management
- If you patch outside the ODA workflow, that metadata becomes inaccurate
- Once the appliance inventory is wrong, future updates become much more difficult, which is Oracle's polite phrasing for "you broke the one thing designed to keep this manageable"

So yes, the official rule really is:

- Patch ODA with ODA patches
- Do not improvise

---

## 3. Out-of-Place Patching (because Oracle would prefer new homes over surgery on the old ones)

Starting with Oracle Database Appliance release `19.11`, ODA uses an out-of-place patching model for Oracle Grid Infrastructure and Oracle Database homes.

That means:

- New Oracle homes are created as part of patching
- Existing homes are not patched in place in the old style

This reduces some upgrade risk and makes rollback logic less horrifying than the older patch-in-place model, though "less horrifying" is still doing a lot of work there.

The practical consequence is:

- Appliance patching updates the infrastructure stack
- Database and DB home updates then move to newer homes created for the new release

---

## 4. Why Oracle Likes the ODA Patching Workflow (and, annoyingly, has a point)

The `odacli`-driven patching workflow is designed to reduce:

- Manual research
- End-to-end integration guesswork
- One-off component patching
- The need to understand every internal dependency before applying updates

That does not make patching trivial.

It does make patching more repeatable and more appliance-aware than manually touching Grid Infrastructure, Oracle Linux, firmware, and management software as separate projects with separate mistakes.

---

## 5. Patching Paths for Bare Metal (buttons for some, commands for others)

Oracle supports patching bare metal ODA through:

- The Browser User Interface (`BUI`)
- `odacli`

**Browser User Interface path**

- Upload the Oracle Database Appliance Server Patch to the repository
- Use `Appliance -> Infrastructure Patching` to step through `DCSADMIN`, `DCSCOMPONENTS`, `SERVERCOMPONENTS`, `GI`, and `STORAGE`
- Use the `Database Home` and `Database` pages afterward to patch homes and databases
- On current bare metal releases, the BUI can drive the full infrastructure patch sequence directly
- Older course material and older ODA releases may still show `Patch Manager` terminology or extra CLI-first steps for some parts of the flow

**CLI path**

- Check the current versions
- Stage the new patch bundle and clone files on the appliance
- Update the repository with the new patch bundle
- Update `DCS Admin`
- Update `DCS Components`
- Run server-component prechecks
- Apply the server-components patch
- Run Grid Infrastructure prechecks
- Update the Grid Infrastructure home
- Run storage prechecks and patch storage components if required
- Update database homes and databases as needed

The BUI is more visual.
The CLI is more explicit.
Neither one excuses you from reading the release notes first.

---

## 6. Prechecks and Patching Discipline (because Oracle would rather detect the fire before you light it)

ODA patching includes prechecks, and Oracle requires you to take them seriously.

Prechecks validate things such as:

- DCS Agent health on all nodes
- Available space for patching
- Minimum supported versions
- Patch tags and repository consistency
- Oracle Clusterware and ASM state
- Database-home patch readiness
- Storage and firmware readiness
- Oracle ORAchk findings

For current bare metal patching, Oracle explicitly requires you to run the prepatch report before `odacli update-servercomponents`.

Representative command:

```bash
/opt/oracle/dcs/bin/odacli create-prepatchreport -sc -v target_release
```

Oracle also lets you generate a combined infrastructure precheck so you are not forced to run three separate rounds of panic:

```bash
/opt/oracle/dcs/bin/odacli create-prepatchreport -sc -gi -st -v target_release
```

And for database home patching, current guidance requires a separate DB-home precheck before `odacli update-dbhome`.

For HA systems, Oracle expects the full precheck flow to cover both nodes unless you are intentionally using a local-only option for a narrower task.

This is the part where the appliance tries to save you from the worst version of yourself.

---

## 7. Basic Bare Metal CLI Patching Sequence (the order matters, and Oracle means it)

Oracle's current bare metal patching guidance follows this general sequence:

1. Download the Oracle Database Appliance Server Patch from My Oracle Support
2. Download the GI clone and DB clone files for the target release
3. Copy the software to the appliance, typically to a temporary location such as `/tmp`
4. Unzip the server patch bundle and read the included `README`
5. Update the repository with the server software file
6. Update `DCS Admin`
7. Update `DCS Components`
8. Update the repository with the GI and DB clone files
9. Run prepatch checks for server components
10. Apply the server-components update
11. Run prepatch checks for Oracle Grid Infrastructure
12. Update the Oracle Grid Infrastructure home
13. Run storage prechecks and patch storage if needed
14. Patch or update database homes and databases to the new release as needed

Important Oracle note:

- Run `odacli update-dcsadmin` and `odacli update-dcscomponents` **before** `odacli update-servercomponents` and `odacli update-gihome`

Representative commands:

```bash
/opt/oracle/dcs/bin/odacli update-repository -f /tmp/oda-sm-release-server.zip
/opt/oracle/dcs/bin/odacli update-dcsadmin -v target_release
/opt/oracle/dcs/bin/odacli update-dcscomponents -v target_release
/opt/oracle/dcs/bin/odacli create-prepatchreport -sc -gi -st -v target_release
/opt/oracle/dcs/bin/odacli update-servercomponents -v target_release
/opt/oracle/dcs/bin/odacli update-gihome -v target_release
/opt/oracle/dcs/bin/odacli update-storage -v target_release --rolling
```

And then confirm job completion with:

```bash
/opt/oracle/dcs/bin/odacli describe-job -i job_ID
```

For current releases, `odacli update-dcscomponents` may be tracked as an admin job, so `odacli list-admin-jobs` and `odacli describe-admin-job -i job_ID` can be part of the reality on the ground.

That sequence is not Oracle being fussy. It is Oracle trying to stop you from updating the appliance in the wrong order and then discovering the consequences at 2:17 AM.

If you are reading older ODA material, you may still see references to `odacli update-server` or `odacli update-dcsagent`.

Current Oracle guidance for recent X11 releases uses the newer flow:

- `odacli update-dcsadmin`
- `odacli update-dcscomponents`
- `odacli update-servercomponents`
- `odacli update-gihome`
- `odacli update-storage`

---

## 8. Bare Metal Patching from the BUI (for people who enjoy buttons but still have to respect the sequence)

The current BUI patching flow for bare metal is roughly:

1. Download the Oracle Database Appliance patch files from My Oracle Support
2. Copy the files onto the appliance
3. Log into the BUI
4. Open `Repository Manager`
5. Use `Update Patch Repository` to upload the patch files by absolute path
6. Monitor the repository update job in `Activity`
7. Open `Appliance -> Infrastructure Patching`
8. Refresh the component details
9. In order, patch `DCSADMIN`, `DCSCOMPONENTS`, `SERVERCOMPONENTS`, `GI`, and `STORAGE`
10. Run `Precheck` before the infrastructure sections that require it
11. Review the precheck reports
12. Click `Apply Patch` only after the report is clean

Important current nuance:

- For current bare metal releases, Oracle documents patching `DCS Admin` and `DCS Components` directly in the BUI
- Some older ODA releases also document a separate `odacli update-dcsagent` step, so if you are working from release-specific notes, check the exact commands for that version
- For current releases, the infrastructure patching flow covers server components, then Oracle Grid Infrastructure, then storage, rather than treating everything as one old-style `update-server` action

For HA systems, the BUI also allows choices such as:

- Selecting which node to update
- Updating both nodes
- Using rollback-on-failure for server component patching
- Using rolling storage patching where supported

So yes, the BUI is helpful. No, it is not a magical exemption from the actual patching order.

---

## 9. Upgrade Paths, Minimum Versions, and Patch Staging (because patching from ancient releases is how legends of pain begin)

Oracle recommends patching from within the previous four ODA releases because those paths are the ones Oracle tests.

That does **not** mean "any old version will wander happily to the newest release if you believe in yourself."

Oracle release notes also publish minimum patch requirements for newer target releases.

That means:

- Every target release has an earliest supported version you can patch from
- Some releases require an intermediate upgrade before you can go farther
- If your appliance is below the required minimum, you patch to the minimum supported release first and only then continue upward

For example, Oracle's February 24, 2026 release notes for `19.30.0.0` list bare metal patching support from `19.29.0.1.0`, `19.29.0.0`, `19.28.0.0`, `19.27.0.0`, and `19.26.0.0`.

The staging workflow is refreshingly unglamorous:

- Log into My Oracle Support from an external machine
- Download the ODA Server Patch and the required GI and DB clone files for the target release
- Copy the files to the appliance using `scp`, `sftp`, or attached USB storage
- Use a temporary directory such as `/tmp`
- Unzip the server patch bundle
- Read the included `README` before you start pressing buttons like a raccoon in a microwave

Oracle generally publishes full ODA patch bundles on a quarterly cadence, and current releases also document monthly Oracle Linux CVE updates for recent versions.

Important operational warnings from current guidance:

- Ensure there is enough local space to stage the patches
- Ensure no jobs are pending or running during the patch window
- Public-network gateway reachability matters in HA environments
- Third-party RPM customizations can interfere with appliance patching
- For some storage-controller combinations, extra firmware-reset steps may be required before patching

Oracle also documents the `ODABR` tool as part of the safety story for bare metal patching on newer releases.

That is Oracle's way of admitting that even well-designed patch workflows sometimes benefit from an emergency parachute.

---

## 10. Patching Database Homes and Databases (because infrastructure is only half the circus)

Once the appliance infrastructure is patched and the correct Oracle Database clone files are in the repository, you still have to patch the database homes and, if needed, the databases themselves.

This matters because databases live in homes, and homes do not upgrade themselves out of politeness.

**Database home patching in the BUI**

- Open the `Database Home` tab
- Select the home you want to patch
- Choose the patch version
- On HA systems, choose the node to update or all nodes
- Run `Precheck`
- Review the pre-patch report
- Apply the patch if the report is clean

**Database home patching in the CLI**

Current Oracle guidance uses a DB-home precheck before the actual home patch:

```bash
/opt/oracle/dcs/bin/odacli create-prepatchreport --dbhome --dbhomeid DB_HOME_ID -v target_release
/opt/oracle/dcs/bin/odacli update-dbhome --id DB_HOME_ID -v target_release
```

**Database patching in the BUI**

- Open the `Database` tab
- Select the database
- Click `Update`
- Choose the destination database home
- Run `Precheck`
- Review the report
- Apply the update

**Database patching in the CLI**

Oracle documents a database-specific workflow when you want to patch a database into a chosen destination home:

```bash
/opt/oracle/dcs/bin/odacli create-prepatchreport --database --database-id DATABASE_ID --to-home DEST_DBHOME_ID
/opt/oracle/dcs/bin/odacli update-database --id DATABASE_ID --to-home DEST_DBHOME_ID
```

This is the practical difference that trips people up:

- `update-dbhome` patches a home
- `update-database` moves a database into a target patched home

So if several databases share the same home, patching that home affects the whole shared arrangement. If you want fine-grained control, you patch or create the destination home first and then move the specific database where you want it.

---

## 11. Key Takeaways (the maintenance-window survival kit)

- Bare metal ODA must be patched with Oracle Database Appliance patch bundles, not with individually applied component patches
- The patch bundle covers the full appliance stack, including DCS, Oracle Linux, firmware, Oracle ILOM, and related components
- Out-of-place patching is the modern ODA model for Oracle homes
- Oracle tests patching paths within the previous four releases, and newer target releases can impose minimum source-version requirements
- The main patching paths are the BUI and `odacli`; on current bare metal releases, the BUI can handle the full infrastructure flow, while older materials may show extra CLI-only steps
- `odacli update-dcsadmin` and `odacli update-dcscomponents` must run before `odacli update-servercomponents`, `odacli update-gihome`, and the rest of the infrastructure flow
- Prechecks are mandatory and exist to stop preventable disasters before they become actual disasters
- Database homes and databases are patched after infrastructure and require clone files plus their own prechecks
- Older references to `update-dcsagent` or `update-server` are release-specific fossils, not the safest default for current X11 systems

---

## 12. Wrap-Up (the patch bundle is your friend, even if maintenance windows are not)

You now know how Oracle wants bare metal ODA patching to work: one appliance-aware patching model, one supported bundle-driven process, one upgrade path constrained by actual minimum versions, and one deeply sincere warning not to freestyle your own infrastructure maintenance theology. The appliance can absolutely be patched cleanly. It just wants the steps followed in the right order, which is honestly not too much to ask from a system holding your databases hostage.
