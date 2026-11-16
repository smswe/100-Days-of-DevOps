# Day 12 - Troubleshoot Apache Connectivity and Configure Firewall Rules

## 1. Task Question

The monitoring tool has reported that one of the application servers in the Stratos Datacenter is unable to serve Apache requests on port `3003`.

The issue could be related to:
- Apache service being down.
- Firewall blocking incoming connections.
- Apache listening on the wrong port or interface.
- Another process already using the required port.

### Requirements

1. Identify the faulty application server.
2. Troubleshoot and fix the issue.
3. Keep the firewall running. Do not stop or disable it.
4. Do not flush or remove existing firewall rules.
5. Allow only the required Apache port through the firewall.
6. Make sure Apache responds on port `3003` on all three application servers.
7. Test connectivity from the Jump Host using:

```bash
curl http://stapp01:3003
curl http://stapp02:3003
curl http://stapp03:3003
```

**Important:** Do not modify any website content inside `/var/www/html`.

---

## 2. Lab Environment

| Server | Hostname | Linux User | Purpose |
|---|---|---|---|
| Jump Host | jump-host | thor | Troubleshooting and testing |
| App Server 1 | stapp01 | tony | Apache server |
| App Server 2 | stapp02 | steve | Apache server |
| App Server 3 | stapp03 | banner | Apache server |

Required Apache port: `3003`

---

## 3. Step 1 – Initial Connectivity Testing

First, I logged in to the Jump Host and checked Apache connectivity on all three application servers.

```bash
curl -I http://stapp01:3003
curl -I http://stapp02:3003
curl -I http://stapp03:3003
```

### Initial results

**stapp01:**

```text
curl: (7) Failed to connect to stapp01 port 3003:
No route to host
```

**stapp02:**

```text
HTTP/1.1 403 Forbidden
Server: Apache/2.4.62 (CentOS Stream)
```

**stapp03:**

The initial test also returned an HTTP 403 response.

### Understanding the results

| Result | Meaning |
|---|---|
| No route to host | The connection was rejected or could not reach the destination |
| 403 Forbidden | Apache received the request but refused access to the requested resource |
| Connection refused | The destination was reachable, but no process accepted the connection on that port |

A 403 response is still an HTTP response. It tells us that Apache is reachable.

Therefore, `stapp01` was the faulty server.

**Important learning:** A 403 error does not automatically mean Apache is down. We must distinguish application-level errors from network connectivity errors.

---

## 4. Step 2 – Confirming Connectivity Using Telnet

I used Telnet to check whether TCP port `3003` was accepting connections.

```bash
telnet stapp01 3003
telnet stapp02 3003
telnet stapp03 3003
```

### Results

- `stapp01`: No route to host.
- `stapp02`: Connected successfully.
- `stapp03`: Connected successfully.

On stapp02 and stapp03, I entered `q` into the Telnet connection and Apache returned:

```text
HTTP/1.1 400 Bad Request
```

This is expected because typing `q` is not a valid HTTP request.

### What does Telnet help us understand?

Telnet can be used to test whether a TCP connection can be established to a particular server and port.

For example:

```bash
telnet stapp02 3003
```

If it displays `Connected to stapp02`, the TCP connection was established.

**Learning:** Telnet checks network connectivity, while curl sends an HTTP request and displays the HTTP response.

---

## 5. Step 3 – Connect to the Faulty Server

Since `stapp01` was the only server showing a connectivity failure, I connected to it.

From the Jump Host:

```bash
ssh tony@stapp01
```

I checked the Apache service:

```bash
sudo systemctl status httpd
```

### Initial output

Apache was in a failed state:

```text
httpd.service - The Apache HTTP Server
Active: failed (Result: exit-code)
```

The logs contained:

```text
(98)Address already in use: AH00072: make_sock
no listening sockets available, shutting down
```

### What does this error mean?

`Address already in use` means another process is already using the network address and port Apache wants to use.

Apache cannot bind to a port that is already occupied by another process.

We needed to identify which process was using port `3003`.

---

## 6. Step 4 – Check Which Process Is Using Port 3003

I ran:

```bash
sudo ss -lntp | grep 3003
```

### Initial output

```text
LISTEN 0 10 127.0.0.1:3003 0.0.0.0:* users:(("sendmail",pid=26121,fd=4))
```

This showed that Sendmail was listening on port `3003`.

### Understanding the command

