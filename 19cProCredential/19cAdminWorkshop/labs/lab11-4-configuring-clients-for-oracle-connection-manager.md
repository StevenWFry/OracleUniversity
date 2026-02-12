# Lab 11-4 - Configuring Clients for Oracle Connection Manager

Practice 5-4 configures a client net service to route through Oracle Connection Manager and verifies connectivity.

---

## 1. Practice Goal

Create a client-side net service alias that points to CMAN endpoint (`localhost:1522`) for `orclcdb`.

---

## 2. Set Client Environment

On client host terminal:

```bash
. oraenv
```

When prompted, enter:

```text
client
```

---

## 3. Create Net Service in Net Manager

Start Net Manager:

```bash
netmgr
```

In the navigator:

1. Select `Service Naming`
2. Click `+` (Add)
3. Net service name: `C_ORCLCDB`
4. Protocol: `TCP/IP`
5. Hostname: `localhost`
6. Port: `1522`
7. Service name: `orclcdb`
8. Click `Finish`

Then save:

- `File` -> `Save Network Configuration`

Close Net Manager.

---

## 4. Test Connection Through CMAN

Open terminal and connect via new alias:

```bash
sqlplus system@C_ORCLCDB
```

Password:

```text
cloud_4U
```

Expected:

- Successful login through Oracle Connection Manager endpoint.

---

## 5. Lab Result

Client routing through CMAN is configured and validated using alias `C_ORCLCDB`.
