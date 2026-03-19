## Lesson 6 - Managing Databases and Oracle Homes in Bare Metal (in which the appliance is finally live, and now you have to behave like someone who owns databases on purpose)

Welcome to the part of ODA administration where the hardware drama settles down and the database management drama begins. This is where you stop asking "did the appliance deploy?" and start asking "what exactly is running on it, where does it live, and how quickly can I delete the wrong thing if I click like an optimist?"

By the end of this lesson, you should be able to:

- View existing databases in the Browser User Interface
- Create a new database using the Browser User Interface
- Delete a database safely from the Browser User Interface
- Upgrade a database to a different database home version
- View, create, and delete database homes from the Browser User Interface
- Understand how database homes fit into database creation and upgrade workflows on bare metal ODA

---

## 1. Viewing Existing Databases (the first step in not accidentally managing the wrong one)

To view databases in ODA:

1. Log in to the Browser User Interface:
   `https://host-name-or-ip:7093/mgmt/index.html`
2. Click the `Database` tab
3. Review the list of existing databases

The database list shows details such as:

- Database name
- Database ID
- Database type
- Database version
- Database class
- Shape
- Storage type
- Status
- Associated database home

Useful navigation behaviors:

- Click the database name to see more details
- Use the `Actions` menu next to a database entry to view more details, upgrade, modify, or delete the database

This is the part where you verify what is actually on the appliance before you start pressing buttons like a caffeinated intern in a change window.

---

## 2. Creating a New Database (because an appliance without databases is just a very expensive Linux conversation)

Before creating a database, make sure the appliance repository already contains the required Oracle RDBMS clone files for the database version you want.

To create a database from the BUI:

1. Log in to the Browser User Interface
2. Click the `Database` tab
3. Click `Create Database`
4. Select `Create Database`
5. Enter the required database information
6. Click `Create`
7. Confirm the action when prompted

Common required fields include:

- `DB Name`
- Optional `DB Unique Name`
- Whether to use an existing DB home or create/select a versioned home
- Database version
- Database edition
- Database class such as `OLTP`, `DSS`, or `IMDB`
- Storage type such as `ASM`
- Deployment type
- Node selection for single-instance deployments
- High availability option where supported
- Database shape
- Administrative passwords

Important naming rule:

- The database name must contain lowercase alphanumeric characters only
- It cannot exceed `8` characters
- The SID is set to the database name

That is one of those gloriously Oracle constraints that feels petty until you ignore it and the wizard politely refuses to continue.

The shape selection matters because it determines:

- Core allocation
- Memory allocation
- Default workload sizing behavior

So when you pick a shape, you are not choosing a decorative label. You are choosing how much database gets to exist before the appliance starts side-eyeing your resource assumptions.

---

## 3. Tracking the Create Job (because "I clicked Create" is not the same thing as "it worked")

After you submit the database creation job, ODA gives you a job ID or Activity entry to track progress.

You can monitor it in either place:

- The `Activity` tab in the Browser User Interface
- The command line with `odacli describe-job`

Representative check:

```bash
odacli describe-job -i jobId
```

That matters because database creation is not instantaneous, and the appliance has no obligation to turn your optimism into a successful job without first making you wait.

---

## 4. Deleting a Database (the irreversible button with the surprisingly casual UI path)

To delete a database from the BUI:

1. Log in to the Browser User Interface
2. Click the `Database` tab
3. Open `Actions` for the target database
4. Click `Delete`
5. Confirm the deletion

Current Oracle documentation also includes a `Force Delete` option in the confirmation dialog when needed.

Operational warning:

- Make sure you are deleting the intended database
- Make sure you understand whether backups, dependent services, or downstream users still care about it

Because "I thought that was the test database" is not a recovery strategy. It is a confession.

---

## 5. Upgrading a Database (same database, newer home, new opportunities for consequences)

Database upgrades in the BUI are tied to database homes.

To upgrade a database from the Browser User Interface:

1. Log in to the Browser User Interface
2. Click the `Database` tab
3. Open `Actions` for the target database
4. Click `Upgrade`
5. Select the destination database home version
6. Confirm the action

Important prerequisite:

- The destination database home version must already exist and be available on the appliance

That means the upgrade path is not just:

- Click Upgrade and hope

It is actually:

- Stage the correct DB home first
- Then move the database to that newer home

In classic Oracle fashion, the visible button is the easy part. The real dependency is hiding behind it with paperwork.

---

## 6. Database Homes (the layer underneath the databases that people ignore until upgrade day)

Even when the daily task is "manage a database," database homes matter because every create, upgrade, and move operation depends on them.

ODA lets you:

- View database homes
- Create database homes
- Delete unused database homes
- Associate databases with existing homes

To view database homes in the BUI:

1. Click the `Database` tab
2. Click `Database Home` in the left menu

From there, you can see:

- DB home name
- ID
- Version
- Location
- Creation time
- Databases associated with that home

Useful action:

- Open `Actions` next to a DB home and click `View Databases` to see which databases are attached to that home

To create a new database home:

1. Click the `Database` tab
2. Click `Database Home`
3. Click `Create Database Home`
4. Select the database version
5. Select the database edition: `Enterprise Edition` or `Standard Edition`
6. Click `Create`
7. Confirm the action when prompted

Important prerequisite:

- The Oracle Database Appliance RDBMS Clone file image for that version must already be in the repository

To delete a database home:

1. Click the `Database` tab
2. Click `Database Home`
3. Open `Actions` next to the target DB home
4. Click `Delete`
5. Confirm the action

Deletion rule:

- You can only delete a DB home if no databases are associated with it

This is why "upgrade the database" is really shorthand for "move the database to a newer, already-prepared database home without ruining production."

---

## 7. CLI Perspective (for people who prefer explicit commands over reassuring buttons)

The BUI is convenient, but the CLI gives you direct control.

Common database-management commands include:

- `odacli list-databases`
- `odacli create-database`
- `odacli describe-database`
- `odacli delete-database`
- `odacli list-dbhomes`
- `odacli create-dbhome`

And for job tracking:

- `odacli list-jobs`
- `odacli describe-job`

This matters because the BUI is good for guided management, but the CLI is where repeatability, automation, and serious troubleshooting start to feel less like optional hobbies.

---

## 8. Key Takeaways (the part where the appliance starts acting like a database platform instead of a rack project)

- Use the `Database` tab in the Browser User Interface to view, create, upgrade, and delete databases
- Review the database list carefully before making changes, because the UI will not save you from confidence
- Database creation depends on the required RDBMS clone files already being in the repository
- Shape, storage, version, and DB home choices all affect how the new database is provisioned
- Database upgrades depend on the destination DB home already existing on the appliance
- Database homes are not background scenery; they are central to create, move, and upgrade operations
- The BUI lets you inspect a DB home and see which databases are associated with it before you change anything
- You can create a DB home only when the required RDBMS clone image is already staged in the repository
- You can delete a DB home only when no databases are attached to it
- Use the Activity view or `odacli describe-job` to verify jobs instead of assuming success because the button was clickable

---

## 9. Wrap-Up (the appliance is now a real database platform, which means your mistakes can become very real too)

You now know how to view, create, delete, and upgrade databases on bare metal ODA, and how database homes sit underneath those actions like the quietly essential infrastructure they are. Next comes the broader operational layer, where managing databases turns into managing the environment those databases depend on, which is where the appliance starts expecting actual adult supervision.
