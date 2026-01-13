## Lesson 1 Lab - Getting Set Up in SQL Developer (in which we make friends with icons and passwords)

Welcome to Practice 1, where the goal is to get SQL Developer open, your connections working, and your preferences configured without accidentally launching a different decade.

You will:

- Start SQL Developer
- Create a database connection
- Browse the HR schema tables
- Set SQL Developer preferences
- Locate the course scripts

---

## 1. Find the Activity Guide (the lab map)

Your Activity Guide is typically under your Home directory in `Labs/Activity Guide`. You may see a `Student Guide` folder instead. Inside are the PDFs you downloaded.

Open the **Security Credentials** PDF first. It contains the usernames and passwords for this course.

Accounts used:

- `ORA41` (first two days)
- `ORA61` (last three days)
- `ORA62` (optional extra practices)

All three usually share the same password. Look around page 5 or 6 of the Activity Guide for the credentials.

---

## 2. Start SQL Developer

- Start SQL Developer from the desktop icon (double-click or right-click > Open).
- Yes, I will call it SQL. I know that is not technically precise. I refuse to feel bad about it.

---

## 3. Create the First Connection (MyConnection)

Right-click **Oracle Connections** and choose **New Connection**.

Use these settings:

- **Connection Name**: `MyConnection` (case matters)
- **Username**: `ORA41`
- **Password**: use the password from the Security Credentials PDF
- Check **Save Password**
- **Service Name**: `ORCLPDB1`

Click **Test** and confirm the status is **Success**.

Click **Save**, then **Connect**.

---

## 4. Browse the HR Tables

- Expand your `MyConnection` in the left panel.
- Expand **Tables**.
- Click **EMPLOYEES** once to see the table structure.
- Click the **Data** tab to view the rows.

This is your first look at the data you will be living with for the rest of the course. Say hello.

---

## 5. Run a Simple Query (and meet the two run buttons)

Open the SQL Worksheet for `MyConnection` and run:

```sql
SELECT last_name, salary
FROM employees
WHERE salary * 12 > 10000;
```

Run it two ways:

- **Run Statement** (`F9`) - interactive grid output
- **Run Script** (`F5`) - script-style output

Recommendation: use **Run Script** for PL/SQL blocks, because it handles multi-line scripts more predictably.

---

## 6. Set Preferences (so the font is readable by humans)

Go to **Tools > Preferences > Code Editor > Fonts** and increase the font size (18 is a popular survival choice).

Then go to **Database > Worksheet Parameters** and set:

- **Default path to look for scripts**: `home/oracle/labs/plsf`

This directory contains:

- **Code Examples**
- **Labs**
- **Solutions**

---

## 7. Explore the Script Folders

From SQL Developer:

- **File > Open**
- Navigate to `home/oracle/labs/plsf`
- Explore **Code Examples**, **Labs**, and **Solutions**

Open a sample solution (for example, Lesson 2) to confirm you can view a script without executing it.

---

## 8. Clean Up

Close any open worksheets and object tabs so your workspace looks tidy for the next lesson.

Congratulations. You now have connections, preferences, and a functioning lab environment. This is the boring part, but it is the foundation upon which all future chaos will be built.