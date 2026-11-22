# Day 16 - Configure Nginx as a Load Balancer

## Objective

Configure Nginx on the Load Balancer (LBR) server to distribute incoming HTTP requests across all three Apache application servers in the Nautilus infrastructure.

## Original Task

Day by day traffic is increasing on one of the websites managed by the `Nautilus` production support team. Therefore, the team has observed a degradation in website performance. Following discussions about this issue, the team has decided to deploy this application on a high availability stack i.e on `Nautilus` infra in `Stratos DC`. They started the migration last month and it is almost done, as only the LBR server configuration is pending.

Configure LBR server as per the information given below:

1. Install `nginx` on the `LBR` (load balancer) server if it is not already installed.
2. Configure load-balancing with the `http` context making use of all `App Servers`. Ensure that you update only the main Nginx configuration file located at `/etc/nginx/nginx.conf`.
3. Make sure you do not update the Apache port that is already defined in the Apache configuration on all app servers. Also make sure Apache service is up and running on all the app servers.
4. Once done, access the website by running `curl http://stlb01:80` in the terminal.

---

## 1. Infrastructure Details

| Server | Role | Service |
|---|---|---|
| stlb01 | Load Balancer | Nginx |
| stapp01 | App Server 1 | Apache |
| stapp02 | App Server 2 | Apache |
| stapp03 | App Server 3 | Apache |

**Important discovery:** The Apache port on all three application servers was `3000`, not `3002`.

## 2. Connect to the Load Balancer

From the Jump Host:

```bash
ssh loki@stlb01
```

Check Nginx installation:

```bash
nginx -v
```

Output:

```text
nginx version: nginx/1.20.1
```

Nginx was already installed, so installation was not required.

If Nginx is not installed on another environment:

```bash
sudo dnf install -y nginx
```

## 3. Verify Apache Backend Servers

From `stlb01`, test connectivity to all three application servers using their existing Apache port.

```bash
curl -I http://stapp01:3000
curl -I http://stapp02:3000
curl -I http://stapp03:3000
```

Output from all three servers:

```text
HTTP/1.1 200 OK
Server: Apache/2.4.62 (CentOS Stream)
Content-Type: text/html; charset=UTF-8
```

**Understanding the result:**

- `200 OK` means the HTTP request was successful.
- `Server: Apache` confirms the response came from Apache.
- Port `3000` is the existing Apache port.
- All three backend servers were reachable and responding.

This confirmed that the backend servers were ready to receive traffic.

**Important lesson:** Never assume the Apache port is 80, 3000, or 3002. Always inspect the existing Apache configuration or test the actual listening port.

## 4. Back Up the Nginx Configuration

Before making changes, create a backup of the main Nginx configuration file.

```bash
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
```

Explanation:

- `cp` copies a file.
- `/etc/nginx/nginx.conf` is the original configuration.
- `/etc/nginx/nginx.conf.bak` is the backup.

A backup allows us to restore the original configuration if a mistake occurs.

## 5. Edit the Main Nginx Configuration

The task specifically required updating only:

```text
/etc/nginx/nginx.conf
```

Open the file using nano:

```bash
sudo nano /etc/nginx/nginx.conf
```

### 5.1 Configure the upstream backend group

Inside the existing `http {}` block, add the following upstream configuration before the existing `server {}` block:

```nginx
upstream nautilus_backend {
    server stapp01:3000;
    server stapp02:3000;
    server stapp03:3000;
}
```

Explanation:

| Directive | Meaning |
|---|---|
| `upstream` | Defines a group of backend servers |
| `nautilus_backend` | Name assigned to the backend group |
| `server` | Adds an application server to the group |
| `stapp01:3000` | App Server 1 and its Apache port |
| `stapp02:3000` | App Server 2 and its Apache port |
| `stapp03:3000` | App Server 3 and its Apache port |

Nginx uses round-robin load balancing by default. Requests are distributed across the configured backend servers in sequence.

### 5.2 Configure the existing server block

The original Nginx configuration already contained a server block listening on port 80.

Inside that existing server block, configure the `location /` section as follows:

