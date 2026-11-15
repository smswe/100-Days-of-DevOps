# Day 11 - Install and Configure Tomcat Server

## Task

The Nautilus application development team has completed a beta version of a Java-based application and plans to deploy it on one of the application servers in the Stratos Datacenter. They have decided to use Tomcat as the application server.

Perform the following steps:

1. Install the Tomcat server on **App Server 1**.
2. Configure Tomcat to run on port **3004**.
3. A file named `ROOT.war` is available on the Jump Host at `/tmp`.
4. Deploy the WAR file on Tomcat so that the application is accessible directly from the base URL:

   `http://stapp01:3004`

---

## Server Details

| Component | Details |
|---|---|
| Jump Host | `thor@jump-host` |
| App Server 1 | `tony@stapp01` |
| Tomcat Port | `3004` |
| WAR File | `ROOT.war` |
| Deployment Directory | `/var/lib/tomcat/webapps/` |

## Step 1: Connect to App Server 1

From the Jump Host:

```bash
ssh tony@stapp01
```

Verify the server:

```bash
whoami
hostname
```

- `whoami` – displays the current logged-in user.
- `hostname` – displays the server name.

## Step 2: Install Tomcat

Install the Tomcat package:

```bash
sudo dnf install -y tomcat
```

Verify installation:

```bash
rpm -q tomcat
```

- `dnf` – package manager used to install software.
- `install` – installs the requested package.
- `-y` – automatically answers yes to confirmation prompts.
- `rpm -q` – queries whether a package is installed.

## Step 3: Configure Tomcat Port

Open the Tomcat configuration file:

```bash
sudo vi /etc/tomcat/server.xml
```

Find the HTTP Connector configuration, which normally uses port `8080`, and change it to:

```xml
<Connector port="3004" protocol="HTTP/1.1"
           connectionTimeout="20000"
           redirectPort="8443" />
```

Save and exit:

```text
Esc
:wq
Enter
```

Verify the configuration:

```bash
sudo grep -n 'Connector.*3004' /etc/tomcat/server.xml
```

**Why change the port?**

Tomcat commonly uses port `8080` by default. The task specifically requires port `3004`, so the HTTP Connector must listen on that port.

## Step 4: Start Tomcat

Enable and start the Tomcat service:

```bash
sudo systemctl enable --now tomcat
```

Check its status:

```bash
sudo systemctl status tomcat
```

Check whether Tomcat is listening on port `3004`:

```bash
sudo ss -lntp | grep 3004
```

- `systemctl` – manages system services.
- `enable` – configures a service to start automatically during boot.
- `--now` – starts the service immediately as well.
- `ss` – displays socket and network connection information.
- `-l` – shows listening sockets.
- `-n` – displays numeric port numbers.
- `-t` – shows TCP sockets.
- `-p` – displays the process using the socket.

## Step 5: Copy ROOT.war from Jump Host

The WAR file is initially available on the Jump Host.

First, verify the file:

```bash
ls -lh /tmp/ROOT.war
```

If the file has a different name, such as `ROOT.WAR`, rename it:

```bash
mv /tmp/ROOT.WAR /tmp/ROOT.war
```

Linux filenames are case-sensitive. `ROOT.WAR` and `ROOT.war` are treated as different filenames.

Copy the WAR file to App Server 1:

```bash
scp /tmp/ROOT.war tony@stapp01:/tmp/ROOT.war
```

Verify the file on App Server 1:

```bash
ls -lh /tmp/ROOT.war
```

- `scp` – securely copies files between servers.
- `/tmp/ROOT.war` – source file on the Jump Host.
- `tony@stapp01` – destination user and server.
- `:/tmp/ROOT.war` – destination file path.

## Step 6: Deploy the WAR File

Check the Tomcat web application directory:

```bash
rpm -ql tomcat | grep webapps
```

Copy the WAR file into the Tomcat deployment directory:

```bash
sudo cp /tmp/ROOT.war /var/lib/tomcat/webapps/ROOT.war
```

Verify:

```bash
sudo ls -lh /var/lib/tomcat/webapps/ROOT.war
```

Restart Tomcat:

```bash
sudo systemctl restart tomcat
```

Check the service:

```bash
sudo systemctl status tomcat
```

**Why is the file named ROOT.war important?**

Tomcat uses the WAR filename to determine the application context path.

- `ROOT.war` → application is deployed at `/`
- `myapp.war` → application is deployed at `/myapp`

Therefore, `ROOT.war` makes the application accessible directly from the base URL without an additional application path.

## Step 7: Verify the Application

From the Jump Host, run:

```bash
curl http://stapp01:3004
```

You can also check the HTTP response headers:

```bash
curl -I http://stapp01:3004
```

A successful deployment should return an HTTP response from the application, commonly `200 OK`.

---

## Troubleshooting Commands

| Command | Purpose |
|---|---|
| `sudo systemctl status tomcat` | Check Tomcat service status |
| `sudo journalctl -u tomcat -n 50 --no-pager` | View recent Tomcat logs |
| `sudo ss -lntp \| grep 3004` | Check whether port 3004 is listening |
| `sudo ls -lh /var/lib/tomcat/webapps/` | Check deployed application files |
| `curl -I http://stapp01:3004` | Check the HTTP response |

## What I Learned

- Tomcat is a Java application server used to run Java web applications.
- A WAR file is a Web Application Archive containing the files needed to deploy a Java web application.
- `server.xml` is used to configure Tomcat, including its HTTP port.
- `systemctl` helps manage services such as Tomcat.
- `scp` securely transfers files between Linux servers.
- Tomcat deploys WAR files placed in its web application deployment directory.
- Naming a WAR file `ROOT.war` makes the application available at the base URL.

## Real-World DevOps Application

In real-world environments, DevOps engineers deploy application packages to application servers, configure service ports, manage system services, and verify application availability.

Automating these steps helps make deployments repeatable and reduces manual errors.

## Final Status

- [x] Tomcat installed on App Server 1
- [x] Tomcat configured to use port 3004
- [x] Tomcat service started
- [x] ROOT.war copied from Jump Host
- [x] ROOT.war deployed to Tomcat
- [x] Application verified using curl

**Day 11 completed – Tomcat Installation and Java Application Deployment.**
