# Day 09 - Troubleshooting and Starting MariaDB Service

## Task Description

A critical issue was reported in the Nautilus application running in Stratos Datacenter. The application servers were unable to connect to the database.

After investigation, the production support team identified that the MariaDB service was down on the database server.

**Requirement:**
- Investigate why MariaDB was not running.
- Identify and fix the underlying issue.
- Start the MariaDB service successfully.

## Server Details

| Component | Details |
|---|---|
| Server | Database Server |
| Hostname | stdb01 |
| User | peter |
| Service | mariadb |
| Database directory | /var/lib/mysql |
| Database service account | mysql |

## Step 1: Connect to the Database Server

From the Jump Host:

```bash
ssh peter@stdb01
```

Verify the server and user:

```bash
whoami
hostname
```

This confirms that we are connected to the correct database server.

## Step 2: Check MariaDB Service Status

```bash
sudo systemctl status mariadb
```

Initially, the output showed:

```text
Loaded: loaded
Active: inactive (dead)
```

**What this means:**

- `Loaded: loaded` – The MariaDB service definition is available to systemd.
- `Active: inactive (dead)` – The MariaDB service is currently stopped.
- `disabled` – The service is not configured to start automatically during system boot.

## Step 3: Attempt to Start MariaDB

```bash
sudo systemctl start mariadb
```

The service failed to start.

We needed to investigate the reason instead of repeatedly restarting it.

## Step 4: Investigate Service Logs

Check detailed service status:

```bash
sudo systemctl status mariadb -l
```

Check MariaDB-specific logs:

```bash
sudo journalctl -xeu mariadb --no-pager
```

The important errors were:

```text
chown: changing ownership of '/var/lib/mysql': Operation not permitted

chmod: changing permissions of '/var/lib/mysql': Operation not permitted

Cannot change ownership of the database directories to the 'mysql' user.
```

**Root cause identified:**

MariaDB was unable to initialise its database directory because it could not change the ownership and permissions of `/var/lib/mysql` as required.

## Step 5: Inspect Database Directory Ownership

```bash
ls -ld /var/lib/mysql
```

Initial output:

```text
drwxr-xr-x 1 root mysql 4096 Apr 29 2026 /var/lib/mysql
```

The directory was owned by `root` and its group was `mysql`.

MariaDB uses the `mysql` system account to access its database files. The directory needed the appropriate ownership for the service to operate.

## Step 6: Check the Filesystem

```bash
findmnt -T /var/lib/mysql
```

The output showed:

```text
TARGET SOURCE FSTYPE OPTIONS
/      overlay overlay rw,...
```

The database directory was on an overlay filesystem, commonly used in container-based environments.

This was a useful diagnostic clue while investigating the ownership error.

## Step 7: Check the MySQL System Account

```bash
id mysql
```

Output:

```text
uid=27(mysql) gid=27(mysql) groups=27(mysql)
```

This confirmed that the `mysql` user and group existed on the server.

## Step 8: Check Directory Attributes

```bash
sudo lsattr -d /var/lib/mysql
```

Output:

```text
--------------e------- /var/lib/mysql
```

The `e` represents the extents attribute. There was no `i` immutable attribute.

Therefore, an immutable directory attribute was not the cause of the issue.

## Step 9: Test Ownership Changes

We created a temporary file to check whether ownership changes were possible on the system.

```bash
sudo touch /tmp/mysql-permission-test
```

Change its ownership:

```bash
sudo chown mysql:mysql /tmp/mysql-permission-test
```

Verify:

```bash
ls -l /tmp/mysql-permission-test
```

Output:

```text
-rw-r--r-- 1 mysql mysql 0 Nov 13 17:48 /tmp/mysql-permission-test
```

This confirmed that ownership changes were possible on `/tmp`.

The temporary test file could be removed using:

```bash
sudo rm -f /tmp/mysql-permission-test
```

## Step 10: Fix Database Directory Ownership

The actual correction was:

```bash
sudo chown -R mysql:mysql /var/lib/mysql
```

**Command explanation:**

- `sudo` – Executes the command with administrator privileges.
- `chown` – Changes ownership.
- `-R` – Applies the ownership change recursively to the directory and its contents.
- First `mysql` – New owner.
- Second `mysql` – New group.
- `/var/lib/mysql` – MariaDB database directory.

This corrected the ownership of the database directory and its contents.

The command completed successfully without displaying an error.

## Step 11: Start MariaDB Again

```bash
sudo systemctl start mariadb
```

This time, the service started successfully.

Verify:

```bash
sudo systemctl status mariadb
```

Final output confirmed:

```text
Active: active (running)
Status: "Taking your SQL requests now..."
```

Additional verification:

```bash
sudo systemctl is-active mariadb
```

Expected output:

```text
active
```

## Important Commands Learned

| Command | Purpose |
|---|---|
| `systemctl status` | Check service status |
| `systemctl start` | Start a service |
| `systemctl stop` | Stop a service |
| `systemctl restart` | Restart a service |
| `systemctl enable` | Enable automatic startup at boot |
| `systemctl is-active` | Check whether a service is running |
| `journalctl` | Inspect system and service logs |
| `ls -ld` | Check directory permissions and ownership |
| `findmnt` | Identify the filesystem containing a path |
| `id` | Display user and group information |
| `chown` | Change ownership |
| `chmod` | Change permissions |
| `lsattr` | Display filesystem attributes |
| `touch` | Create an empty file |
| `rm` | Remove files |

## What I Learned

1. MariaDB is a database management system used to store and manage application data.
2. A Linux service can be installed but stopped or disabled.
3. `systemctl` is used to manage background services.
4. `journalctl` helps identify the actual reason behind service failures.
5. Linux uses users and groups to control file access.
6. MariaDB normally runs using the `mysql` system account.
7. `/var/lib/mysql` is the default database data directory in this environment.
8. `chown` changes ownership, whereas `chmod` changes permissions.
9. The `-R` option applies changes recursively.
10. Correct ownership and permissions are important for database services to start successfully.

## Real-Time DevOps Learning

Imagine a production database server suddenly stops accepting connections.

A DevOps engineer should:

1. Check whether the database service is running.
2. Inspect service logs to identify the startup failure.
3. Check database directory ownership and permissions.
4. Investigate filesystem restrictions if ownership changes fail.
5. Correct the identified issue without deleting existing database files.
6. Start the service and verify that it is running.

This approach helps resolve production issues systematically instead of reinstalling services or making random configuration changes.

## Final Status

- [x] Connected to the database server.
- [x] Identified that MariaDB was inactive.
- [x] Investigated service logs.
- [x] Identified the database directory ownership issue.
- [x] Corrected ownership of `/var/lib/mysql`.
- [x] Started MariaDB successfully.
- [x] Verified that MariaDB is active and running.

**Task Status: Completed**

**Additional note:** MariaDB was initially configured as `disabled` for automatic startup. The `systemctl enable mariadb` command is an additional step to configure startup after reboot; it should be run and verified separately.

---

*100 Days of DevOps – Day 09*