| Part | Meaning |
|---|---|
| `sudo` | Run with administrator privileges |
| `ss` | Display socket information |
| `-l` | Show listening sockets |
| `-n` | Show numeric IP addresses and port numbers |
| `-t` | Show TCP sockets |
| `-p` | Show the process using the socket |
| `grep 3003` | Filter output containing port 3003 |

### Understanding the output

```text
127.0.0.1:3003
```

`127.0.0.1` is the loopback address, also called localhost.

It means Sendmail was listening on port `3003` on the local machine.

The output also showed:

```text
sendmail
pid=26121
```

This identified Sendmail as the process occupying the port.

---

## 7. Step 5 – Check Apache Port Configuration

I checked Apache's configuration:

```bash
sudo grep -n '^Listen' /etc/httpd/conf/httpd.conf
```

Output:

```text
47:Listen 3003
```

This confirmed that Apache was configured to listen on port `3003`.

### What is the Listen directive?

Apache uses the `Listen` directive to determine which port and network address it should accept connections on.

For example:

```apache
Listen 3003
```

This tells Apache to listen on port `3003`.

The port configuration was correct. The problem was that Sendmail was already occupying the same port.

---

## 8. Step 6 – Validate Apache Configuration

Before starting Apache, I checked whether its configuration had syntax errors.

```bash
sudo httpd -t
```

Output:

```text
AH00558: httpd: Could not reliably determine the server's fully qualified domain name
Syntax OK
```

### What does this mean?

- `httpd -t` checks Apache configuration syntax.
- `Syntax OK` means the configuration file has valid syntax.
- `AH00558` is a warning about the ServerName configuration. It was not the cause of the startup failure.

This confirmed that Apache's configuration syntax was valid.

---

## 9. Step 7 – Identify the Process Occupying Port 3003

I checked Sendmail's service status:

```bash
sudo systemctl status sendmail
```

Output showed:

```text
sendmail.service - Sendmail Mail Transport Agent
Active: active (running)
Main PID: 26121 (sendmail)
```

I also checked the process:

```bash
sudo ps -fp 26121
```

Output:

```text
UID   PID    PPID   CMD
root  26121  1      sendmail: accepting connections
```

To confirm the executable, I ran:

```bash
sudo readlink -f /proc/26121/exe
```

Output:

```text
/usr/sbin/sendmail.sendmail
```

### What did we learn?

We now had clear evidence:

1. Apache needed port `3003`.
2. Sendmail was already listening on port `3003`.
3. Apache could not start because the port was occupied.

The root cause of Apache's startup failure was the Sendmail port conflict.

---

## 10. Step 8 – Stop Sendmail and Start Apache

Since Sendmail was occupying the required Apache port, I stopped the Sendmail service.

```bash
sudo systemctl stop sendmail
```

Then I started Apache:

```bash
sudo systemctl start httpd
```

I initially typed the following command incorrectly:

```bash
sudo systemctl enable--now httpd
```

It returned:

```text
Unknown command verb enable--now
```

The correct command requires a space between `enable` and `--now`:

```bash
sudo systemctl enable --now httpd
```

### What does enable --now mean?

- `enable` configures the service to start automatically during system boot.
- `--now` starts the service immediately as well.

This means Apache starts now and is configured to start automatically after a reboot.

I verified the service:

```bash
sudo systemctl status httpd
```

Output showed:

```text
httpd.service - The Apache HTTP Server
Loaded: loaded (...; enabled; preset: disabled)
Active: active (running)
```

It also displayed:

```text
Server configured, listening on: port 3003
```

Apache was now running successfully.

---

## 11. Step 9 – Check Whether Apache Is Listening on All Interfaces

Even after Apache started, the Jump Host initially could not connect.

So I checked the listening socket again:

```bash
sudo ss -lntp | grep 3003
```

Output:

```text
LISTEN 0 511 *:3003 *:* users:(("httpd",pid=55989,fd=4),...)
```

### Understanding `*:3003`

The `*` indicates Apache is listening on all local network interfaces for port `3003`, rather than only on localhost.

This was important because the Jump Host needs to connect through the server's network interface.

I also tested Apache locally:

```bash
curl -I http://localhost:3003
```

Output:

```text
HTTP/1.1 403 Forbidden
Server: Apache/2.4.62 (CentOS Stream)
```

Apache was responding locally.

I checked the server IP:

