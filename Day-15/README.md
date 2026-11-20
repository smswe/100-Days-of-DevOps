# Day 15 - Install and Configure Nginx with SSL

## Task Description

The system admins team of `xFusionCorp Industries` needs to deploy a new application on `App Server 1` in `Stratos Datacenter`. They have some prerequisites to get the server ready for application deployment.

**Requirements:**

1. Install and configure `nginx` on `App Server 1`.
2. On `App Server 1`, there is a self-signed SSL certificate and key present at `/tmp/nautilus.crt` and `/tmp/nautilus.key`. Move them to an appropriate location and deploy them in Nginx.
3. Create an `index.html` file with the content `Welcome!` under the Nginx document root.
4. For final testing, access the App Server 1 link (via hostname) from the Jump Host using curl. Example: `curl -Ik https://<app-server-name>/`.

---

## Environment

| Component | Details |
|---|---|
| Jump Host | `thor@jump-host` |
| App Server 1 | `tony@stapp01` |
| Operating System | CentOS Stream 9 |
| Web Server | Nginx |
| HTTPS Port | 443 |
| Document Root | `/usr/share/nginx/html` |
| SSL Directory | `/etc/nginx/ssl/` |

## Step 1 – Connect to App Server 1

From the Jump Host:

```bash
ssh tony@stapp01
```

- `ssh` – Secure Shell, used to connect to another Linux server.
- `tony` – Username on App Server 1.
- `stapp01` – Hostname of App Server 1.

The first connection displayed an authenticity confirmation because the host was not previously in the SSH known-hosts list.

Entered `yes` and provided the password to connect.

## Step 2 – Install Nginx

Initially entered:

```bash
sudo dnf-y install nginx
```

This returned:

```text
sudo: dnf-y: command not found
```

**Reason:** Missing space between `dnf` and `-y`. Linux interpreted `dnf-y` as a command name.

Correct command:

```bash
sudo dnf -y install nginx
```

- `sudo` – Runs the command with administrator privileges.
- `dnf` – Package manager used in CentOS/RHEL-based Linux distributions.
- `-y` – Automatically answers yes to installation prompts.
- `install nginx` – Installs the Nginx web server.

Installation completed successfully.

Installed packages included:
- nginx
- nginx-core
- nginx-filesystem
- centos-logos-httpd
- logrotate

## Step 3 – Create an SSL Directory

The SSL certificate and private key were initially available under `/tmp`.

Created a dedicated directory:

```bash
sudo mkdir -p /etc/nginx/ssl
```

**Command explanation:**

- `mkdir` – Creates a directory.
- `-p` – Creates parent directories if needed and does not report an error if the directory already exists.
- `/etc/nginx/ssl` – Directory selected to store SSL files.

## Step 4 – Move SSL Certificate and Key

Moved the certificate:

```bash
sudo mv /tmp/nautilus.crt /etc/nginx/ssl/
```

Moved the private key:

```bash
sudo mv /tmp/nautilus.key /etc/nginx/ssl/
```

**Command explanation:**

- `mv` – Moves or renames files.
- `/tmp/nautilus.crt` – Source certificate.
- `/tmp/nautilus.key` – Source private key.
- `/etc/nginx/ssl/` – Destination directory.

Verified the files:

```bash
ls -l /etc/nginx/ssl
```

Output:

```text
total 8
-rw-r--r-- 1 root root 2170 Nov 20 17:22 nautilus.crt
-rw-r--r-- 1 root root 3267 Nov 20 17:22 nautilus.key
```

Both files were successfully moved.

## Step 5 – Set Ownership and Permissions

Set ownership:

```bash
sudo chown root:root /etc/nginx/ssl/nautilus.crt /etc/nginx/ssl/nautilus.key
```

Set certificate permissions:

```bash
sudo chmod 644 /etc/nginx/ssl/nautilus.crt
```

Set private key permissions:

```bash
sudo chmod 600 /etc/nginx/ssl/nautilus.key
```

**Understanding permissions:**

| Permission | Meaning |
|---|---|
| `644` | Owner can read/write; group and others can read |
| `600` | Only owner can read/write |
| `root:root` | Root user owns the files and the root group is assigned |

**Why different permissions?**

- The SSL certificate is public information, so it can be readable.
- The private key is sensitive and should be accessible only to root.

## Step 6 – Configure Nginx for HTTPS

Created a separate SSL configuration file:

```bash
sudo vi /etc/nginx/conf.d/ssl.conf
```

Added the following configuration:

