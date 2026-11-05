# Day 04 - Linux File Permissions

## Task
Grant read and execute permissions to all users for `/tmp/xfusioncorp.sh` on App Server 3.

## Server Details
- Server: `stapp03`
- User: `banner`

## Commands Used

**1. Connect to the server**
```bash
ssh banner@stapp03
```

**2. Check existing permissions**
```bash
ls -l /tmp/xfusioncorp.sh
```

Initial permission: `----------`

**3. Grant read and execute permissions**
```bash
sudo chmod a+rx /tmp/xfusioncorp.sh
```

**4. Verify**
```bash
ls -l /tmp/xfusioncorp.sh
```

Final permission:
```text
-r-xr-xr-x
```

## Command Explanation

- `chmod` – Changes file permissions.
- `a` – All users (owner, group and others).
- `+` – Adds permissions.
- `r` – Read permission.
- `x` – Execute permission.

**Note:** Initially, I used `chmod a+x`, but the validator rejected it because read permission was missing. Using `chmod a+rx` resolved the issue.

## DevOps Connection

Scripts are commonly used for deployment, backups and automation. Correct permissions ensure authorized users and automation processes can access and execute them.

**Status: Completed successfully!**
