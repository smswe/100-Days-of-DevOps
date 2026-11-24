# Day 19 - Configure Apache to Host Two Static Websites

## Task

xFusionCorp Industries is planning to host two static websites on their infrastructure in Stratos Datacenter.

Perform the following tasks:

1. Install the `httpd` package and dependencies on App Server 3.
2. Configure Apache to serve on port `6400`.
3. Host two website backups:
   - `/home/thor/blog` from jump host → `http://localhost:6400/blog/`
   - `/home/thor/demo` from jump host → `http://localhost:6400/demo/`
4. Verify both websites using `curl` on App Server 3.

---

## Server Details

| Component | Details |
|---|---|
| Jump Host | `thor@jump-host` |
| App Server 3 | `banner@stapp03` |
| Apache service | `httpd` |
| Apache configuration | `/etc/httpd/conf/httpd.conf` |
| Apache document root | `/var/www/html` |
| Required port | `6400` |

## Step 1 – Connect to App Server 3

From the jump host:

```bash
ssh banner@stapp03
```

- `ssh` securely connects to a remote server.
- `banner` is the Linux username.
- `stapp03` is App Server 3.

## Step 2 – Install Apache and Dependencies

```bash
sudo dnf -y install httpd
```

**Explanation:**

- `dnf` – package manager used in CentOS/RHEL.
- `install` – installs a package.
- `-y` – automatically confirms installation.
- `httpd` – Apache HTTP Server package.

The installation also installs required dependencies, including APR, Apache core, and HTTP tools.

Verify installation:

```bash
which httpd
```

Observed output:

```text
/usr/sbin/httpd
```

This confirms that the Apache executable is available.

## Step 3 – Check Apache Service

```bash
sudo systemctl status httpd
```

Initially, the service showed:

```text
Active: inactive (dead)
```

This means Apache was installed but not running.

**Important:** The correct service name is `httpd`, not `apache`.

Check the service using:

```bash
sudo systemctl status httpd
```

## Step 4 – Configure Apache to Listen on Port 6400

Apache's main configuration file is:

```text
/etc/httpd/conf/httpd.conf
```

Open it using Nano:

```bash
sudo nano /etc/httpd/conf/httpd.conf
```

Find the existing directive:

```apache
Listen 80
```

Change it to:

```apache
Listen 6400
```

**Explanation:**

- `Listen` tells Apache which port to accept incoming HTTP connections on.
- Port `80` is the default HTTP port.
- Port `6400` is the port required by this task.

Save the file:

- `Ctrl + O` – save.
- Press `Enter` – confirm filename.
- `Ctrl + X` – exit Nano.

Verify:

```bash
sudo grep -n '^Listen' /etc/httpd/conf/httpd.conf
```

Expected:

```text
Listen 6400
```

## Step 5 – Understand Apache Document Root

Apache's default document root is:

```text
/var/www/html
```

This is the directory from which Apache serves website files.

For this task, the required directory structure is:

```text
/var/www/html/
├── blog/
└── demo/
```

The URL paths map to directories:

| URL | Directory |
|---|---|
| `http://localhost:6400/blog/` | `/var/www/html/blog/` |
| `http://localhost:6400/demo/` | `/var/www/html/demo/` |

Create both directories on App Server 3:

```bash
sudo mkdir -p /var/www/html/blog
sudo mkdir -p /var/www/html/demo
```

**Explanation:**

- `mkdir` – creates directories.
- `-p` – creates parent directories if necessary and avoids errors if the directories already exist.

## Step 6 – Copy Website Backups from Jump Host

The website backups are available on the jump host:

```text
/home/thor/blog
/home/thor/demo
```

Exit App Server 3:

```bash
exit
```

You should return to:

```text
thor@jump-host ~$
```

Copy the blog directory:

```bash
scp -r /home/thor/blog banner@stapp03:/tmp/
```

Copy the demo directory:

```bash
scp -r /home/thor/demo banner@stapp03:/tmp/
```

**Explanation:**

- `scp` – securely copies files between servers.
- `-r` – recursively copies directories and their contents.
- `/home/thor/blog` – source directory on jump host.
- `banner@stapp03:/tmp/` – destination directory on App Server 3.

The copied directories will be available as:

```text
/tmp/blog/
/tmp/demo/
```

## Step 7 – Copy Website Files into Apache Document Root

Reconnect to App Server 3:

```bash
ssh banner@stapp03
```

Copy the contents of the blog backup:

```bash
sudo cp -r /tmp/blog/. /var/www/html/blog/
```

Copy the contents of the demo backup:

```bash
sudo cp -r /tmp/demo/. /var/www/html/demo/
```

**Important:**

The `/.` at the end means copy all contents of the source directory, including hidden files.

This avoids accidentally creating an extra nested directory such as:

```text
/var/www/html/blog/blog/
```

Check the copied files:

```bash
ls -la /var/www/html/blog/
ls -la /var/www/html/demo/
```

## Step 8 – Set Ownership and Permissions

Set Apache ownership:

```bash
sudo chown -R apache:apache /var/www/html/blog
sudo chown -R apache:apache /var/www/html/demo
```

**Explanation:**

- `chown` – changes ownership.
- `-R` – applies recursively to all files and subdirectories.
- `apache:apache` – Apache user and group.

Set directory permissions:

```bash
sudo find /var/www/html/blog /var/www/html/demo -type d -exec chmod 755 {} \;
```

Set file permissions:

```bash
sudo find /var/www/html/blog /var/www/html/demo -type f -exec chmod 644 {} \;
```

