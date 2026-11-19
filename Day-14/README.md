# Day 14 - Troubleshoot Apache Service on Application Servers

## Task / Question

The production support team of xFusionCorp Industries has deployed some of the latest monitoring tools to keep an eye on every service, application, etc. running on the systems.

One of the monitoring systems reported about Apache service unavailability on one of the app servers in `Stratos DC`.

**Task requirements:**

1. Identify the faulty application server.
2. Troubleshoot and fix the Apache service issue.
3. Make sure Apache is up and running on all three application servers.
4. Make sure Apache is running on port `3004` on all application servers.
5. The servers might not have hosted any website code yet, so we do not need to worry about whether Apache serves web pages. We only need to ensure that the Apache service is running.

---

## 1. Understand the Task

### What is Apache?

Apache HTTP Server is a web server software that receives HTTP requests from clients and sends HTTP responses.

For example:

- A user opens a website in a browser.
- The browser sends an HTTP request to the server.
- Apache receives the request.
- Apache responds with the requested page or an HTTP error.

Apache is commonly managed through the `httpd` service on CentOS/RHEL-based Linux systems.

### What is httpd?

`httpd` is the name of the Apache HTTP Server daemon on this Linux distribution.

We use `systemctl` to manage this service.

```bash
systemctl status httpd
```

This checks whether Apache is running, stopped, or failed.

### What is port 3004?

A port is a communication endpoint used by an application.

For this task, Apache must listen on TCP port `3004` on all three application servers.

The required servers are:

| Server | SSH user | Required port |
|---|---|---|
| stapp01 | tony | 3004 |
| stapp02 | steve | 3004 |
| stapp03 | banner | 3004 |

---

## 2. Initial Connectivity Testing

I started from the Jump Host and checked whether Apache was responding on all three servers.

### Command

```bash
curl -I http://stapp01:3004
curl -I http://stapp02:3004
curl -I http://stapp03:3004
```

### What does `curl -I` mean?

- `curl` is a command-line tool used to communicate with URLs and servers.
- `-I` requests only the HTTP response headers.
- `http://stapp01:3004` specifies the server and port.

### Initial results

**stapp01:**

```text
curl: (7) Failed to connect to stapp01 port 3004: Connection refused
```

**stapp02:**

```text
HTTP/1.1 403 Forbidden
Server: Apache/2.4.62 (CentOS Stream)
```

**stapp03:**

```text
HTTP/1.1 403 Forbidden
Server: Apache/2.4.62 (CentOS Stream)
```

### Understanding the results

| Result | Meaning |
|---|---|
| Connection refused | The connection was rejected because no service was accepting connections at that destination |
| HTTP 403 Forbidden | The server responded, but access to the requested resource was denied |
| HTTP 200 OK | The request succeeded |

**Important:** HTTP 403 is acceptable for this task. The task does not require the website content to be available. It requires Apache to be running and listening on port 3004.

### Conclusion

`stapp01` was the faulty server because it refused connections on port 3004.

---

## 3. Troubleshooting stapp01

### Step 1 – Connect to App Server 1

From the Jump Host:

```bash
ssh tony@stapp01
```

This connects to App Server 1 using the user `tony`.

### Step 2 – Check Apache service status

```bash
systemctl status httpd
```

Initial output:

```text
httpd.service - The Apache HTTP Server

Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled)
Active: failed (Result: exit-code)

(98)Address already in use: AH00072
no listening sockets available, shutting down
```

### Understanding the error

The important error was:

```text
(98)Address already in use
```

This means Apache tried to use a port that was already occupied by another process.

Apache could not bind to the configured port, so it failed to start.

We needed to identify which process was using port 3004.

---

## 4. Identify the Process Using Port 3004

### Command

```bash
sudo ss -lntp | grep 3004
```

### Output

```text
LISTEN 0 10 127.0.0.1:3004 0.0.0.0:* users:(("sendmail",pid=70183,fd=4))
```

### Understanding the command

`ss` is used to inspect network sockets.

| Option | Meaning |
|---|---|
| `-l` | Show listening sockets |
| `-n` | Display numeric addresses and port numbers |
| `-t` | Show TCP sockets |
| `-p` | Show the process using the socket |
| `grep 3004` | Filter output to show port 3004 |

### Understanding the output

```text
127.0.0.1:3004
```

The process was listening on port 3004 on the local loopback interface.

```text
sendmail
```

This identifies the application occupying the port.

```text
pid=70183
```

This is the process ID of Sendmail.

### Root cause discovered

Sendmail was using port 3004, while Apache was also configured to use port 3004.

This port conflict prevented Apache from starting.

---

## 5. Verify Apache Port Configuration

Before fixing the issue, I checked Apache's configuration file.

### Command

```bash
sudo grep -n '^Listen' /etc/httpd/conf/httpd.conf
```

### Output

```text
47:Listen 3004
```

### Explanation

- `grep` searches for matching text.
- `-n` displays the matching line number.
- `^Listen` searches for lines beginning with `Listen`.
- `/etc/httpd/conf/httpd.conf` is Apache's main configuration file.

The output confirmed that Apache was configured to listen on port 3004.

I initially mistyped the configuration file path:

```bash
sudo grep -n '^Listen' /etc/httpd/cong/httpd.conf
```

It returned:

```text
No such file or directory
```