```bash
hostname -I
```

Output:

```text
10.244.221.3
```

The Apache service was running and listening correctly. However, the Jump Host still could not reach it.

This meant we needed to investigate the firewall.

---

## 12. Step 10 – Investigate Firewall Configuration

First, I checked whether the firewalld service existed:

```bash
sudo systemctl status firewall
sudo systemctl status firewalld
```

Both returned:

```text
Unit firewalld.service could not be found.
```

This did not prove that the server had no firewall rules. It only told us that the firewalld service was not available under that service name.

Next, I checked iptables:

```bash
sudo iptables -S
```

### Initial output

```text
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT

-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited

-A FORWARD -j REJECT --reject-with icmp-host-prohibited
```

### Understanding the important rules

**Rule 1:**

```bash
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
```

Allows incoming packets belonging to already established or related connections.

**Rule 2:**

```bash
-A INPUT -p icmp -j ACCEPT
```

Allows ICMP traffic, which is commonly used by ping.

**Rule 3:**

```bash
-A INPUT -i lo -j ACCEPT
```

Allows traffic through the loopback interface.

**Rule 4:**

```bash
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
```

Allows new TCP connections to port `22`, which is used by SSH.

**Rule 5:**

```bash
-A INPUT -j REJECT --reject-with icmp-host-prohibited
```

Rejects incoming traffic that has not matched an earlier ACCEPT rule.

### What was missing?

There was no ACCEPT rule for TCP port `3003`.

Therefore, new incoming connections to Apache's port were being rejected.

This explained why Apache worked locally but the Jump Host could not connect.

I also tried:

```bash
sudo nft list ruleset
```

Output:

```text
sudo: nft: command not found
```

The nft command was unavailable in this environment. I continued using the available iptables rules.

---

## 13. Step 11 – Allow Apache Port 3003 Through the Firewall

The task specifically required keeping the firewall active and allowing only the required Apache port.

I added a rule to accept new TCP connections on port `3003`:

```bash
sudo iptables -I INPUT 5 -p tcp --dport 3003 -m state --state NEW -j ACCEPT
```

### Understanding every part

| Command part | Meaning |
|---|---|
| `sudo` | Run as administrator |
| `iptables` | Manage firewall rules |
| `-I` | Insert a rule |
| `INPUT` | Incoming traffic chain |
| `5` | Insert at position 5 |
| `-p tcp` | Match TCP traffic |
| `--dport 3003` | Match destination port 3003 |
| `-m state` | Use connection-state matching |
| `--state NEW` | Match new connections |
| `-j ACCEPT` | Allow matching traffic |

### Why did I use `-I INPUT 5`?

The existing firewall had a REJECT rule:

```bash
-A INPUT -j REJECT --reject-with icmp-host-prohibited
```

Firewall rules are evaluated in order.

If a packet reaches the REJECT rule before an allow rule, it gets rejected.

By inserting the new rule at position 5, it is placed before the existing REJECT rule.

The new rule allows the required Apache traffic without removing the existing firewall protections.

---

## 14. Step 12 – Test Connectivity Again

After adding the firewall rule, I exited from App Server 1:

```bash
exit
```

I returned to the Jump Host:

```text
thor@jump-host ~$
```

Then I tested Apache again:

```bash
curl -I http://stapp01:3003
```

### Final result for stapp01

```text
HTTP/1.1 403 Forbidden
Date: Mon, 16 Nov 2026 16:35:56 GMT
Server: Apache/2.4.62 (CentOS Stream)
Last-Modified: Fri, 12 Dec 2025 14:14:29 GMT
Content-Type: text/html; charset=UTF-8
```

### Success!

Earlier, the result was:

```text
No route to host
```

After adding the firewall rule, the result became:

```text
HTTP/1.1 403 Forbidden
```

This confirms that the Jump Host can now reach Apache on `stapp01:3003`.

The 403 response is an application-level response. We did not modify website content because the task specifically prohibited it.

---

## 15. Troubleshooting Summary

