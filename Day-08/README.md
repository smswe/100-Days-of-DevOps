# Day 08 - Install Ansible 4.8.0 on Jump Host

## Task Description

During the weekly meeting, the Nautilus DevOps team decided to use **Ansible** as their configuration management and automation tool.

The Jump Host will act as the Ansible Controller, which will be used to manage and execute tasks on the other servers.

**Requirement:**

- Install Ansible version `4.8.0` on the Jump Host.
- Use `pip3` only for installation.
- Make sure Ansible is available globally so that all users on the system can execute Ansible commands.

## Server Details

| Component           | Details            |
| ------------------- | ------------------ |
| Server              | Jump Host          |
| User                | thor               |
| Role                | Ansible Controller |
| Installation method | pip3               |
| Required version    | 4.8.0              |

## Step 1: Verify the Current User and Host

```bash
whoami
hostname
```

This confirms that we are working on the Jump Host.

## Step 2: Check pip3

```bash
pip3 --version
```

This verifies that the Python 3 package installer is available.

## Step 3: Install Ansible 4.8.0

```bash
sudo pip3 install ansible==4.8.0
```

**Command explanation:**

- `sudo` – Runs the command with administrator privileges.
- `pip3` – Python 3 package installer.
- `install` – Installs the specified package.
- `ansible==4.8.0` – Installs the exact requested Ansible version.

Using `sudo` installs the package system-wide instead of restricting it to the current user's local Python environment.

## Step 4: Verify Ansible Installation

```bash
ansible --version
```

This displays the installed Ansible version and related configuration details.

## Step 5: Check the Ansible Binary Location

```bash
which ansible
```

Expected location in a typical pip3 installation:

```text
/usr/local/bin/ansible
```

A system-wide executable location makes the command accessible to users whose PATH includes that directory.

## Step 6: Verify Access for Another User

```bash
sudo -u root ansible --version
```

This checks whether another account, in this case root, can execute the Ansible command.

## What I Learned

1. Ansible is an automation and configuration management tool.
2. The machine from which Ansible runs is called the **Control Node**.
3. In this task, the Jump Host acts as the Ansible Control Node.
4. Ansible can manage remote servers and automate repetitive administrative tasks.
5. `pip3` can be used to install Ansible and its Python dependencies.
6. Installing globally helps make the Ansible executable available to multiple users.

## Real-Time Use Case

Imagine a DevOps administrator needs to install a package on 50 servers.

Instead of connecting to each server individually and running the installation command, the administrator can use Ansible from a Control Node to automate the task across all 50 servers.

This saves time, reduces manual effort, and ensures consistency.

---
