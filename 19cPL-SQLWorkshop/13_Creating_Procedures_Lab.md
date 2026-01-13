## Lesson 11 Lab - Creating Procedures (Part 1: New Connection)

This practice resets your SQL Developer connection and sets a script path for the rest of the unit.

You will:

- Delete the old connection
- Create a new connection for `ora61`
- Set the script path to `home/oracle/labs/plpu`

---

## 1. Delete the Old Connection

- Right-click **My Connection**
- Choose **Delete**
- Confirm the deletion

---

## 2. Create the New Connection

Create a new connection with:

- **Connection Name**: `My Connection`
- **Username**: `ora61`
- **Password**: `cloud_4U` (verify in Activity Guide)
- **Service Name**: `orclpdb1`
- Check **Save Password**

Click **Test** and confirm **Status: Success**.

Click **Save**, then **Connect**.

---

## 3. Set the Script Path

Go to:

- **Tools > Preferences > Database > Worksheet**

Set **Default path to look for scripts** to:

- `home/oracle/labs/plpu`

Use **Browse** if typing the path is inconvenient.

---

## 4. Confirm

Expand the connection and confirm the HR schema tables appear.

---

## Wrap-Up

Connection reset complete. You are ready for Practice 11-2 (procedures with parameters and exceptions).