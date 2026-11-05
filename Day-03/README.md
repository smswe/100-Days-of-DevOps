# Day 03 - Linux Security: Disable Direct Root SSH Login

## Challenge Question

Following security audits, the xFusionCorp Industries security team has rolled out new protocols, including restriction of direct root SSH login.

**Task:** Disable direct SSH root login on all application servers within the Stratos Datacenter.

## 1. Server Details

| Server | Hostname | Login Username |
|---|---|---|
| App Server 1 | `stapp01` | `tony` |
| App Server 2 | `stapp02` | `steve` |
| App Server 3 | `stapp03` | `banner` |

## 2. What is Root?

In Linux, `root` is the superuser or administrator account.

It has extensive privileges to:
- Read and modify system files.
- Install and remove software.
- Manage users and permissions.
- Control system services.

Allowing direct root SSH login can increase security risks. Therefore, administrators can disable direct root login while retaining administrative access through `sudo`.

## 3. Connect to the Application Servers

From the Jump Host, connect to each application server.

**App Server 1:**

```bash
ssh tony@stapp01
```

**App Server 2:**

```bash
ssh steve@stapp02
```

**App Server 3:**

```bash
ssh banner@stapp03
```

## 4. Modify the SSH Server Configuration

The SSH server configuration file is:

```text
/etc/ssh/sshd_config
```

Open the configuration file:

```bash
sudo vi /etc/ssh/sshd_config
```

Find the setting:

```text
PermitRootLogin
```

Set the active configuration to:

```text
PermitRootLogin no
```

If the line is commented out, remove the `#` and configure it correctly.

Save the file using:

- Press `Esc`.
- Type `:wq`.
- Press `Enter`.

**Important:** Make sure there are no conflicting active `PermitRootLogin` settings.

## 5. Validate the SSH Configuration

Run:

```bash
sudo sshd -t
```

This checks the SSH configuration for syntax errors.

If the command produces no output, the configuration passed the syntax check.

## 6. Verify the Effective Configuration

Run:

```bash
sudo sshd -T | grep permitrootlogin
```

Expected output:

```text
permitrootlogin no
```

This confirms that the effective SSH configuration disables direct root login.

## 7. Restart the SSH Service

Check the SSH service:

```bash
sudo systemctl status sshd
```

Restart it to apply the configuration:

```bash
sudo systemctl restart sshd
```

Verify that the service is running:

```bash
sudo systemctl status sshd
```

Expected status:

```text
Active: active (running)
```

Keep your existing session open until you verify that normal administrative SSH access still works.

## 8. Important Linux Concepts

| Command / Term | Purpose |
|---|---|
| `ssh` | Secure remote connection |
| `sudo` | Execute commands with elevated privileges |
| `vi` | Linux text editor |
| `sshd` | SSH server daemon |
| `sshd_config` | SSH server configuration file |
| `PermitRootLogin no` | Disables direct root SSH login |
| `sshd -t` | Checks SSH configuration syntax |
| `sshd -T` | Displays effective SSH configuration |
| `systemctl status` | Checks service status |
| `systemctl restart` | Restarts a service |
| `grep` | Searches for matching text |

## 9. Difference Between sshd -t and sshd -T

| Command | Explanation |
|---|---|
| `sudo sshd -t` | Checks whether the configuration is valid |
| `sudo sshd -T` | Displays the effective SSH server configuration |

Remember:

- Lowercase `-t` = Configuration test.
- Uppercase `-T` = Display effective configuration.

## 10. Troubleshooting

During configuration, an initial verification showed:

```text
permitrootlogin yes
```

After checking the configuration file, I found an active setting:

```text
PermitRootLogin yes
```

I changed the active setting to:

```text
PermitRootLogin no
```

Another issue was that `grep` is case-sensitive by default. The effective configuration output uses lowercase `permitrootlogin`, so the search pattern must match its case.

## 11. Real-World DevOps Connection

SSH is commonly used by DevOps engineers and system administrators to access remote servers.

Disabling direct root login is an example of **Linux security hardening**.

In production environments:
- Administrators connect using individual user accounts.
- Privileged operations are performed using `sudo`.
- SSH access is restricted according to security policies.
- Configuration changes are validated before restarting services.

## 12. Final Outcome

- Connected to all three application servers.
- Updated the SSH server configuration.
- Disabled direct root SSH login.
- Validated the configuration using `sshd -t`.
- Verified the effective setting using `sshd -T`.
- Restarted the SSH service.

**Status: Completed ✓**

### Key Learning

Learned how to configure SSH security, disable direct root login, validate SSH configuration and manage Linux services using `systemctl`.
