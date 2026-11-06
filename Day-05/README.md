# Day 05 - SELinux Installation and Configuration

## 1. Task

Following a security audit, the requirement was to install SELinux packages and permanently disable SELinux on App Server 2. A reboot was not required because maintenance reboot was already scheduled.

## 2. Server Details

- **Server:** App Server 2
- **Hostname:** `stapp02`
- **Username:** `steve`
- **Operating System:** CentOS Stream 9

## 3. What is SELinux?

SELinux stands for **Security-Enhanced Linux**.

It is a Linux security mechanism that provides an additional layer of access control. It controls which processes and users can access files, directories and other system resources.

SELinux uses **Mandatory Access Control (MAC)** to enforce security policies.

### Why is SELinux needed?

Consider a web server running on Linux. It needs access to website files, but it should not be allowed to access sensitive system files.

Normal Linux permissions provide access control, while SELinux adds another security layer to restrict unauthorized access.

## 4. Steps Performed

### Step 1: Connect to App Server 2

```bash
ssh steve@stapp02
```

### Step 2: Identify the Operating System

```bash
cat /etc/os-release
```

Output confirmed:

```text
NAME="CentOS Stream"
VERSION="9"
```

### Step 3: Check Existing SELinux Packages

```bash
rpm -qa | grep -i selinux
```

This command lists installed packages containing `selinux` in their names.

### Step 4: Install Required SELinux Packages

```bash
sudo dnf install -y policycoreutils policycoreutils-python-utils selinux-policy selinux-policy-targeted
```

**Command explanation:**

- `sudo` – Executes the command with administrator privileges.
- `dnf` – Package manager used in CentOS Stream 9.
- `install` – Installs the specified packages.
- `-y` – Automatically confirms the installation.

The required packages and dependencies were installed successfully.

### Step 5: Permanently Disable SELinux

Open the SELinux configuration file:

```bash
sudo vi /etc/selinux/config
```

Locate the `SELINUX` setting and change it to:

```ini
SELINUX=disabled
```

Save and exit using:

1. Press `Esc`.
2. Type `:wq`.
3. Press `Enter`.

### Step 6: Verify the Configuration

```bash
grep '^SELINUX=' /etc/selinux/config
```

Expected output:

```text
SELINUX=disabled
```

**Understanding the grep command:**

- `grep` – Searches for matching text.
- `^` – Matches the beginning of a line.
- `SELINUX=` – Searches for the SELinux configuration setting.

Using `^` helps avoid matching commented-out lines such as `#SELINUX=enforcing`.

## 5. Understanding SELinux Modes

SELinux has three operating modes:

| Mode | Description |
|---|---|
| Enforcing | Enforces security policies and blocks unauthorized access. |
| Permissive | Allows access but logs policy violations. |
| Disabled | Turns off SELinux protection. |

## 6. Important SELinux Commands

```bash
getenforce
```
Displays the current SELinux mode.

```bash
sestatus
```
Displays detailed SELinux status.

```bash
sudo vi /etc/selinux/config
```
Opens the permanent SELinux configuration file.

## 7. Important Learning

There is a difference between the current SELinux status and its permanent configuration.

- `getenforce` shows the current runtime mode.
- `/etc/selinux/config` specifies the mode to use during boot.

Changing the configuration file to `SELINUX=disabled` does not immediately change the current runtime status. A reboot is required for the new setting to take effect.

The task specifically required no reboot, so the server was left running with the updated configuration.

## 8. Real-World DevOps Connection

SELinux is important in production environments because it provides additional security for Linux servers.

DevOps engineers may encounter SELinux-related permission denials while deploying applications, configuring services or troubleshooting server access.

Understanding SELinux helps engineers identify whether an access issue is caused by normal Linux permissions or SELinux security policies.

## 9. Final Outcome

- Required SELinux packages installed successfully.
- Permanent configuration changed to `SELINUX=disabled`.
- Configuration verified using `grep`.
- Server was not rebooted, as instructed.

**Status: Completed successfully!**
