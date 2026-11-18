# Day 13 - Install and Configure iptables Firewall on App Servers

## Task / Question

We have one of our websites up and running on our Nautilus infrastructure in Stratos DC. The Security Team has raised a concern that Apache port `6000` is open to everyone because no firewall has been installed on these hosts.

We need to configure firewall rules to restrict access to this port.

### Requirements

1. Install `iptables` and all its dependencies, including `iptables-services`, on all three application servers:
   - App Server 1 – `stapp01`
   - App Server 2 – `stapp02`
   - App Server 3 – `stapp03`

2. Block incoming traffic on TCP port `6000` from everyone except the Load Balancer host `stlb01`.

3. Traffic from the Load Balancer must be allowed to access port `6000`.

4. Save the firewall rules in:

   `/etc/sysconfig/iptables`

5. Enable the iptables service so that the rules persist after a reboot.

6. Do not reboot the servers.

---

## 1. Understanding the Task

### What is a firewall?

A firewall is a security mechanism that controls incoming and outgoing network traffic based on predefined rules.

Think of an office building:

- The building has an entrance.
- Security checks who is allowed inside.
- Employees with permission can enter.
- Visitors without permission are denied.

A server firewall works similarly. It checks network traffic and decides whether to allow or block it.

### What is iptables?

`iptables` is a Linux command-line firewall utility used to configure rules that control network traffic.

For this task, we need to control incoming TCP traffic to port `6000`.

### What is port 6000?

A port is a communication endpoint used by applications on a server.

For example:

| Port | Common use |
|---|---|
| 22 | SSH |
| 80 | HTTP |
| 443 | HTTPS |
| 3000 | Common application development port |
| 6000 | Port specified in this lab |

The task specifically asks us to protect port `6000`.

### What is the Load Balancer?

A Load Balancer distributes incoming client requests across application servers.

In this lab, `stlb01` is the Load Balancer.

The requirement is:

- Load Balancer → App Server port 6000: ALLOW
- Any other source → App Server port 6000: BLOCK

This is an example of restricting access based on the source IP address.

---

## 2. Understand the Two Required Firewall Rules

We need two important rules on every application server.

### Rule 1 – Allow the Load Balancer

```bash
sudo iptables -I INPUT 1 -p tcp -s <LBR_IP> --dport 6000 -j ACCEPT
```

Meaning:

| Option | Explanation |
|---|---|
| `sudo` | Run the command with administrator privileges |
| `iptables` | Firewall management command |
| `-I` | Insert a rule |
| `INPUT` | Chain that handles incoming traffic |
| `1` | Insert at position 1 |
| `-p tcp` | Match TCP traffic |
| `-s` | Specify the source IP address |
| `<LBR_IP>` | Actual IP address of the Load Balancer |
| `--dport 6000` | Destination port 6000 |
| `-j ACCEPT` | Allow matching traffic |

### Rule 2 – Block everyone else

```bash
sudo iptables -I INPUT 2 -p tcp --dport 6000 -j REJECT
```

Meaning:

| Option | Explanation |
|---|---|
| `-I INPUT 2` | Insert the rule at position 2 in INPUT |
| `-p tcp` | Match TCP traffic |
| `--dport 6000` | Match destination port 6000 |
| `-j REJECT` | Reject matching traffic |

**Why is rule order important?**

iptables checks rules from top to bottom.

1. First, it checks whether the traffic came from the Load Balancer.
2. If yes, the first rule allows it.
3. Otherwise, the second rule rejects traffic to port 6000.

If we put the REJECT rule before the ACCEPT rule, even the Load Balancer could be blocked.

---

## 3. Step-by-Step Implementation

### Step 1 – Connect to App Server 1

From the Jump Host:

```bash
ssh tony@stapp01
```

### Step 2 – Install iptables

```bash
sudo dnf install -y iptables
```

Explanation:

- `dnf` is the package manager used on this CentOS/RHEL-based system.
- `install` installs a package.
- `-y` automatically confirms installation prompts.

### Step 3 – Install iptables-services

```bash
sudo dnf install -y iptables-services
```

Why do we need this package?

`iptables` provides the firewall command, while `iptables-services` provides the systemd service integration used to start, stop, enable, and restore saved firewall rules.

Both are needed for this task.

### Step 4 – Verify the installation

Commands used:

```bash
which iptables
```

This displays the location of the iptables executable.

Example output:

```text
/usr/sbin/iptables
```

Check the version:

```bash
iptables --version
```

Example output:

```text
iptables v1.8.10 (legacy)
```

Check the installed package:

```bash
rpm -q iptables-legacy
```

Example output:

```text
iptables-legacy-1.8.10-11.1.el9.x86_64
```

Check the service package:

```bash
rpm -q iptables-services
```

### Important Learning: Why did `rpm -q iptables` say the package was not installed?

The system installed the package named `iptables-legacy`.

`rpm -q` checks the exact RPM package name. It does not necessarily use the command name or package capability that DNF resolved.

Therefore:

```bash
rpm -q iptables
```

may report that the exact package is not installed, while:

```bash
rpm -q iptables-legacy
```

confirms the actual installed package.

The `iptables` command can still work because it is provided by the installed implementation.

---

## 4. Find the Load Balancer IP

Before creating firewall rules, we must find the actual IP address of `stlb01`.

Run this from the Jump Host:

```bash
getent hosts stlb01
```

Example format:

```text
10.x.x.x stlb01
```

Use the actual IP returned by your lab. Do not copy the example IP.

Why do we need it?

The firewall must identify traffic coming specifically from the Load Balancer.

We will use this IP in the `-s` option of the ACCEPT rule.

---

## 5. Inspect Existing Firewall Rules

