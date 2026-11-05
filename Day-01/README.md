# Day 01 – Linux User Management

## Challenge Question

Create a user named `siva` on App Server 1 within the Stratos Datacenter. The user must have a non-interactive shell.

## 1. Server Details

- Server: App Server 1
- Hostname: `stapp01`
- Username used to connect: `tony`
- New user to create: `siva`

## 2. Connect to App Server 1

From the Jump Host:

```bash
ssh tony@stapp01
```

SSH (Secure Shell) allows secure remote access to another Linux server.

## 3. Create the User with a Non-Interactive Shell

```bash
sudo useradd -s /sbin/nologin siva
```

### Command Breakdown

- `sudo` – Executes the command with administrator privileges.
- `useradd` – Creates a new Linux user.
- `-s` – Specifies the user's login shell.
- `/sbin/nologin` – Prevents normal interactive login.
- `siva` – Username being created.

## 4. Verify User Creation

```bash
getent passwd siva
```

Expected output format:

```text
siva:x:UID:GID::/home/siva:/sbin/nologin
```

The important part is the final field:

```text
/sbin/nologin
```

This confirms that the user has a non-interactive login shell.

## 5. Important Linux Concepts

| Concept | Explanation |
|---|---|
| `useradd` | Creates a user account |
| `sudo` | Runs commands with elevated privileges |
| `/etc/passwd` | Stores user account information |
| `getent passwd` | Retrieves user account information |
| `/bin/bash` | Interactive shell |
| `/sbin/nologin` | Prevents normal interactive login |

## 6. Real-World DevOps Connection

In real-world environments, service accounts are often created for applications, automated processes and background services.

These accounts generally do not need interactive shell access. Using `/sbin/nologin` helps reduce unnecessary access and improves server security.

## 7. Final Outcome

- Connected to App Server 1 using SSH.
- Created the user `siva`.
- Assigned `/sbin/nologin` as the login shell.
- Verified the account using `getent passwd`.

**Status: Completed ✓**

### Key Learning

Learned how to create Linux users and configure non-interactive login shells, an important part of Linux administration and server security.
