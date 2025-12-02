## Lesson 0 Lab – Getting Your Tools to Actually Work (Course Introduction Practice)

And look, before you can write a single heroic `SELECT` statement, you have to survive the most dangerous part of any course: **finding the files and logging in**. This lab is about exactly that.

You will:

- Find the **Activity Guide** and know which PDF is which.
- Locate the **SQL labs** folders (`SQL1` and `SQL2`).
- Use the correct **usernames and passwords** for each set of practices.
- Create a database connection in **SQL Developer**.
- Browse the **HR tables** and peek at their structure and data.
- Tweak SQL Developer preferences so the text isn’t microscopic.

Nothing “advanced” happens here. It’s just all the stuff that, if you get wrong, makes everything *later* look broken.

---

## 1. Finding the Activity Guide (the PDF that actually matters)

1. In your course environment, open the **Home** folder.
2. Navigate to the `ekd` folder.
3. Inside, open the folder with your **course code** (for example, something like `D108644GC10`).
4. You’ll see several PDFs:
   - `...sg1.pdf`, `...sg2.pdf` – **student guides** (lecture material).
   - `..._ag.pdf` – the **Activity Guide**. This is the lab bible.
5. Right‑click the Activity Guide (`*_ag.pdf`) and open it with **Document Viewer**.

If you can’t find or open this guide, stop. Every lab, solution, and connection detail lives in here.

---

## 2. Finding the Lab Folders (SQL1 vs SQL2)

Next, head back to the **Home** directory and locate the `Labs` folder.

Inside `Labs` you’ll see:

- `SQL1` – used for **practices for lessons/chapters 1–11** (small HR data set).
- `SQL2` – used for **practices for lessons 12–20** (larger HR data set).

Rule of thumb:

- If you’re working in early chapters, you live in `SQL1`.
- When the Activity Guide says you’re on lesson 12 or later, move to `SQL2`.

Using the wrong folder is the SQL equivalent of reheating last week’s coffee: it technically works, but everything tastes wrong.

---

## 3. Lab Usernames and Passwords (the part everyone forgets)

In the Activity Guide, navigate to the **Course Practice Environment / Security Credentials** page (around page 7).

You’ll see credentials like:

- For **practices 1–11**:
  - Username: `ora1`
  - Password: `ora1`
- For **practices 12–20**:
  - Username: `ora21` or `ora22`
  - Password: matches the username (`ora21` or `ora22`)

Important:

- Passwords are **case‑sensitive**.
- The guide also shows you the recommended **connection name** in SQL Developer (for example, `myconnection`). Use it so your screen looks like the screenshots.

---

## 4. Opening SQL Developer and the Welcome Page

1. On the desktop, find the **SQL Developer** icon.
2. Double‑click it (or right‑click → *Open*).
3. SQL Developer will take over the screen. Resize it so it shares space nicely with the Activity Guide (each on half the screen).

The **Start/Welcome page**:

- If it doesn’t show automatically, you can open it from `Help` → **Start Page**.
- It offers links to:
  - Recent / auto‑detected connections
  - Resources and tools

You’re free to explore, but most of the time you’ll close it to reclaim screen space and focus on the **Connections** pane and the **SQL Worksheet**.

---

## 5. Creating Your First Connection in SQL Developer

Now for the part where we find out if your database is actually alive.

1. In the **Connections** panel, right‑click on **Oracle Connections** and choose **New Connection...**  
   - Or click the green `+` icon in the toolbar.

2. Fill in the connection details (as given in the Activity Guide):

   - **Connection Name**: `myconnection` (or whatever the guide specifies).
   - **Username**: `ora1`
   - **Password**: `ora1`
   - Check **Save Password**.
   - Connection type: **Basic**.
   - **Hostname / Port**: use the values provided by your lab (often defaults are prefilled).
   - **Service Name**: `pdborcl`

3. Click **Test**.
   - If the **Status** shows `Success`, celebrate quietly.
   - If it fails, re‑check username, password, service name, and typos.

4. Click **Save**, then **Connect**.

The connection icon will change:

- Before connect: a plain database cylinder.
- After connect: database cylinder with a **plug** icon → you have an active session.

If you don’t see the plug icon, you’re not actually connected, no matter how optimistic you feel.

---

## 6. Browsing the HR Tables

With `myconnection` open and connected:

1. Expand your connection in the **Connections** tree.
2. Expand the **Tables** node.
3. You should see the HR tables listed (EMPLOYEES, DEPARTMENTS, JOBS, etc.).

### 6.1 Inspect the EMPLOYEES table structure

1. Click the `EMPLOYEES` table.
2. In the main pane, open the **Columns** (or **Structure**) tab.
3. Review:
   - Column names.
   - Data types.
   - Whether each column allows `NULL`.

This is where you confirm that `EMPLOYEE_ID` really is the primary key and not just something that “looks important.”

### 6.2 View data in the DEPARTMENTS table

1. Click the `DEPARTMENTS` table.
2. Switch to the **Data** tab.
3. Scroll through the rows.

You should see department IDs, names, and any foreign key links (like location IDs). This is the easiest way to check “what’s actually in this table?” without writing a single query.

---

## 7. Making SQL Developer Usable (fonts and line numbers)

By default, SQL Developer sometimes behaves like it’s being displayed on a billboard two miles away. Let’s fix that.

1. Go to **Tools → Preferences**.
2. Expand **Code Editor** → **Fonts**.
3. Increase the font size (for example, to **16**).  
   - Check the sample text to see if your eyes relax.
4. Still in Code Editor, choose **Line Gutter** and:
   - Enable **Show line numbers**.
5. Click **OK**.

You should now see:

- Larger text in the SQL Worksheet.
- Line numbers down the left side.

This makes it much easier to follow along with examples that say “Look at line 3” instead of “the third squint‑inducing character from the left.”

---

## 8. What This Lab Was Really About

By finishing this lab, you should now be able to:

- Open the **Activity Guide** and find:
  - Practice descriptions
  - Lab credentials
  - Screenshots and solutions
- Know when to use **SQL1** vs **SQL2** lab folders.
- Use the correct **lab accounts** (`ora1` for early chapters, `ora21/ora22` later).
- Create and test a **SQL Developer connection** to `pdborcl`.
- Browse table structures and data in the **HR schema**.
- Adjust **editor fonts** and show **line numbers** so future exercises are readable.

In short: your environment now works, your tools are set up, and you’re no longer fighting the UI. Which means starting in the next lesson, you can finally focus on what SQL is doing instead of where the **_ag.pdf` went.