```nginx
server {
    listen 443 ssl;
    server_name stapp01;

    ssl_certificate /etc/nginx/ssl/nautilus.crt;
    ssl_certificate_key /etc/nginx/ssl/nautilus.key;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Saved and exited using:

```text
Esc
:wq
Enter
```

**Configuration explanation:**

| Directive | Purpose |
|---|---|
| `server` | Defines a virtual server configuration |
| `listen 443 ssl` | Enables HTTPS on port 443 |
| `server_name stapp01` | Sets the hostname for this virtual server |
| `ssl_certificate` | Specifies the SSL certificate path |
| `ssl_certificate_key` | Specifies the private key path |
| `root` | Defines the website document root |
| `index index.html` | Sets the default webpage |
| `location /` | Defines how incoming requests are handled |
| `try_files` | Checks whether the requested file exists; returns 404 if it does not |

**Important:** HTTPS uses port 443 by default. Nginx uses the configured certificate and private key to establish encrypted connections.

## Step 7 – Create index.html

Created the webpage in the Nginx document root:

```bash
echo "Welcome!" | sudo tee /usr/share/nginx/html/index.html
```

**Command explanation:**

- `echo` – Prints text.
- `Welcome!` – Required webpage content.
- `|` – Pipe; sends the output of one command to another.
- `tee` – Writes the received content into a file.
- `sudo` – Allows writing to a protected system directory.

Verified the webpage:

```bash
cat /usr/share/nginx/html/index.html
```

Output:

```text
Welcome!
```

The webpage was created successfully.

## Step 8 – Test Nginx Configuration

Before starting Nginx, checked the configuration:

```bash
sudo nginx -t
```

Output:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

**Command explanation:**

- `nginx` – Nginx executable.
- `-t` – Tests configuration syntax and checks whether referenced files can be accessed.
- `sudo` – Runs the test with administrator privileges.

This confirms that the Nginx configuration is valid.

## Step 9 – Enable and Start Nginx

Executed:

```bash
sudo systemctl enable --now nginx
```

Output:

```text
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /usr/lib/systemd/system/nginx.service.
```

**Command explanation:**

- `systemctl` – Manages system services using systemd.
- `enable` – Configures the service to start automatically at boot.
- `--now` – Starts the service immediately.
- `nginx` – Name of the service.

Verified service status:

```bash
sudo systemctl status nginx
```

Important output:

```text
Loaded: loaded (.../nginx.service; enabled)
Active: active (running)
```

Nginx was successfully started and enabled.

## Step 10 – Test HTTPS Locally

While still logged into App Server 1:

```bash
curl -Ik https://localhost/
```

Output:

```text
HTTP/1.1 200 OK
Server: nginx/1.20.1
Date: Fri, 20 Nov 2026 17:47:56 GMT
Content-Type: text/html
Content-Length: 9
Last-Modified: Fri, 20 Nov 2026 17:45:29 GMT
Connection: keep-alive
ETag: "6b008739-9"
Accept-Ranges: bytes
```

**Command explanation:**

- `curl` – Transfers data to or from a URL.
- `-I` – Requests only HTTP response headers.
- `-k` – Allows connections to self-signed or otherwise untrusted SSL certificates.
- `https://localhost/` – Tests the local HTTPS endpoint.

**HTTP/1.1 200 OK** confirms that Nginx successfully responded to the HTTPS request.

## Step 11 – Final Test from Jump Host

Exited App Server 1:

```bash
exit
```

Returned to:

```text
thor@jump-host
```

Ran the final test using the App Server hostname:

```bash
curl -k https://stapp01/
```

Output:

```text
Welcome!
```

This confirms:

- The Jump Host can reach App Server 1 using its hostname.
- HTTPS connectivity is working.
- Nginx is serving the correct webpage.
- The SSL certificate and private key are configured successfully.

---

## Important Commands Learned

| Command | Purpose |
|---|---|
| `dnf -y install nginx` | Install Nginx |
| `mkdir -p` | Create a directory and missing parent directories |
| `mv` | Move files |
| `ls -l` | List files with permissions and ownership |
| `chown` | Change file ownership |
| `chmod` | Change file permissions |
| `vi` | Edit configuration files |
| `echo` | Print text |
| `tee` | Write command output to a file |
| `cat` | Display file contents |
| `nginx -t` | Test Nginx configuration |
| `systemctl enable --now nginx` | Enable and start Nginx |
| `systemctl status nginx` | Check Nginx service status |
| `curl -Ik` | Test HTTPS response headers |
| `curl -k` | Test HTTPS while bypassing certificate trust validation |

## Troubleshooting Notes

**1. `sudo: dnf-y: command not found`**

Cause: Missing space between `dnf` and `-y`.

Correction:

```bash
sudo dnf -y install nginx
```

**2. `-bash: $'\r': command not found`**

Cause: An accidental carriage-return character (`^M`) was entered into the terminal.

It was harmless and did not affect the Nginx setup.

**3. Self-signed SSL certificate**

A self-signed certificate is not automatically trusted by curl. Use `-k` during testing:

```bash
curl -k https://stapp01/
```

This bypasses certificate verification for the test; it does not make the certificate trusted.

---

## Final Result

| Requirement | Status |
|---|---|
| Install Nginx on App Server 1 | Completed |
| Move SSL certificate and key | Completed |
| Configure SSL in Nginx | Completed |
| Set certificate and private key permissions | Completed |
| Create `index.html` with `Welcome!` | Completed |
| Validate Nginx configuration | Completed |
| Enable and start Nginx | Completed |
| Test HTTPS locally | Completed |
| Test website from Jump Host using hostname | Completed |

**Final Status: COMPLETED**

Successfully installed and configured Nginx with HTTPS on App Server 1 (`stapp01`). The self-signed SSL certificate and private key were deployed, and the application webpage returned `Welcome!` when accessed from the Jump Host.

**Key Learning:** Nginx is a web server that can serve websites over HTTP and HTTPS. SSL certificates and private keys enable encrypted HTTPS communication, while `systemctl` manages the service and `curl` helps verify connectivity and responses.
