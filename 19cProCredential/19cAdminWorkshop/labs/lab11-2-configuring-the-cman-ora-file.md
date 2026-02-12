# Lab 11-2 - Configuring the `cman.ora` File

Practice 5-2 configures Oracle Connection Manager on the client software home host, including listener endpoint, logging paths, and startup validation.

---

## 1. Practice Goal

Configure `cman.ora` with:

- CMAN listening endpoint
- access control/rule framework values
- logging and tracing directories

Then start CMAN using `cmctl`.

Important:

- In this lab, `ORACLE_HOME` must point to the **client** home, not the database home.

---

## 2. Set Environment to Client Home

Open terminal and set environment:

```bash
. oraenv
```

When prompted, enter:

```text
client
```

---

## 3. Copy Sample `cman.ora`

From network admin directory under client home, copy sample file:

```bash
cd $ORACLE_HOME/network/admin
cp samples/cman.ora cman.ora
ls
```

Expected:

- `cman.ora` exists in current directory.

---

## 4. Edit `cman.ora`

Open file (example editor used in transcript: `gedit`):

```bash
gedit cman.ora &
```

Apply required changes exactly:

- replace host values from FQ host to `localhost`
- set listener port to `1522`
- set `LOG_DIRECTORY` to `/home/oracle/logs`
- set `TRACE_DIRECTORY` to `/home/oracle/trace`

Transcript warning:

- Preserve syntax/formatting carefully.
- Typos and accidental spacing/format edits can prevent CMAN startup.

Save and close file.

---

## 5. Create Required Directories

Create logging and tracing directories:

```bash
mkdir -p /home/oracle/logs /home/oracle/trace
```

---

## 6. Start CMAN with `cmctl`

Run Connection Manager control utility and connect admin session:

```bash
cmctl
```

In `CMCTL>` prompt:

```text
ADMINISTER CMAN_LOCALHOST
STARTUP
```

Expected:

- startup completes successfully.

Exit utility:

```text
EXIT
```

---

## 7. Lab Result

You configured `cman.ora` in the client home, created required log/trace paths, and started `CMAN_LOCALHOST` successfully via `cmctl`.
