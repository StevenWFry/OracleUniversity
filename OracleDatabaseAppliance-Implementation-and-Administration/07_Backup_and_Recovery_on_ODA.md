## Lesson 7 - Backup and Recovery on ODA (in which everyone remembers that databases are mortal and optimism is not a retention policy)

Welcome to the backup and recovery chapter, where Oracle Database Appliance stops being a platform you deploy and starts being a platform you must protect from disk failures, user mistakes, bad upgrades, and the timeless enterprise tradition of deleting the wrong thing with absolute confidence.

By the end of this lesson, you should be able to:

- Explain the backup and recovery model used on Oracle Database Appliance
- Create and attach a backup policy for an ODA database
- Distinguish the major backup levels used by ODA
- Describe the main recovery options available for full database recovery
- Understand the workflows for backing up to disk and Oracle Object Storage

---

## 1. Why Backup and Recovery Matters on ODA (because engineered systems still contain humans)

Every organization running ODA needs a backup and recovery strategy.

ODA is not immune to the usual reasons databases suffer:

- Disk failures
- Corruption
- User error
- Administrative mistakes
- Upgrade failures
- Site-level problems, depending on the architecture around it

Oracle Database Appliance provides backup and recovery features that let you:

- Create a backup policy
- Attach that policy to a database
- Schedule regular backups automatically
- Restore or recover the database when the day inevitably becomes educational

This is not optional hygiene. It is the difference between "we had an incident" and "we have a career-changing story."

---

## 2. Backup Policy Basics (the part where you tell the appliance how paranoid to be)

On ODA, backups are driven by a backup policy.

The backup policy defines things such as:

- Backup destination
- Recovery window
- Crosscheck behavior
- Object Store credential details when applicable
- Compression settings when applicable

You can create a backup policy in:

- The Browser User Interface
- The command-line interface

And then you can:

- Attach the policy during database creation
- Attach it later to an existing database
- Update it if the backup design changes

Once a database has a backup policy attached, the `DCS` agent automatically schedules:

- Database backups
- Archive log backups

Default schedule:

- Level 0 backup every Sunday
- Level 1 backup Monday through Saturday
- Archive log backup every 30 minutes

You can change the backup day and scheduler frequency, because Oracle understands that not every organization wants Sunday to be its default weekly act of faith.

---

## 3. Backup Destinations (pick your recovery flavor)

Current ODA documentation supports three main backup destinations:

- Internal FRA (`Disk`)
- Oracle Object Storage
- External FRA (`NFS`)

This lesson focuses mainly on the first two, because those are the workflows emphasized in the source material.

**Disk / Internal FRA**

- Backups are written to the Oracle Fast Recovery Area on disk
- Recovery is fast because the backups are local
- It requires significant local storage, often up to two to three times the database size

**Oracle Object Storage**

- Backups are stored in OCI Object Storage
- This gives you off-box durability and remote retrieval
- It requires configured Object Store credentials and a target bucket/container

**NFS / External FRA**

- Current ODA releases also support Network File System backup targets
- NFS can be used as a centralized external backup location

Practical takeaway:

- Local disk is convenient for speed
- Object Storage is useful for off-appliance backup durability
- NFS exists if the environment is already built around that storage pattern

---

## 4. Backup Levels (full, incremental, and the archival box you do not want to discover is missing)

ODA supports several backup types.

**Level 0**

- A full backup of all blocks in the datafiles
- Serves as the parent for later level 1 backups
- By default, occurs weekly on Sunday

This is the big baseline backup. It is the adult in the room.

**Level 1**

- An incremental backup that captures changed blocks
- By default, runs Monday through Saturday
- ODA's scheduled level 1 behavior is differential by default, meaning it captures changes since the most recent level 0 or level 1 backup

The important idea is:

- Level 0 gives you the base image
- Level 1 gives you the changes after that

**LongTerm**

- A long-term or archival backup
- Includes everything needed to restore and recover the database
- Exempt from the normal backup retention policy

Important current nuance:

- Long-term backups require an `NFS` or `Object Storage` backup policy in current ODA documentation

**Archivelog**

- Captures archive logs not yet backed up to the destination
- Scheduled automatically when the backup policy is attached
- Defaults to every 30 minutes

That archive-log schedule is the quiet little detail that matters enormously when somebody later asks how much data loss you are willing to tolerate.

---

## 5. Backup Reports (the metadata receipt you will care about very deeply when things go wrong)

For every backup, the `DCS` agent generates and stores a backup report.

The backup report contains the metadata required for restore and recovery operations.

It is not a full recovery catalog replacement, but it is absolutely part of the recovery story.

Why it matters:

- It records the metadata required to recover or restore a database
- It supports backup-report-based recovery
- It gives you a durable reference to what was backed up and when

