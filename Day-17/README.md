# Day 17 - Configure PostgreSQL Database, User and Permissions

## Task

The Nautilus application development team is planning to deploy a new application on Nautilus infrastructure in Stratos Datacenter.

The application uses PostgreSQL. PostgreSQL is already installed on the Nautilus DB Server.

Perform the following tasks:

1. Create a database user `kodekloud_gem`.
2. Set the user's password to `BruCStnMT5`.
3. Create a database named `kodekloud_db1`.
4. Grant full database-level privileges to `kodekloud_gem` on `kodekloud_db1`.

**Important:** Do not restart the PostgreSQL service.

---

## Server Details

| Component | Details |
|---|---|
| Jump Host | `thor@jump-host` |
| Database Server | `peter@stdb01` |
| Database Software | PostgreSQL |
| PostgreSQL Version | 13.23 |
| Service Name | `postgresql` |
| Database User | `kodekloud_gem` |
| Database Name | `kodekloud_db1` |
| Required Password | `BruCStnMT5` |

## Step 1 – Connect to the Database Server

From the jump host:

```bash
ssh peter@stdb01
```

- `ssh` – securely connects to a remote server.
- `peter` – Linux username.
- `stdb01` – database server hostname.

When connecting for the first time, SSH may ask you to verify the server fingerprint. Type `yes` to accept it if it is the expected lab server.

## Step 2 – Check PostgreSQL Service Status

```bash
sudo systemctl status postgresql
```

Observed output:

```text
postgresql.service - PostgreSQL database server
Loaded: loaded (/usr/lib/systemd/system/postgresql.service)
Active: active (running)
Main PID: 15715 (postmaster)
```

**Explanation:**

- `systemctl` – manages Linux services.
- `status` – checks the current service status.
- `postgresql` – PostgreSQL service name.

The service was already running.

**Important:** The task explicitly says not to restart PostgreSQL. Since the service was already active, no restart was required.

## Step 3 – Switch to the PostgreSQL Linux User

```bash
sudo su - postgres
```

**Explanation:**

- `sudo` – executes a command with elevated privileges.
- `su` – switches to another Linux user.
- `-` – loads the target user's login environment.
- `postgres` – default Linux account used to administer PostgreSQL.

The prompt changes to:

```text
[postgres@stdb01 ~]$
```

## Step 4 – Open PostgreSQL Shell

```bash
psql
```

Observed output:

```text
psql (13.23)
Type "help" for help.

postgres=#
```

**Explanation:**

- `psql` – PostgreSQL command-line client.
- `postgres=#` – PostgreSQL shell prompt.
- `#` – indicates a PostgreSQL superuser prompt.

We can now execute SQL commands to manage databases and users.

## Step 5 – Create the Database User

```sql
CREATE USER kodekloud_gem WITH PASSWORD 'BruCStnMT5';
```

Expected output:

```text
CREATE ROLE
```

**Explanation:**

| Part | Meaning |
|---|---|
| `CREATE USER` | Creates a PostgreSQL user |
| `kodekloud_gem` | Username |
| `WITH PASSWORD` | Sets the password |
| `'BruCStnMT5'` | Required password |

**Important PostgreSQL concept:**

In PostgreSQL, users are technically roles with the `LOGIN` attribute. Therefore, creating a user also creates a role.

## Step 6 – Create the Database

```sql
CREATE DATABASE kodekloud_db1;
```

Expected output:

```text
CREATE DATABASE
```

**Explanation:**

- `CREATE DATABASE` – creates a new database.
- `kodekloud_db1` – database name.

The database is created with the PostgreSQL administrative user as its owner by default, unless another owner is specified.

## Step 7 – Grant Database Permissions

```sql
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db1 TO kodekloud_gem;
```

Expected output:

```text
GRANT
```

**Explanation:**

| Part | Meaning |
|---|---|
| `GRANT` | Assigns permissions |
| `ALL PRIVILEGES` | Grants all available database-level privileges through this command |
| `ON DATABASE` | Specifies the database |
| `kodekloud_db1` | Target database |
| `TO` | Specifies the recipient |
| `kodekloud_gem` | User receiving permissions |

This grants database-level privileges such as:

- CONNECT – connect to the database.
- CREATE – create schemas.
- TEMPORARY – create temporary tables.