```nginx
location / {
    proxy_pass http://nautilus_backend;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

Explanation:

| Directive | Purpose |
|---|---|
| `location /` | Matches requests to the root path and general website paths |
| `proxy_pass` | Forwards requests to the upstream backend group |
| `proxy_set_header Host $host` | Passes the original hostname |
| `X-Real-IP` | Passes the original client IP |
| `X-Forwarded-For` | Preserves client IP information through proxies |
| `X-Forwarded-Proto` | Passes the original request protocol |

**Important:** The existing server block listening on port 80 was modified. A second competing server block was not required.

### 5.3 Save the file in nano

Nano shortcuts:

| Action | Shortcut |
|---|---|
| Save | `Ctrl + O` |
| Confirm filename | `Enter` |
| Exit | `Ctrl + X` |
| Search | `Ctrl + W` |

The task required the main Nginx configuration file, so no separate load-balancer configuration file was created.

## 6. Validate the Nginx Configuration

Before starting or reloading Nginx, test the configuration:

```bash
sudo nginx -t
```

Output:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Explanation:

- `nginx -t` checks the Nginx configuration syntax.
- It also checks whether referenced configuration files can be accessed.
- A successful test means Nginx can understand the configuration.

Always validate the configuration before applying changes.

## 7. Enable and Start Nginx

Run:

```bash
sudo systemctl enable --now nginx
```

Explanation:

- `systemctl` manages system services.
- `enable` configures Nginx to start automatically during system boot.
- `--now` starts the service immediately.

Check the service:

```bash
sudo systemctl status nginx
```

Output confirmed:

```text
Loaded: loaded (...; enabled)
Active: active (running)
```

This confirms Nginx is running on the Load Balancer.

## 8. Final Testing

Exit from the Load Balancer:

```bash
exit
```

You should return to the Jump Host.

Run the exact command specified in the task:

```bash
curl http://stlb01:80
```

Successful output:

```text
Welcome to xFusionCorp Industries!
```

This confirmed that the website was accessible through the Load Balancer URL.

---

## 9. Architecture – How Load Balancing Works

```text
                User / Client
                      |
                      |
                HTTP Request
                      |
                      v
              stlb01 (Nginx)
                 Port 80
                      |
                      |
             Upstream Backend Group
              nautilus_backend
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
       stapp01     stapp02     stapp03
       Apache      Apache      Apache
       :3000       :3000       :3000
          |           |           |
          +-----------+-----------+
                      |
                      v
               HTTP Response
                      |
                      v
                  Client
```

### Request flow

1. A client sends an HTTP request to `stlb01:80`.
2. Nginx receives the request.
3. Nginx identifies the upstream group `nautilus_backend`.
4. Nginx forwards the request to one of the three Apache servers.
5. Apache processes the request and returns the response.
6. Nginx sends the response back to the client.

## 10. Important Commands Learned

| Command | Purpose |
|---|---|
| `nginx -v` | Check Nginx version |
| `curl -I URL` | Display HTTP response headers |
| `curl URL` | Retrieve website response |
| `sudo cp source destination` | Back up a configuration file |
| `sudo nano file` | Edit a file using nano |
| `sudo nginx -t` | Test Nginx configuration |
| `sudo systemctl enable --now nginx` | Enable and start Nginx |
| `sudo systemctl status nginx` | Check Nginx service status |
| `sudo systemctl is-active httpd` | Check Apache service status |
| `sudo systemctl enable --now httpd` | Enable and start Apache |

## 11. Troubleshooting and Lessons Learned

### Mistake 1: Assuming the Apache port was 3002

Initially, port `3002` was considered, but testing showed that all three Apache servers were actually listening on port `3000`.

Correct backend testing:

```bash
curl -I http://stapp01:3000
curl -I http://stapp02:3000
curl -I http://stapp03:3000
```

**Lesson:** Always verify the actual backend port before configuring a reverse proxy.

### Mistake 2: Confusing Nginx and Apache services

The Load Balancer runs Nginx, while the application servers run Apache.

| Server | Service to check |
|---|---|
| stlb01 | nginx |
| stapp01 | httpd |
| stapp02 | httpd |
| stapp03 | httpd |

Running `sudo systemctl is-active httpd` on `stlb01` can return `inactive`. That alone does not indicate a problem because Apache is not the load-balancer service.

### Mistake 3: Understanding HTTP 200

An HTTP 200 response from an individual backend confirms that the server responded successfully.

However, backend testing and final load-balancer testing are two different checks:

```bash
curl -I http://stapp01:3000
```

Tests one backend.

```bash
curl http://stlb01:80
```

Tests the website through the Nginx load balancer.

---

## Final Result

**Status: Completed**

- [x] Nginx already installed on stlb01.
- [x] All three Apache backends tested on port 3000.
- [x] Backend servers returned HTTP 200 OK.
- [x] Nginx upstream group configured.
- [x] Reverse proxy configured in the existing HTTP server block.
- [x] Only the main Nginx configuration file was edited.
- [x] Nginx configuration test successful.
- [x] Nginx service enabled and running.
- [x] Final `curl http://stlb01:80` returned the website.

**Final successful response:**

```text
Welcome to xFusionCorp Industries!
```

## Key Learning

Nginx can act as a reverse proxy and load balancer. Instead of clients connecting directly to individual application servers, they connect to Nginx. Nginx forwards their requests to the configured backend servers, distributing traffic using its load-balancing algorithm.

This improves traffic distribution and provides a foundation for a highly available application architecture.

**Day 16 – Nginx Load Balancer Configuration: COMPLETED**