**Permission explanation:**

| Permission | Meaning |
|---|---|
| `755` | Owner can read, write and execute; others can read and execute |
| `644` | Owner can read and write; others can read |

Apache needs read access to website files and execute permission on directories to traverse them.

## Step 9 – Validate Apache Configuration

```bash
sudo httpd -t
```

Expected output:

```text
Syntax OK
```

This checks the Apache configuration for syntax errors before starting the service.

## Step 10 – Start and Enable Apache

```bash
sudo systemctl enable --now httpd
```

**Explanation:**

- `enable` – configures Apache to start automatically at boot.
- `--now` – starts Apache immediately.

Check status:

```bash
sudo systemctl status httpd
```

Expected:

```text
Active: active (running)
```

If Apache was already running when its configuration changed, reload it:

```bash
sudo systemctl reload httpd
```

## Step 11 – Verify Apache Listening on Port 6400

Initially, the following command failed:

```bash
sudo ss -tulnp | grep 6400
```

Error:

```text
sudo: ss: command not found
```

**Reason:** The `ss` command is provided by the `iproute` package, which was not available on the server.

Install it:

```bash
sudo dnf install -y iproute
```

Then verify:

```bash
sudo ss -tulnp | grep 6400
```

This checks whether a process is listening on TCP port `6400`.

Alternative command, if `netstat` is installed:

```bash
sudo netstat -tulnp | grep 6400
```

## Step 12 – SELinux and Firewall Troubleshooting (Only If Required)

If Apache fails to bind to port `6400` because of SELinux restrictions, install the SELinux management utility:

```bash
sudo dnf install -y policycoreutils-python-utils
```

Check whether port `6400` is already assigned:

```bash
sudo semanage port -l | grep http_port_t
```

If the port is not defined, add it:

```bash
sudo semanage port -a -t http_port_t -p tcp 6400
```

If it already exists with another type, modify the existing entry instead:

```bash
sudo semanage port -m -t http_port_t -p tcp 6400
```

If `firewalld` is active and blocking access, allow port `6400`:

```bash
sudo firewall-cmd --permanent --add-port=6400/tcp
sudo firewall-cmd --reload
```

These are conditional troubleshooting steps, not mandatory commands for every lab.

## Step 13 – Final Website Verification

Run both commands directly on App Server 3:

### Test Blog

```bash
curl http://localhost:6400/blog/
```

### Test Demo

```bash
curl http://localhost:6400/demo/
```

Both commands should return their respective website content.

If you receive the website HTML or text instead of a connection error, Apache is serving the website successfully.

---

## Complete Command Reference

### Jump Host

```bash
# Connect to App Server 3
ssh banner@stapp03
```

### App Server 3

```bash
# Install Apache
sudo dnf -y install httpd

# Configure Apache port
sudo nano /etc/httpd/conf/httpd.conf

# Verify port
sudo grep -n '^Listen' /etc/httpd/conf/httpd.conf

# Create website directories
sudo mkdir -p /var/www/html/blog
sudo mkdir -p /var/www/html/demo

# Exit to jump host
exit
```

### Jump Host – Transfer Backups

```bash
# Copy blog backup
scp -r /home/thor/blog banner@stapp03:/tmp/

# Copy demo backup
scp -r /home/thor/demo banner@stapp03:/tmp/

# Reconnect to App Server 3
ssh banner@stapp03
```

### App Server 3 – Complete Configuration

```bash
# Copy blog website contents
sudo cp -r /tmp/blog/. /var/www/html/blog/

# Copy demo website contents
sudo cp -r /tmp/demo/. /var/www/html/demo/

# Set ownership
sudo chown -R apache:apache /var/www/html/blog
sudo chown -R apache:apache /var/www/html/demo

# Set directory permissions
sudo find /var/www/html/blog /var/www/html/demo -type d -exec chmod 755 {} \;

# Set file permissions
sudo find /var/www/html/blog /var/www/html/demo -type f -exec chmod 644 {} \;

# Validate Apache configuration
sudo httpd -t

# Start and enable Apache
sudo systemctl enable --now httpd

# Check Apache service
sudo systemctl status httpd

# Install ss command if missing
sudo dnf install -y iproute

# Verify listening port
sudo ss -tulnp | grep 6400

# Test blog
curl http://localhost:6400/blog/

# Test demo
curl http://localhost:6400/demo/
```

---

## Key Learning Points

| Concept | Purpose |
|---|---|
| `httpd` | Apache HTTP Server service/package |
| `httpd.conf` | Main Apache configuration file |
| `Listen 6400` | Configures Apache listening port |
| Document Root | Directory containing website files |
| `scp -r` | Transfers directories between servers |
| `cp -r` | Copies directories recursively |
| `chown` | Changes file ownership |
| `chmod` | Changes file permissions |
| `httpd -t` | Checks Apache configuration syntax |
| `systemctl` | Manages Linux services |
| `ss -tulnp` | Displays listening network sockets and processes |
| `curl` | Tests HTTP endpoints from the command line |

## Final Checklist

- [x] Apache package installed on App Server 3.
- [ ] Apache configured to listen on port `6400`.
- [ ] Blog backup copied to `/var/www/html/blog/`.
- [ ] Demo backup copied to `/var/www/html/demo/`.
- [ ] Apache configuration validated.
- [ ] Apache service running.
- [ ] Port `6400` listening.
- [ ] Blog URL tested successfully.
- [ ] Demo URL tested successfully.

**Status:** Mark as Completed only after both `curl` commands return the expected website content and the KodeKloud task validates successfully.