This is the document you will either be grateful exists or furious nobody saved properly.

---

## 6. Recovery Options (four ways to ask Oracle to rewind the damage)

ODA recovery always performs a full or whole database restore/recover operation using `RMAN`.

The main recovery options are:

- `LATEST`
- `PITR`
- `SCN`
- `BackupReport`

**LATEST**

- Performs complete recovery
- Requires valid backups and all required archived logs and online redo logs

Use this when the goal is:

- Get back to the latest recoverable state

**PITR**

- Point-in-Time Recovery
- Recovers to a specified timestamp within the current incarnation of the database

Use this when:

- A user error happened
- A bad change happened at a known time
- An upgrade or administrative operation needs to be rewound

**SCN**

- Performs incomplete recovery to a specific System Change Number
- Useful when you know the exact SCN target

This is the precise version of "take me back before things got stupid."

**BackupReport**

- Uses the SCN recorded in the backup report
- Similar to `SCN` recovery, except the SCN is taken from the backup report instead of being manually specified

That makes backup-report recovery useful when you trust the backup artifact more than your memory of exactly when the disaster started.

---

## 7. Backup to Disk Workflow (fast recovery, expensive disk appetite)

The high-level workflow for backing up to disk is:

1. Create a backup policy with destination set to `Disk`
2. Define the recovery window
3. Attach the backup policy to the target database
4. Allow automatic backups to run
5. Manage obsolete backups over time
6. Restore or recover the database from the available backups

Why teams like disk backups:

- Local recovery is faster
- FRA is tightly integrated with Oracle recovery operations

Why teams resent disk backups:

- They consume a lot of local storage
- They are still on the appliance side of the disaster boundary

This is convenient recovery, not magical immunity.

---

## 8. Backup to Oracle Object Storage Workflow (because off-appliance backups are what adults eventually ask for)

The high-level workflow for backing up to Oracle Object Storage is:

1. Create Object Store credentials
2. Create a backup policy with destination set to `Object Store`
3. Specify the Object Store credential name or resource, the bucket/container, and the recovery window
4. Attach the backup policy to the target database
5. Allow automatic backups to run
6. Manage obsolete backups
7. Restore or recover from the stored backups when needed

ODA stores Object Store credential details in an encrypted Oracle wallet.

Object Storage advantages include:

- Off-appliance backup retention
- Secure remote storage
- Better durability against appliance-local failures
- Built-in encryption behavior for the storage platform

Practical warning:

- For non-TDE databases, backup encryption passwords matter and must be saved safely
- If you lose the needed backup password, you do not get to solve that with positive thinking

---

## 9. Creating and Attaching a Backup Policy (the step that turns backup from idea into behavior)

At a practical level, the workflow is:

1. Create the backup policy
2. Attach it to the database
3. Verify the schedules are in place
4. Optionally run a manual backup immediately

### Creating a Backup Policy in the BUI

To create a backup policy from the Browser User Interface:

1. Click the `Database` tab
2. Click `Backup Policy` in the left navigation
3. Click `Create Backup Policy`
4. Enter the backup-policy name
5. Select the recovery window in days
6. Choose the backup destination
7. Configure any destination-specific fields
8. Click `Create`
9. Confirm the action

Destination choices include:

- `Internal FRA` for disk backup
- `ObjectStore` for Oracle Object Storage backup
- `External FRA` for `NFS`
- `None` if you are defining the policy shell before setting a destination

Important practical warning:

- Disk backup can consume up to two or three times the size of the database
- Check available space before you create a disk-based policy, unless you enjoy building your own storage shortage

If you choose Object Storage, you also need:

- An Object Store credential
- A target bucket or container name

If you are attaching the policy to a TDE-enabled database, current ODA documentation also requires a TDE wallet backup location.

### Updating an Existing Backup Policy

To update a backup policy in the BUI:

1. Click the `Database` tab
2. Click `Backup Policy`
3. Open `Actions` for the target policy
4. Click `Update`
5. Change the recovery window, crosscheck option, or Object Store credential as needed
6. Click `Update`
7. Confirm the action

This lets you adjust policy behavior without pretending the original settings were somehow carved into stone tablets.

### Attaching a Backup Policy to an Existing Database

To attach a backup policy to an existing database in the current BUI flow:

1. Click the `Database` tab
2. Open `Actions` for the target database
3. Click `Modify`
4. Select a backup policy from the `Select Backup Policy` list
5. Specify and confirm the backup encryption password when required
6. Click `Modify`
7. Check the `Activity` tab for job completion

When the job completes successfully:

- The backup policy is associated with the database
- The `DCS` agent schedules automatic database backups
- The `DCS` agent also schedules archive-log backups

