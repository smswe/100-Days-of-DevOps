# Day 18 - Install and Configure MariaDB Server

## Task

We need to set up a database server on the **Nautilus DB Server** in **Stratos Datacenter**.

Perform the following steps:

1. Install and configure MariaDB server.
2. Create a database named `kodekloud_db4`.
3. Create a user called `kodekloud_sam` and set its password to `GyQkFRVNr3`.
4. Grant full permissions to user `kodekloud_sam` on database `kodekloud_db4`.

---

## Step 1: Connect to the Database Server

From the jump host, connect to the Nautilus DB server:

```bash
ssh peter@stdb01
```

- `ssh` – securely connects to a remote server.
- `peter` – Linux username.
- `stdb01` – database server hostname.

## Step 2: Check and Install MariaDB

Check whether MariaDB server is already installed:

```bash
rpm -q mariadb-server
```

Install MariaDB:

```bash
sudo dnf install -y mariadb-server
```

**Command explanation:**

| Command | Meaning |
|---|---|
| `dnf` | Package manager in RHEL/CentOS |
| `install` | Installs a package |
| `-y` | Automatically confirms installation |
| `mariadb-server` | MariaDB database server package |

## Step 3: Start and Enable MariaDB

```bash
sudo systemctl enable --now mariadb
```

Check service status:

```bash
sudo systemctl status mariadb
```

Expected status:

```text
Active: active (running)
```

**Difference between start and enable:**

- `start` – starts the service immediately.
- `enable` – configures the service to start automatically during system boot.
- `--now` – enables and starts the service together.

## Step 4: Access MariaDB

Log in to MariaDB:

```bash
sudo mysql
```

Alternative:

```bash
sudo mariadb
```

Expected prompt:

```text
MariaDB [(none)]>
```

This means we are now inside the MariaDB SQL shell.

## Step 5: Create the Database

```sql
CREATE DATABASE kodekloud_db4;
```

Expected output:

```text
Query OK, 1 row affected
```

**Explanation:**

- `CREATE DATABASE` creates a new database.
- `kodekloud_db4` is the database name.

A database is like a container that stores application data in tables.

## Step 6: Create the Database User

```sql
CREATE USER 'kodekloud_sam'@'%' IDENTIFIED BY 'GyQkFRVNr3';
```

Expected output:

```text
Query OK, 0 rows affected
```

**Explanation:**

- `CREATE USER` – creates a database user.
- `kodekloud_sam` – username.
- `@'%'` – allows connections from any host, subject to network and server security controls.
- `IDENTIFIED BY` – sets the user's password.

**Important:** MariaDB users are identified by both username and host. For example, `'kodekloud_sam'@'localhost'` and `'kodekloud_sam'@'%'` are different accounts.

## Step 7: Grant Full Permissions

```sql
GRANT ALL PRIVILEGES ON kodekloud_db4.* TO 'kodekloud_sam'@'%';
```

Expected output:

```text
Query OK, 0 rows affected
```

**Understanding the command:**

| Part | Meaning |
|---|---|
| `GRANT` | Gives permissions |
| `ALL PRIVILEGES` | Grants all available privileges |
| `kodekloud_db4.*` | Applies to all tables in this database |
| `TO` | Specifies the user receiving permissions |
| `'kodekloud_sam'@'%'` | Target database account |

This allows the user to perform operations such as creating, reading, updating, and deleting data in the database.

**Note:** `FLUSH PRIVILEGES` is not required after `CREATE USER` and `GRANT`, because MariaDB applies these changes directly.

## Step 8: Verify the Configuration

### 8.1 Check whether the database exists

```sql
SHOW DATABASES;
```

Look for:

```text
kodekloud_db4
```

### 8.2 Check whether the user exists

```sql
SELECT User, Host FROM mysql.user WHERE User = 'kodekloud_sam';
```

Expected account:

```text
kodekloud_sam | %
```

### 8.3 Verify user permissions

```sql
SHOW GRANTS FOR 'kodekloud_sam'@'%';
```

Expected grant should include:

```text
GRANT ALL PRIVILEGES ON `kodekloud_db4`.* TO `kodekloud_sam`@`%`
```

## Step 9: Exit MariaDB

```sql
EXIT;
```

This exits the MariaDB shell and returns to the Linux terminal.

---

## Complete Command Reference

### Linux commands

```bash
# Connect to DB server
ssh peter@stdb01

# Check MariaDB installation
rpm -q mariadb-server

# Install MariaDB
sudo dnf install -y mariadb-server

# Start and enable MariaDB
sudo systemctl enable --now mariadb

# Check service status
sudo systemctl status mariadb

# Access MariaDB
sudo mysql
```

### SQL commands

```sql
-- Create database
CREATE DATABASE kodekloud_db4;

-- Create user
CREATE USER 'kodekloud_sam'@'%' IDENTIFIED BY 'GyQkFRVNr3';

-- Grant full permissions
GRANT ALL PRIVILEGES ON kodekloud_db4.* TO 'kodekloud_sam'@'%';

-- Verify database
SHOW DATABASES;

-- Verify user
SELECT User, Host FROM mysql.user WHERE User = 'kodekloud_sam';

-- Verify permissions
SHOW GRANTS FOR 'kodekloud_sam'@'%';

-- Exit
EXIT;
```

---

## Important Learning Points

| Concept | What I Learned |
|---|---|
| MariaDB | Open-source relational database management system |
| `dnf` | Installs software packages on CentOS/RHEL |
| `systemctl` | Manages Linux services |
| `CREATE DATABASE` | Creates a database |
| `CREATE USER` | Creates a database account |
| `GRANT` | Assigns permissions to a user |
| `SHOW DATABASES` | Lists available databases |
| `SHOW GRANTS` | Displays permissions assigned to a user |

## PostgreSQL vs MariaDB

| PostgreSQL | MariaDB |
|---|---|
| `psql` | `mysql` or `mariadb` |
| `postgresql` service | `mariadb` service |
| `CREATE USER` | `CREATE USER` |
| `CREATE DATABASE` | `CREATE DATABASE` |
| `GRANT ALL PRIVILEGES ON DATABASE ...` | `GRANT ALL PRIVILEGES ON database.* ...` |

Both are relational database systems, but their administration commands and permission models differ.

---

## Final Outcome

- [x] MariaDB server installation/configuration commands prepared.
- [x] Database creation command prepared.
- [x] User creation command prepared.
- [x] Full database permissions command prepared.
- [x] Verification commands included.

**Status:** Mark as Completed after the KodeKloud lab validates successfully.