Before making changes, inspect the current rules:

```bash
sudo iptables -S
```

This displays the firewall rules in command format.

Another useful command:

```bash
sudo iptables -L INPUT -n --line-numbers
```

Explanation:

- `-L` lists firewall rules.
- `INPUT` displays incoming traffic rules.
- `-n` displays numeric IP addresses and ports.
- `--line-numbers` shows the position of each rule.

**Important:** Never blindly flush existing firewall rules. Other services may depend on them.

For example, App Server 1 already has a rule from Day 12 allowing TCP port `3003`. We must preserve it.

---

## 6. Add the Firewall Rules

After identifying the Load Balancer IP and inspecting the current rules, insert the two rules on each app server.

Replace `<LBR_IP>` with the actual Load Balancer IP.

Allow the Load Balancer:

```bash
sudo iptables -I INPUT 1 -p tcp -s <LBR_IP> --dport 6000 -j ACCEPT
```

Reject everyone else:

```bash
sudo iptables -I INPUT 2 -p tcp --dport 6000 -j REJECT
```

Verify:

```bash
sudo iptables -L INPUT -n --line-numbers
```

The two new rules should appear in this order:

1. ACCEPT traffic from the Load Balancer to TCP port 6000.
2. REJECT other TCP traffic to port 6000.

---

## 7. Save Firewall Rules Permanently

One important concept:

Rules added using the `iptables` command are held in the active firewall configuration in memory. They are not automatically guaranteed to survive a reboot.

Save the rules:

```bash
sudo iptables-save | sudo tee /etc/sysconfig/iptables
```

Explanation:

| Command | Meaning |
|---|---|
| `iptables-save` | Prints all current iptables rules |
| `\|` | Sends the output of one command to another |
| `tee` | Writes the received content into a file |
| `/etc/sysconfig/iptables` | File where the rules are saved |

We use this command instead of manually writing only the two new rules because it saves the complete current firewall configuration, including existing rules.

Verify the saved file:

```bash
sudo cat /etc/sysconfig/iptables
```

---

## 8. Enable and Start the iptables Service

Run:

```bash
sudo systemctl enable --now iptables
```

Explanation:

- `systemctl` manages system services.
- `enable` configures the service to start automatically at boot.
- `--now` starts the service immediately.
- `iptables` is the service name.

Check its status:

```bash
sudo systemctl status iptables
```

Check whether it is enabled:

```bash
sudo systemctl is-enabled iptables
```

Check whether it is running:

```bash
sudo systemctl is-active iptables
```

Expected results:

```text
enabled
active
```

No reboot is required for this lab.

---

## 9. Repeat on All Three App Servers

The same firewall configuration must be applied to:

| Server | SSH user | Required configuration |
|---|---|---|
| stapp01 | tony | Install packages, add rules, save and enable |
| stapp02 | steve | Install packages, add rules, save and enable |
| stapp03 | banner | Install packages, add rules, save and enable |

Use the same Load Balancer source IP on all three servers.

Inspect existing rules separately on each server before inserting the new rules.

---

## 10. Useful Commands Learned

| Command | Purpose |
|---|---|
| `dnf install -y iptables` | Install iptables |
| `dnf install -y iptables-services` | Install iptables service integration |
| `which iptables` | Find the command location |
| `iptables --version` | Check iptables version |
| `rpm -q iptables-legacy` | Check installed implementation package |
| `getent hosts stlb01` | Resolve Load Balancer hostname |
| `iptables -S` | Display firewall rules |
| `iptables -L INPUT -n --line-numbers` | List incoming rules with line numbers |
| `iptables -I INPUT` | Insert an incoming firewall rule |
| `iptables-save` | Export active firewall rules |
| `systemctl enable --now iptables` | Enable and start the service |
| `systemctl status iptables` | Check service status |
| `systemctl is-enabled iptables` | Check automatic startup configuration |
| `systemctl is-active iptables` | Check whether the service is running |

---

## 11. Important Concepts to Remember

### ACCEPT

Allows matching network traffic.

### REJECT

Blocks matching traffic and sends a rejection response.

### DROP

Blocks matching traffic silently without sending a rejection response.

### INPUT chain

Controls traffic coming into the local server.

### OUTPUT chain

Controls traffic leaving the local server.

### FORWARD chain

Controls traffic passing through the server to another destination.

### Source IP (`-s`)

Identifies where the traffic originated.

### Destination port (`--dport`)

Identifies the port on the receiving server that the traffic is trying to access.

### Persistent firewall rules

Saving rules and enabling the iptables service ensures the rules can be restored and applied automatically after a reboot.

---

## 12. Final Verification Checklist

- [ ] `iptables` installed on stapp01, stapp02 and stapp03.
- [ ] `iptables-services` installed on all three servers.
- [ ] Load Balancer IP identified.
- [ ] Existing firewall rules inspected and preserved.
- [ ] Load Balancer allowed to access TCP port 6000.
- [ ] Other sources rejected from TCP port 6000.
- [ ] Rules saved to `/etc/sysconfig/iptables` on all three servers.
- [ ] iptables service enabled and active on all three servers.
- [ ] No server reboot performed.

---

## 13. Real-World DevOps Scenario

Imagine three application servers behind a Load Balancer.

Without firewall restrictions, any machine that can reach the application servers may attempt to access port 6000 directly.

This can bypass the intended entry point.

By allowing only the Load Balancer's IP address, we make the Load Balancer the permitted entry point for this port.

This is an example of **network access control and least-privilege security**: allow only the traffic that is required.

**Key takeaway:** A service being available does not mean it should be accessible to everyone. DevOps engineers must ensure that network access is restricted to trusted sources.
