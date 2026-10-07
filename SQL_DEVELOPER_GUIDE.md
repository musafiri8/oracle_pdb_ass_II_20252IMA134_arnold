# 🛠️ SQL Developer Step-by-Step Guide

> **Author:** MUSAFIRI Arnold | **Student ID:** 20252IMA134  
> **Tool:** Oracle SQL Developer (Version 26.2.0.186.2220)

This guide provides the exact step-by-step instructions to complete all Oracle PDB tasks using Oracle SQL Developer.

---

## 📋 Prerequisites

Before starting, ensure you have:
- Oracle SQL Developer installed and running
- Access to an Oracle Database with CDB (Container Database) configured
- `SYS` credentials with `SYSDBA` privileges

---

## 🔗 Step 0: Connect to the CDB Root

1. Open **Oracle SQL Developer**
2. Click the **green `+`** icon (New Connection)
3. Fill in the connection details:

| Field              | Value                                    |
|--------------------|------------------------------------------|
| **Connection Name** | CDB_Root_Connection                     |
| **Username**       | SYS                                      |
| **Password**       | (your SYS password)                      |
| **Role**           | SYSDBA                                   |
| **Hostname**       | localhost (or your DB server)            |
| **Port**           | 1521                                     |
| **Service Name**   | (your CDB service, e.g., ORCL or XEPDB) |

4. Click **Test** → should show "Status: Success"
5. Click **Connect**

### Verify You Are in the CDB Root

Run this in the **SQL Worksheet** (press **F5** to run as script):

```sql
-- Confirm you are in the root container
SELECT SYS_CONTEXT('USERENV', 'CON_NAME') AS current_container FROM dual;
```

Expected result: `CDB$ROOT`

```sql
-- Check if Oracle Managed Files is configured
SELECT value AS db_create_file_dest FROM v$parameter WHERE name = 'db_create_file_dest';
```

> ⚠️ If `db_create_file_dest` is empty (NULL), you will need to use `FILE_NAME_CONVERT` when creating PDBs. Ask your lab administrator for the correct paths. If it has a value, you can proceed as-is.

---

## ✅ Task 1: Create PDB `ar_pdb_20252IMA134`

### Step 1.1: Create the Pluggable Database

Run from the **CDB Root** connection:

```sql
CREATE PLUGGABLE DATABASE ar_pdb_20252IMA134
  ADMIN USER pdbadmin IDENTIFIED BY "YourChosenPassword";
```

> 📸 **Screenshot 1:** Capture the output showing "Pluggable database created."

### Step 1.2: Open the PDB

```sql
ALTER PLUGGABLE DATABASE ar_pdb_20252IMA134 OPEN;
```

### Step 1.3: Save the PDB State (auto-open on DB restart)

```sql
ALTER PLUGGABLE DATABASE ar_pdb_20252IMA134 SAVE STATE;
```

### Step 1.4: Verify PDB is Open

```sql
SELECT name, open_mode FROM v$pdbs WHERE name = 'AR_PDB_20252IMA134';
```

Expected output:

| NAME               | OPEN_MODE  |
|--------------------|------------|
| AR_PDB_20252IMA134 | READ WRITE |

> 📸 **Screenshot 2:** Capture this query result showing PDB is in READ WRITE mode.

### Step 1.5: Switch Session into the PDB

```sql
ALTER SESSION SET CONTAINER = ar_pdb_20252IMA134;
```

### Step 1.6: Create Your User Account

```sql
CREATE USER arnold_plsqlauca_20252IMA134
  IDENTIFIED BY "YourChosenPassword";
```

### Step 1.7: Grant Privileges

```sql
GRANT CREATE SESSION, CREATE TABLE, CREATE VIEW, CREATE SEQUENCE, CREATE PROCEDURE
  TO arnold_plsqlauca_20252IMA134;

ALTER USER arnold_plsqlauca_20252IMA134 QUOTA UNLIMITED ON USERS;
```

> 💡 If you get an error about the `USERS` tablespace not existing, run:
> ```sql
> SELECT tablespace_name FROM dba_tablespaces;
> ```
> Then use one of the available tablespaces in the `ALTER USER ... QUOTA UNLIMITED ON <tablespace>;` command.

### Step 1.8: Verify User Creation

```sql
SELECT username, account_status FROM dba_users
WHERE username = 'ARNOLD_PLSQLAUCA_20252IMA134';
```

Expected output:

| USERNAME                       | ACCOUNT_STATUS |
|--------------------------------|----------------|
| ARNOLD_PLSQLAUCA_20252IMA134  | OPEN           |

> 📸 **Screenshot 3:** Capture this result showing the user is created and OPEN.

### Step 1.9: Create a New Connection for Your PDB User (Optional but Recommended)

1. In SQL Developer, create a **new connection**:

| Field              | Value                              |
|--------------------|------------------------------------|
| **Connection Name** | Arnold_PDB_Connection             |
| **Username**       | arnold_plsqlauca_20252IMA134      |
| **Password**       | (the password you chose)           |
| **Role**           | default                            |
| **Hostname**       | localhost                           |
| **Port**           | 1521                                |
| **Service Name**   | ar_pdb_20252IMA134                 |

