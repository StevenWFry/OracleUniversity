# Lab 6-2 (Part 2) - Initialization Parameters and the SPFILE/PFILE Search Game

And look, the instance does not start up because of vibes. It starts because it finds a parameter file and does what it is told. In this lab you watch Oracle prove just how literal it is about that.

This aligns to **Practice 6-1: Investigating Initialization Parameter Files** in the activity guide.

---

## 1. Assumptions

- You are logged in as the `oracle` OS user.
- The `orclcdb` database instance is started.

---

## 2. Locate the Default SPFILE

1. Set the environment:

   ```bash
   . oraenv
   # When prompted:
   ORACLE_SID = [orclcdb] ? orclcdb
   ```

2. Connect as SYSDBA:

   ```bash
   sqlplus / as sysdba
   ```

3. Show the SPFILE location:

   ```sql
   SHOW PARAMETER spfile;
   ```

   Expected path (formatted):

   ```text
   /u01/app/oracle/product/19.3.0/dbhome_1/dbs/spfileorclcdb.ora
   ```

   Note: SPFILE naming convention is `spfile<SID>.ora`.

---

## 3. Inspect the Sample `init.ora`

1. Drop to the OS shell:

   ```sql
   HOST
   ```

2. List the parameter directory contents:

   ```bash
   cd $ORACLE_HOME/dbs
   ls
   ```

3. View the sample `init.ora`:

   ```bash
   more init.ora
   ```

4. Return to SQL*Plus:

   ```bash
   exit
   ```

---

## 4. Create a PFILE from the SPFILE

1. Create a text PFILE:

   ```sql
   CREATE PFILE='$ORACLE_HOME/dbs/initorclcdb.ora' FROM spfile;
   ```

2. Drop to the OS shell and inspect it:

   ```sql
   HOST
   ```

   ```bash
   cd $ORACLE_HOME/dbs
   more initorclcdb.ora
   ```

3. Return to SQL*Plus:

   ```bash
   exit
   ```

---

## 5. Prove the SPFILE Search Order

1. Shut down the instance:

   ```sql
   SHUTDOWN IMMEDIATE;
   ```

2. Rename the SPFILE so Oracle cannot find it:

   ```sql
   HOST
   ```

   ```bash
   cd $ORACLE_HOME/dbs
   mv spfileorclcdb.ora spfileorclcdb.ora_original
   ```

3. Return to SQL*Plus and start up:

   ```bash
   exit
   ```

   ```sql
   STARTUP;
   ```

4. Confirm the instance is using a PFILE:

   ```sql
   SHOW PARAMETER spfile;
   ```

   The `VALUE` column should be empty.

---

## 6. Restore the SPFILE

1. Shut down:

   ```sql
   SHUTDOWN IMMEDIATE;
   ```

2. Rename the SPFILE back:

   ```sql
   HOST
   ```

   ```bash
   cd $ORACLE_HOME/dbs
   mv spfileorclcdb.ora_original spfileorclcdb.ora
   ```

3. Start the instance and verify the SPFILE is back:

   ```bash
   exit
   ```

   ```sql
   STARTUP;
   SHOW PARAMETER spfile;
   ```

---

## 7. Wrap-Up

You have now:

- Located the default SPFILE for `orclcdb`
- Inspected the sample `init.ora`
- Created a PFILE from the SPFILE
- Proven that Oracle falls back to a PFILE when the SPFILE is missing
- Restored the SPFILE and confirmed normal startup behavior
