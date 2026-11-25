# Day 20 - Configure Nginx with PHP-FPM 8.3

## Original Task / Question

The `Nautilus` application development team is planning to launch a new PHP-based application, which they want to deploy on `Nautilus` infra in `Stratos DC`. The development team had a meeting with the production support team and they have shared some requirements regarding the infrastructure. Below are the requirements they shared:

**a.** Install `nginx` on `app server 2`, configure it to use port `8092` and its document root should be `/var/www/html`.

**b.** Install `php-fpm` version `8.3` on `app server 2`, it must use the unix socket `/var/run/php-fpm/default.sock` (create the parent directories if don't exist).

**c.** Configure php-fpm and nginx to work together.

**d.** Once configured correctly, you can test the website using `curl http://stapp02:8092/index.php` command from jump host.

**NOTE:** We have copied two files, `index.php` and `info.php`, under `/var/www/html` as part of the PHP-based application setup. Please do not modify these files.

---

## 1. Connect to App Server 2

From Jump Host:

```bash
ssh steve@stapp02
```

Verify the server:

```bash
hostname
```

## 2. Install and start Nginx

Check Nginx:

```bash
nginx -v
```

Install if not already installed:

```bash
sudo dnf install -y nginx
```

Enable and start Nginx:

```bash
sudo systemctl enable --now nginx
```

Check service status:

```bash
sudo systemctl status nginx
```

**Explanation:**
- `dnf install` installs Nginx.
- `systemctl enable` enables automatic startup after reboot.
- `--now` starts the service immediately.
- `systemctl status` checks whether the service is running.

## 3. Configure Nginx

Edit the main configuration:

```bash
sudo nano /etc/nginx/nginx.conf
```

Configure the existing server block:

```nginx
server {
    listen       8092;
    listen       [::]:8092;
    server_name  _;
    root         /var/www/html;

    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/var/run/php-fpm/default.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    error_page 404 /404.html;
    location = /404.html {
    }

    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
    }
}
```

Save the file:
- `Ctrl + O` – Save.
- `Enter` – Confirm filename.
- `Ctrl + X` – Exit Nano.

### Configuration explanation

| Directive | Purpose |
|---|---|
| `listen 8092` | Nginx listens on port 8092 |
| `listen [::]:8092` | Enables IPv6 listening on port 8092 |
| `root /var/www/html` | Sets the website document root |
| `index index.php` | Uses index.php as the default page |
| `location /` | Handles normal website requests |
| `location ~ \.php$` | Matches PHP file requests |
| `fastcgi_pass` | Forwards PHP requests to PHP-FPM |
| `SCRIPT_FILENAME` | Provides the PHP script's actual file path |

Nginx cannot execute PHP code by itself. It forwards PHP requests to PHP-FPM through FastCGI.

## 4. Enable PHP 8.3 module

Check available PHP versions:

```bash
sudo dnf module list php
```

The lab provided PHP versions 8.1, 8.2, 8.3 and 8.4.

Enable PHP 8.3:

```bash
sudo dnf module enable php:8.3 -y
```

Install PHP-FPM:

```bash
sudo dnf install -y php-fpm
```

Verify the version:

```bash
php-fpm -v
```

**Actual installed version:**

```text
PHP 8.3.36 (fpm-fcgi)
```

Dependencies installed included:
- `php-common`
- `httpd-filesystem`

## 5. Configure PHP-FPM Unix socket

Open the PHP-FPM pool configuration:

```bash
sudo nano /etc/php-fpm.d/www.conf
```

Find the existing `listen` directive and set:

```ini
listen = /var/run/php-fpm/default.sock
```

Configure socket ownership and permissions:

```ini
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
```

**Explanation:**

- `listen` specifies where PHP-FPM creates its socket.
- `listen.owner` specifies the socket owner.
- `listen.group` specifies the socket group.
- `listen.mode` sets socket access permissions.

The socket path must match the path configured in Nginx.

Nginx configuration:

```nginx
fastcgi_pass unix:/var/run/php-fpm/default.sock;
```

PHP-FPM configuration:

```ini
listen = /var/run/php-fpm/default.sock
```

## 6. Create the parent directory

```bash
sudo mkdir -p /var/run/php-fpm
```

`-p` creates missing parent directories and does not report an error if the directory already exists.

## 7. Enable and start PHP-FPM

```bash
sudo systemctl enable --now php-fpm
```

Check status:

```bash
sudo systemctl status php-fpm
```

Actual result:

```text
Active: active (running)
Status: "Ready to handle connections"
```

Verify the socket:

```bash
ls -l /var/run/php-fpm/default.sock
```

Observed output:

```text
srw-rw----+ 1 root root 0 ... /var/run/php-fpm/default.sock
```

The `s` at the beginning indicates a Unix socket.

## 8. Validate Nginx configuration

```bash
sudo nginx -t
```

Actual output:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Restart Nginx:

```bash
sudo systemctl restart nginx
```

Restart PHP-FPM:

```bash
sudo systemctl restart php-fpm
```

Check both services:

```bash
sudo systemctl status nginx
sudo systemctl status php-fpm
```

Both services showed:

```text
Active: active (running)
```

## 9. Test the application from Jump Host

The application files were already present in:

```text
/var/www/html/index.php
/var/www/html/info.php
```

We did not modify either file.

Exit App Server 2:

```bash
exit
```

From the Jump Host, run the required test:

```bash
curl http://stapp02:8092/index.php
```

**Actual output:**

```text
Welcome to xFusionCorp Industries!
```

This confirms that the PHP application endpoint returned the expected content.

---

## 10. Nano editor shortcuts

| Shortcut | Purpose |
|---|---|
| `Ctrl + O` | Save file |
| `Enter` | Confirm filename |
| `Ctrl + X` | Exit Nano |
| `Ctrl + K` | Cut current line |
| `Ctrl + U` | Paste cut text |
| `Ctrl + 6` | Start text selection |
| `Ctrl + W` | Search text |
| `Ctrl + \` | Search and replace |

In browser-based terminals, some shortcuts may be intercepted by the browser. If Nano search does not work with `Ctrl + W`, try `F6` or check the terminal keyboard shortcut settings.

---

## 11. Important DevOps concepts learned

| Concept | Real-world usage |
|---|---|
| Nginx | Web server and reverse proxy |
| PHP-FPM | Processes PHP application requests |
| FastCGI | Communication protocol between Nginx and PHP-FPM |
| Unix socket | Local communication between processes |
| Document root | Directory containing website files |
| `systemctl` | Manages Linux services |
| `nginx -t` | Validates Nginx configuration |
| `ss -tulnp` | Checks listening network ports |
| `journalctl` | Troubleshoots service logs |
| `curl` | Tests application endpoints |
| `dnf module` | Selects a package version stream |

### Architecture

```text
Jump Host
    |
    | curl http://stapp02:8092/index.php
    |
    v
Nginx - App Server 2
    |
    | Port 8092
    | Document root: /var/www/html
    |
    | FastCGI request
    v
PHP-FPM 8.3
    |
    | Unix socket:
    | /var/run/php-fpm/default.sock
    |
    v
Executes index.php
    |
    v
Returns PHP response through Nginx
```

---

## 12. Completion checklist

- [x] Nginx installed on App Server 2.
- [x] Nginx configured on port 8092.
- [x] Document root configured as `/var/www/html`.
- [x] PHP-FPM 8.3 installed.
- [x] PHP-FPM configured to use `/var/run/php-fpm/default.sock`.
- [x] Parent directory created.
- [x] Nginx and PHP-FPM services running.
- [x] Nginx configuration validated.
- [x] Application tested from Jump Host using curl.
- [x] Existing `index.php` and `info.php` files