2. Click **Test** → should show "Status: Success"

---

## ✅ Task 2: Create and Delete Temporary PDB `ar_to_delete_pdb_20252IMA134`

> ⚠️ First, switch back to the CDB Root! Run:
> ```sql
> ALTER SESSION SET CONTAINER = CDB$ROOT;
> ```
> Or simply use your original CDB Root connection.

### Step 2.1: Create the Temporary PDB

```sql
CREATE PLUGGABLE DATABASE ar_to_delete_pdb_20252IMA134
  ADMIN USER tempadmin IDENTIFIED BY "TempPassword123";
```

> 📸 **Screenshot 4:** Capture the output showing "Pluggable database created."

### Step 2.2: Verify the PDB Exists

```sql
SELECT name, open_mode FROM v$pdbs
WHERE name = 'AR_TO_DELETE_PDB_20252IMA134';
```

Expected output:

| NAME                          | OPEN_MODE |
|-------------------------------|-----------|
| AR_TO_DELETE_PDB_20252IMA134  | MOUNTED   |

> 📸 **Screenshot 5:** Capture this query result confirming the PDB exists.

### Step 2.3: Close the PDB (Required Before Dropping)

```sql
ALTER PLUGGABLE DATABASE ar_to_delete_pdb_20252IMA134 CLOSE IMMEDIATE;
```

> 💡 If the PDB is already in MOUNTED state, this step may not be needed, but it's safe to run.

### Step 2.4: Drop the PDB Including Datafiles

```sql
DROP PLUGGABLE DATABASE ar_to_delete_pdb_20252IMA134 INCLUDING DATAFILES;
```

> 📸 **Screenshot 6:** Capture the output showing "Pluggable database dropped."

### Step 2.5: Confirm the PDB No Longer Exists

```sql
SELECT name FROM v$pdbs WHERE name = 'AR_TO_DELETE_PDB_20252IMA134';
```

Expected output: **no rows selected**

> 📸 **Screenshot 7:** Capture this result showing 0 rows returned — the PDB is gone.

---

## ✅ Task 3: Oracle Enterprise Manager (OEM)

### Accessing OEM Database Express

1. **Find the OEM URL** — run this from your CDB Root connection:

```sql
SELECT DBMS_XDB_CONFIG.GETHTTPSPORT() AS oem_https_port FROM dual;
```

If the result is `5500`, then your OEM URL is: **https://localhost:5500/em**

> 💡 If the port is `0`, OEM Express may not be configured. Enable it with:
> ```sql
> EXEC DBMS_XDB_CONFIG.SETHTTPSPORT(5500);
> ```

2. **Open a web browser** and go to: `https://localhost:5500/em`
3. **Accept** the self-signed certificate warning
4. **Log in** with:
   - **Username:** SYS
   - **Password:** (your SYS password)
   - **As:** SYSDBA

5. Navigate the dashboard to see:
   - Database status and performance
   - PDB containers (including `ar_pdb_20252IMA134`)

> 📸 **Screenshot 8:** Capture the OEM dashboard with your username visible.

---

## 📸 Screenshot Naming Suggestions

Save your screenshots in the correct folders:

```
screenshots/
├── pdb_creation/
│   ├── 01_pdb_creation_command.png
│   ├── 02_pdb_open_state.png
│   └── 03_user_created.png
├── pdb_deletion/
│   ├── 01_temp_pdb_created.png
│   ├── 02_temp_pdb_verified.png
│   ├── 03_temp_pdb_dropped.png
│   └── 04_temp_pdb_confirmed_deleted.png
└── oem_dashboard/
    └── 01_oem_dashboard.png
```

---

## 🔄 Quick Reference: Switching Containers

```sql
-- Go back to CDB Root
ALTER SESSION SET CONTAINER = CDB$ROOT;

-- Switch into your PDB
ALTER SESSION SET CONTAINER = ar_pdb_20252IMA134;

-- Check which container you are in
SELECT SYS_CONTEXT('USERENV', 'CON_NAME') AS current_container FROM dual;

-- List all PDBs
SELECT name, open_mode FROM v$pdbs;
```

---

## ⚠️ Common Errors & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `ORA-65005: missing or invalid file name pattern` | OMF not configured | Set `db_create_file_dest` or use `FILE_NAME_CONVERT` |
| `ORA-65019: pluggable database ... already open` | PDB is already open | Skip the OPEN command or close first |
| `ORA-65020: pluggable database ... already closed` | PDB already closed | Skip the CLOSE command |
| `ORA-01109: database not open` | Trying to create user in unopened PDB | Open the PDB first with `ALTER PLUGGABLE DATABASE ... OPEN` |
| `ORA-01017: invalid username/password` | Wrong credentials | Double-check password; Oracle is case-sensitive for quoted passwords |

---

> 💡 **Tip:** Always use **F5 (Run Script)** instead of **Ctrl+Enter (Run Statement)** for DDL commands in SQL Developer. This ensures you see the full output.