**Important:** PostgreSQL database-level privileges do not automatically grant all privileges on tables and sequences inside the database. For this KodeKloud task, the expected command is the `GRANT ALL PRIVILEGES ON DATABASE` statement above.

## Step 8 – Verify the Database

Inside PostgreSQL, run:

```sql
\l
```

`\l` lists all databases.

Observed result included:

```text
Name          | Owner
--------------+----------
kodekloud_db1 | postgres
postgres      | postgres
template0     | postgres
template1     | postgres
```

The database `kodekloud_db1` was successfully created.

The access privileges included:

```text
kodekloud_gem=CTc/postgres
```

This confirms the user received the database-level privileges.

## Step 9 – Verify the User

Run:

```sql
\du
```

`\du` lists PostgreSQL roles and their attributes.

Observed output:

```text
Role name    | Attributes
-------------+-----------
kodekloud_gem|
postgres     | Superuser, Create role, Create DB, Replication, Bypass RLS
```

This confirms that `kodekloud_gem` exists.

## Step 10 – Password Typo Troubleshooting

During the first attempt, the password was entered incorrectly.

Incorrect command:

```sql
CREATE USER kodekloud_gem WITH PASSWORD 'BruCStnMTS';
```

Required password:

```text
BruCStnMT5
```

**Difference:** The last character was entered as the letter `S` instead of the number `5`.

The user and database had already been created successfully, so there was no need to drop and recreate them.

Correct the password using:

```sql
ALTER USER kodekloud_gem WITH PASSWORD 'BruCStnMT5';
```

Expected output:

```text
ALTER ROLE
```

**Explanation:**

- `ALTER USER` – modifies an existing PostgreSQL user.
- `WITH PASSWORD` – updates the password.

This is a useful recovery command when a user exists but the password is incorrect.

## Step 11 – Exit PostgreSQL

```sql
\q
```

This exits the PostgreSQL shell.

Then return from the Linux `postgres` account:

```bash
exit
```

**Important:** Linux commands are case-sensitive. Use lowercase `exit`, not uppercase `EXIT`.

---

## Complete Command Reference

### Linux Commands

```bash
# Connect to database server
ssh peter@stdb01

# Check PostgreSQL service
sudo systemctl status postgresql

# Switch to PostgreSQL Linux account
sudo su - postgres

# Open PostgreSQL terminal
psql
```

### PostgreSQL SQL Commands

```sql
-- Create database user
CREATE USER kodekloud_gem WITH PASSWORD 'BruCStnMT5';

-- Create database
CREATE DATABASE kodekloud_db1;

-- Grant database-level privileges
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db1 TO kodekloud_gem;

-- List databases
\l

-- List users and roles
\du

-- Correct password if required
ALTER USER kodekloud_gem WITH PASSWORD 'BruCStnMT5';

-- Exit PostgreSQL
\q
```

### Return to Linux Shell

```bash
exit
```

---

## Important Learning Points

| Command | Purpose |
|---|---|
| `systemctl status` | Checks service status |
| `sudo su - postgres` | Switches to PostgreSQL Linux account |
| `psql` | Opens PostgreSQL command-line interface |
| `CREATE USER` | Creates a PostgreSQL login role |
| `CREATE DATABASE` | Creates a database |
| `GRANT` | Assigns database privileges |
| `ALTER USER` | Modifies an existing user |
| `\l` | Lists databases |
| `\du` | Lists roles/users |
| `\q` | Exits PostgreSQL |

## PostgreSQL vs MariaDB

| PostgreSQL | MariaDB |
|---|---|
| `psql` | `mysql` / `mariadb` |
| `postgresql` service | `mariadb` service |
| `\l` lists databases | `SHOW DATABASES;` |
| `\du` lists roles | `SELECT User, Host FROM mysql.user;` |
| `GRANT ALL PRIVILEGES ON DATABASE db TO user;` | `GRANT ALL PRIVILEGES ON db.* TO 'user'@'%';` |

Both are relational database management systems, but their administration commands and permission models are different.

---

## Final Checklist

- [x] PostgreSQL service checked and confirmed running.
- [x] Database user `kodekloud_gem` created.
- [x] Password corrected to `BruCStnMT5`.
- [x] Database `kodekloud_db1` created.
- [x] Database-level privileges granted.
- [x] User verified using `\du`.
- [x] Database verified using `\l`.
- [x] PostgreSQL service was not restarted.

**Status: Completed – KodeKloud task requirements satisfied after correcting the password.**
