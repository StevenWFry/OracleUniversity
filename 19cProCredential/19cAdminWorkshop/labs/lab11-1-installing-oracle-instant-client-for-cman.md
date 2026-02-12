# Lab 11-1 - Installing Oracle Instant Client for Connection Manager

Practice 5-1 prepares Lesson 5 by installing Oracle client software components needed for Connection Manager and Oracle Net Listener.

---

## 1. Practice Goal

Perform a custom Oracle client install that includes:

- Connection Manager
- Oracle Net Listener

Then add a client entry to `oratab` for easier environment setup.

Assumption:

- Instant Client zip is available under `/stage`.

---

## 2. Unzip Client Software

As `oracle` OS user:

```bash
cd /stage
ls
unzip V982064-01.zip
```

Expected:

- Creates `/stage/client` directory.

---

## 3. Launch Oracle Client Installer

```bash
cd /stage/client
pwd
./runInstaller
```

Installer flow from transcript:

1. Installation type: `Custom`
2. Oracle base/home style path:
 - `/u01/app/oracle/product/client_1`
3. Components:
 - `Connection Manager`
 - `Oracle Net Listener`
4. Click `Install`

---

## 4. Execute Root Scripts

When installer prompts for root script execution:

1. Open root terminal.
2. Run the script path shown by installer.
3. Accept default local bin path when prompted (press Enter if default is correct).
4. Return to installer and click `OK`.

Expected result:

- Installer reports Oracle client installation successful.
- Close installer.

---

## 5. Add `oratab` Entry for Client Home

Open terminal and append client entry:

```bash
echo "client:/u01/app/oracle/product/client_1:N" >> /etc/oratab
```

Note:

- This may require elevated privileges depending on lab image permissions.

---

## 6. Lab Result

You installed the required Oracle client components for CMAN exercises and added a `client` home entry in `oratab` for easier environment switching in later labs.
