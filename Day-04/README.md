# Day 04 - Linux File Permissions

## Challenge

The task was to grant read and execute permissions to the script `/tmp/xfusioncorp.sh` for all users on App Server 3.

## Server Details

- **Server:** App Server 3
- **Hostname:** `stapp03`
- **Username:** `banner`
- **File:** `/tmp/xfusioncorp.sh`

## Step 1: Connect to App Server 3

From the Jump Host, connect using SSH:

```bash
ssh banner@stapp03
```

## Step 2: Check Existing File Permissions

```bash
ls -l /tmp/xfusioncorp.sh
```

Initial output:

```text
---------- 1 root root 40 Nov 5 13:11 /tmp/xfusioncorp.sh
```

The file initially had no permissions for the owner, group, or other users.

## Step 3: Grant Permissions

First, I tried:

```bash
sudo chmod a+x /tmp/xfusioncorp.sh
```

However, the KodeKloud validator did not accept this permission setting.

The reason was that the file had execute permission but no read permission.

I then executed:

```bash
sudo chmod a+rx /tmp/xfusioncorp.sh
```

This granted read and execute permissions to all users.

## Step 4: Verify Permissions

```bash
ls -l /tmp/xfusioncorp.sh
```

Final output:

```text
-r-xr-xr-x 1 root root 40 Nov 5 13:11 /tmp/xfusioncorp.sh
```

The final permissions were successfully applied, and the KodeKloud task was completed.

## Understanding the Command

### chmod

`chmod` stands for Change Mode. It is used to modify file and directory permissions in Linux.

### Breaking down `sudo chmod a+rx`

| Part | Meaning |
|---|---|
| `sudo` | Execute with administrator privileges |
| `chmod` | Change file permissions |
| `a` | All users: owner, group and others |
| `+` | Add permissions |
| `r` | Read permission |
| `x` | Execute permission |

### Understanding Linux Permissions

Linux has three permission categories:

- **Owner:** The user who owns the file.
- **Group:** Users belonging to the file's group.
- **Others:** Everyone else.

The final permission output:

```text
-r-xr-xr-x
```

- `r-x` – Owner can read and execute.
- `r-x` – Group members can read and execute.
- `r-x` – Other users can read and execute.

No write permission was granted.

## Useful Commands Learned

```bash
ls -l filename
```
Displays file permissions.

```bash
chmod a+rx filename
```
Adds read and execute permissions for everyone.

```bash
chmod a+x filename
```
Adds execute permission for everyone.

```bash
chmod a-x filename
```
Removes execute permission for everyone.

```bash
chmod a-w filename
```
Removes write permission for everyone.

## Real-World DevOps Connection

In DevOps, scripts are frequently used for automation, deployment, backups, monitoring, and server maintenance.

For example, a deployment script must have appropriate permissions before users or automation processes can execute it.

Incorrect permissions can cause permission-denied errors during automated tasks.

Understanding Linux permissions helps DevOps engineers manage access securely and troubleshoot script execution problems.

## Key Takeaway

Today I learned how Linux file permissions work and how to use `chmod` to grant read and execute permissions to all users.

**Final Status: Completed successfully on App Server 3.**