Afterward, review the database details page to confirm the new backup-policy and destination information are shown correctly.

CLI examples look like this:

```bash
odacli create-backupconfig -d Disk -n prod_disk_policy -w 7
odacli modify-database -in mydb -bin prod_disk_policy -bp
```

And for Object Store:

```bash
odacli create-objectstoreswift -e https://swift-endpoint -n objstore1 -t tenancy -u username
odacli create-backupconfig -d objectstore -n prod_obj_policy -on objstore1 -c backupbucket -w 7
odacli modify-database -in mydb -bin prod_obj_policy -bp
```

Once that policy is attached, the appliance stops waiting for your good intentions and starts making actual backups.

---

## 10. Performing Backup and Recovery Operations (the part where the theory gets tested)

You can use either:

- The Browser User Interface
- `odacli`

to perform backup, restore, and recovery operations.

Operationally, the common pattern is:

1. Attach or verify the backup policy
2. Run or wait for a successful backup
3. Save or verify the backup report
4. Choose the correct recovery mode when needed
5. Restore and recover the database

Recovery planning questions to ask before you click anything:

- Do you want the latest recoverable state?
- Do you need to rewind to a point in time?
- Do you know the exact SCN?
- Is the backup report the safest recovery reference?

Those questions determine whether the recovery is calm, precise, and intentional, or whether it turns into a beautifully documented panic.

### Creating a Manual Backup from the BUI

After a backup policy is attached to a database, you can create a manual backup outside the normal schedule.

To create a manual backup in the Browser User Interface:

1. Click the `Database` tab
2. Click the target database name
3. Review the current backup policy, backup destination, and related details
4. Click `Manual Backup`
5. Select the backup type from the list
6. Confirm the action

The same area also provides:

- `Update Database Backup Schedule`
- `Update Archive Log Backup Schedule`

If any of these actions are disabled, the usual reason is simple:

- The database does not yet have a backup policy attached

Once the job completes successfully:

- The manual backup appears in the backup list
- A backup report is generated
- You can review the job from the `Activity` tab

This is useful when you want a backup before:

- A risky change
- A patching window
- A database upgrade
- Any operation likely to end with someone saying "that was not supposed to happen"

### Recovering a Database from the BUI

To recover a database from backup in the Browser User Interface:

1. Click the `Database` tab
2. Select the target database
3. On the Database Information page, click `Recover`
4. Choose the recovery option
5. Provide the backup encryption password when required
6. Click `Recover Database`
7. Track the job in the `Activity` tab

Common BUI recovery choices include:

- Recover full database to the specified backup
- Recover full database to the latest recoverable state
- Recover full database to a specified timestamp
- Recover full database to a specified `SCN`

The exact recovery path you choose depends on the failure scenario:

- `Latest` for least possible data loss
- Timestamp for a known point of bad user behavior
- `SCN` for precise recovery targets
- Specific backup when you intentionally want to anchor recovery to that backup artifact

And yes, Oracle will absolutely ask for the backup encryption password at exactly the moment you realize nobody wrote it down in a respectable place.

---

## 11. Key Takeaways (the part where backup strategy becomes less theoretical and more survival-oriented)

- ODA backup and recovery is policy-driven
- Once a backup policy is attached, the `DCS` agent schedules database and archive-log backups automatically
- The BUI path for backup policy management is `Database -> Backup Policy`
- Attaching a backup policy to an existing database is done through `Actions -> Modify` on the database
- Default scheduling is Level 0 on Sunday, Level 1 Monday through Saturday, and archive-log backup every 30 minutes
- The main backup destinations are `Disk`, `Oracle Object Storage`, and, in current ODA documentation, `NFS`
- `Level 0`, `Level 1`, `LongTerm`, and `Archivelog` backups serve different purposes and should not be treated as interchangeable
- ODA recovery is a full `RMAN` database recovery using `LATEST`, `PITR`, `SCN`, or `BackupReport`
- Backup reports matter because they contain the metadata needed for restore and recovery
- Disk backup is fast and local; Object Storage backup is more durable beyond the appliance itself
- Disk-based backup policies require serious space planning before you create them
- Manual backups, backup-schedule changes, and archive-log schedule changes are all available from the database details page in the BUI
- Database recovery from the BUI is driven from the `Recover` action on the database details page and requires the correct recovery mode and backup password when applicable

---

## 12. Wrap-Up (because the real value of backups is only revealed when everything else has gone wrong)

You now know how ODA backup policy works, what the main backup levels are, how the recovery options differ, and how disk and Object Storage backup workflows are structured. Next comes the rest of the administrative reality: patching, diagnostics, networking, storage, and all the other ways an appliance politely reminds you that uptime is a full-time hobby.
