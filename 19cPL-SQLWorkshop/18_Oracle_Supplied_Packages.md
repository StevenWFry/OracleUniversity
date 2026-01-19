## Lesson 18 - Using Oracle-Supplied Packages (in which Oracle does the heavy lifting)

Welcome to Lesson 16. This is where we use Oracle's built-in packages instead of reinventing the wheel. We'll focus on DBMS_OUTPUT, UTL_FILE, UTL_MAIL, and a brief intro to JSON structures.

By the end of this lesson, you should be able to:

- Describe how DBMS_OUTPUT works
- Use UTL_FILE to read/write OS files
- Describe UTL_MAIL features
- Describe JSON data structures

---

## 0. Where This Lesson Fits (Student Guide Lesson 16: Oracle-supplied packages)

In the Student Guide, this maps to **Lesson 16: Using Oracle-Supplied Packages in Application Development**. The agenda there walks through:

- What Oracle-supplied packages are and why to use them
- `DBMS_OUTPUT` basics
- `UTL_FILE` for file I/O (including exceptions)
- `UTL_MAIL` for sending email after DBA setup

This chapter follows that same outline, and your lab is the practical side of **Activity Guide Practice 16**, where you generate a text file report with `UTL_FILE` listing employees whose salary beats their departmental average.

---

## 1. Oracle-Supplied Packages (the built-in toolbox)

Oracle ships with 200+ packages. They extend database functionality and are installed during database creation. You should **not** modify them.

Common examples:

- `DBMS_OUTPUT`
- `UTL_FILE`
- `UTL_MAIL`

---

## 2. DBMS_OUTPUT

DBMS_OUTPUT writes text to a buffer. Messages appear **after** the calling subprogram finishes.

Key procedures:

- `PUT`
- `PUT_LINE`
- `GET_LINE`
- `GET_LINES`

---

## 3. UTL_FILE (writing OS files)

UTL_FILE lets PL/SQL read and write text files on the server.

Steps:

1) Create a directory object
2) Grant permissions
3) Use `UTL_FILE.FOPEN` to open files

Example directory:

```sql
CREATE OR REPLACE DIRECTORY reports_dir AS '/home/oracle/labs/plpu/reports';
```

Example procedure outline:

```sql
f_file := UTL_FILE.FOPEN('REPORTS_DIR', 'sal_report.txt', 'W');
UTL_FILE.PUT_LINE(f_file, 'Report generated...');
UTL_FILE.FCLOSE(f_file);
```

Common exceptions:

- `INVALID_PATH`
- `INVALID_MODE`
- `INVALID_FILEHANDLE`
- `READ_ERROR`
- `WRITE_ERROR`

---

## 4. UTL_MAIL (sending email)

UTL_MAIL allows PL/SQL to send email. Requires DBA setup:

- Install package spec and body
- Set `SMTP_OUT_SERVER`
- Grant privileges to users

Available procedures:

- `SEND`
- `SEND_ATTACH_RAW`
- `SEND_ATTACH_VARCHAR2`

Example:

```sql
UTL_MAIL.SEND(
  sender     => 'hr@company.com',
  recipients => 'user@company.com',
  subject    => 'Report',
  message    => 'See attached.'
);
```

---

## 5. JSON (quick intro)

Oracle supports JSON structures in PL/SQL, used for structured data exchange. More details appear later in the course.

---

## 6. Wrap-Up

You now know how to:

- Use DBMS_OUTPUT for messaging
- Write OS files with UTL_FILE
- Send mail using UTL_MAIL
- Understand where JSON fits

Next up: **Practice 16**, where you generate a report using UTL_FILE.