| Step | Problem / Check | Finding |
|---|---|---|
| 1 | curl from Jump Host | stapp01 unreachable |
| 2 | Telnet test | stapp02 and stapp03 accepted TCP connections |
| 3 | Apache service status | httpd failed on stapp01 |
| 4 | Check port using ss | Sendmail occupied port 3003 |
| 5 | Check Apache configuration | `Listen 3003` was correct |
| 6 | Apache syntax test | `Syntax OK` |
| 7 | Sendmail service check | Sendmail was active |
| 8 | Stop Sendmail and start Apache | Apache became active |
| 9 | Check Apache listening socket | Apache listened on `*:3003` |
| 10 | Check iptables | Port 3003 was not allowed |
| 11 | Add firewall rule | Incoming TCP port 3003 allowed |
| 12 | Test from Jump Host | HTTP 403 response received |

---

## 16. Important Commands Learned

| Command | Purpose |
|---|---|
| `curl -I URL` | Check HTTP response headers |
| `telnet hostname port` | Test TCP connectivity |
| `systemctl status httpd` | Check Apache service status |
| `systemctl start httpd` | Start Apache |
| `systemctl stop sendmail` | Stop Sendmail |
| `systemctl enable --now httpd` | Enable and start Apache |
| `ss -lntp` | Display listening TCP ports and processes |
| `grep` | Search for matching text |
| `httpd -t` | Check Apache configuration syntax |
| `ps -fp PID` | Inspect a process |
| `readlink -f` | Resolve a symbolic link to its full path |
| `hostname -I` | Display server IP addresses |
| `iptables -S` | Display firewall rules |
| `iptables -I` | Insert a firewall rule |
| `curl -I localhost:3003` | Test Apache locally |

---

## 17. What I Learned

### 1. A running service does not guarantee network access

Apache can be running and listening on the correct port, but the firewall may still block incoming connections.

### 2. Multiple services cannot bind to the same address and port

Sendmail was using port `3003`, which Apache needed. This caused Apache to fail during startup.

### 3. `ss` helps identify port conflicts

Instead of guessing which application is using a port, we can identify the process using:

```bash
sudo ss -lntp
```

### 4. Firewall rules are processed in order

An ACCEPT rule must be placed before a matching REJECT rule for the traffic to be allowed.

### 5. HTTP 403 is different from a connectivity failure

- `No route to host`: connection could not be established.
- `403 Forbidden`: the HTTP server responded but denied access to the requested resource.

### 6. Troubleshoot layer by layer

A useful troubleshooting sequence is:

1. Check the service.
2. Check the listening port.
3. Identify port conflicts.
4. Test locally.
5. Inspect firewall rules.
6. Fix the specific issue.
7. Test remotely.

---

## 18. Final Verification Checklist

The following reflects the work completed and the remaining checks.

- [x] Identified stapp01 as the faulty server.
- [x] Found Sendmail occupying port 3003.
- [x] Stopped Sendmail.
- [x] Started Apache.
- [x] Enabled Apache to start automatically.
- [x] Confirmed Apache listening on `*:3003`.
- [x] Identified missing firewall permission for port 3003.
- [x] Added an ACCEPT rule for new TCP connections on port 3003.
- [x] Confirmed Jump Host can reach stapp01.
- [x] Confirmed stapp02 responds to HTTP requests.
- [ ] Perform final curl verification for stapp03.
- [ ] Perform final three-server curl verification.

### Final test to run from Jump Host

```bash
curl -I http://stapp01:3003
curl -I http://stapp02:3003
curl -I http://stapp03:3003
```

Expected: Each server should return an HTTP response.

---

## 19. Real-World DevOps Connection

Imagine an application is running on a Linux server, and the service team reports that the application is unavailable.

A DevOps or Linux administrator should not immediately restart every service or disable the firewall.

Instead, investigate:

- Is the application process running?
- Is the required port listening?
- Is another process occupying that port?
- Is the server firewall allowing incoming connections?
- Can the application be reached locally?
- Can it be reached from another server?

This lab demonstrates a real production troubleshooting pattern: **service failure, port conflict, firewall investigation, targeted rule correction, and remote verification.**

The same approach is useful when troubleshooting web servers, APIs, databases, and other network services.

---

## 20. Day 12 Completion Status

**Day 12 – Apache Connectivity Troubleshooting and Firewall Configuration**

Root cause on stapp01:
- Sendmail occupied TCP port 3003.
- Apache could not start until Sendmail was stopped.
- After Apache started, the firewall was still rejecting incoming connections to port 3003.
- Added a specific iptables ACCEPT rule for TCP port 3003.
- Verified successful HTTP connectivity from the Jump Host.

**Note:** The final stapp03 and all-three-server verification is still to be recorded after testing.