The correct directory is `conf`, not `cong`.

---

## 6. Verify Sendmail Service

To confirm that Sendmail was running, I checked its service status.

### Command

```bash
systemctl status sendmail
```

### Output

```text
sendmail.service - Sendmail Mail Transport Agent

Active: active (running)

Main PID: 70183 (sendmail)
```

This confirmed that Sendmail was running and occupying port 3004.

### What is Sendmail?

Sendmail is a mail transfer agent (MTA).

It is used to transfer email between systems.

In this lab, Sendmail was not supposed to occupy the port required by Apache.

---

## 7. Fix the Port Conflict

### Step 1 – Stop Sendmail

```bash
sudo systemctl stop sendmail
```

This stops the Sendmail service immediately.

### Step 2 – Verify Sendmail stopped

```bash
systemctl status sendmail
```

Output:

```text
sendmail.service - Sendmail Mail Transport Agent

Active: inactive (dead)
```

This confirms Sendmail has stopped.

Stopping Sendmail released the port that Apache needed.

### Step 3 – Start Apache

```bash
sudo systemctl start httpd
```

This starts the Apache HTTP Server.

### Step 4 – Verify Apache

```bash
systemctl status httpd
```

Output:

```text
httpd.service - The Apache HTTP Server

Active: active (running)

Main PID: 96784 (httpd)

Server configured, listening on: port 3004
```

### Result

Apache successfully started on `stapp01` and was configured to listen on port 3004.

---

## 8. Check App Server 2

I connected to App Server 2:

```bash
ssh steve@stapp02
```

Then checked which process was listening on port 3004:

```bash
sudo ss -lntp | grep 3004
```

Output:

```text
LISTEN 0 511 *:3004 *:* users:(("httpd",pid=67757,...))
```

### Explanation

The output showed:

```text
httpd
```

Apache was already listening on port 3004.

The `*` indicates that the listening socket is bound to all available local IPv4 interfaces.

The initial Jump Host test also returned:

```text
HTTP/1.1 403 Forbidden
Server: Apache/2.4.62 (CentOS Stream)
```

This confirms that the server was responding through Apache.

No changes were needed based on these checks.

---

## 9. Enable Apache Automatically on stapp01

After starting Apache, I noticed that its service status showed:

```text
Loaded: loaded (.../httpd.service; disabled)
Active: active (running)
```

This means Apache was running now, but it was not enabled to start automatically when the server boots.

To make it persistent, I ran:

```bash
sudo systemctl enable --now httpd
```

### Understanding the command

| Part | Meaning |
|---|---|
| `systemctl` | Manage Linux services |
| `enable` | Configure the service to start automatically during boot |
| `--now` | Start the service immediately as well |
| `httpd` | Apache service name |

Output:

```text
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service
→ /usr/lib/systemd/system/httpd.service
```

### Verify the service

```bash
systemctl status httpd
```

Final status showed:

```text
Loaded: loaded (.../httpd.service; enabled)
Active: active (running)
```

This confirms that Apache is both running and enabled.

---

## 10. Final Understanding: What Was the Actual Problem?

The issue was not caused by Apache's configuration file or missing website content.

It was a port conflict.

The sequence was:

1. Apache was configured to listen on port 3004.
2. Sendmail was already listening on port 3004.
3. Apache attempted to start.
4. Linux rejected Apache's attempt to bind to the occupied port.
5. Apache failed with `Address already in use`.
6. We stopped Sendmail.
7. Port 3004 became available.
8. Apache started successfully.
9. We enabled Apache to start automatically after reboot.

### Before and after

| Item | Before | After |
|---|---|---|
| Sendmail on stapp01 | Active | Inactive |
| Apache on stapp01 | Failed | Active |
| Apache port | Unable to bind | 3004 |
| Apache boot startup | Disabled | Enabled |

---

## 11. Useful Commands Learned

| Command | Purpose |
|---|---|
| `curl -I http://hostname:3004` | Test HTTP connectivity |
| `systemctl status httpd` | Check Apache service status |
| `sudo ss -lntp` | Display listening TCP sockets and processes |
| `grep -n '^Listen' /etc/httpd/conf/httpd.conf` | Find Apache listening-port configuration |
| `systemctl status sendmail` | Check Sendmail service |
| `sudo systemctl stop sendmail` | Stop Sendmail |
| `sudo systemctl start httpd` | Start Apache |
| `sudo systemctl enable --now httpd` | Enable and start Apache |
| `systemctl is-enabled httpd` | Check whether Apache starts automatically |
| `systemctl is-active httpd` | Check whether Apache is currently running |

---

## 12. Real-World DevOps Learning

Imagine a production server running multiple services.

Apache needs port 3004, but another application accidentally starts using the same port.

The monitoring system detects that Apache is unavailable.

A DevOps engineer should not immediately reinstall Apache or change its configuration.

Instead, the engineer should:

1. Check the service status.
2. Read the error message.
3. Identify the port Apache needs.
4. Find which process is using that port.
5. Understand whether that process should be running.
6. Resolve the conflict safely.
7. Restart Apache.
8. Verify service health and network connectivity.
9. Enable the service at boot if required.

**Key takeaway:** When a service fails to start, always investigate the error and identify the process using the required port before making changes.

This is a common Linux production-support and DevOps troubleshooting scenario.
