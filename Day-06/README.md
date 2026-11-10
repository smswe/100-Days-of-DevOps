# Day 6 - Installing Cronie and Scheduling Cron Jobs on Multiple Servers

## Task

The Nautilus system administrators wanted to deploy scripts on all application servers in the Stratos Datacenter on a scheduled basis. Before deploying those scripts, they wanted to test the scheduling functionality using a sample cron job.

The requirements were:

1. Install the `cronie` package on all application servers.
2. Start the `crond` service.
3. Add the following cron job for the root user:

```bash
*/5 * * * * echo hello > /tmp/cron_text
```

## Servers Used

| Server | User |
|---|---|
| App Server 1 (`stapp01`) | `tony` |
| App Server 2 (`stapp02`) | `steve` |
| App Server 3 (`stapp03`) | `banner` |

## Step 1: Connecting to the Application Servers

I connected to each application server from the Jump Host using SSH.

```bash
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03
```

SSH allows secure remote access to Linux servers.

## Step 2: Installing the Cronie Package

On each application server, I installed Cronie using:

```bash
sudo dnf install -y cronie
```

**Understanding the command:**

- `sudo` – Runs the command with administrator privileges.
- `dnf` – Package manager used in CentOS Stream.
- `install` – Installs a package.
- `-y` – Automatically confirms the installation.
- `cronie` – Package that provides cron scheduling functionality.

To verify the installation:

```bash
rpm -q cronie
```

This displays the installed Cronie package and its version.

## Step 3: Starting the Cron Service

After installing Cronie, I enabled and started the cron service:

```bash
sudo systemctl enable --now crond
```

**Understanding the command:**

- `systemctl` – Used to manage system services.
- `enable` – Configures the service to start automatically during system boot.
- `--now` – Starts the service immediately.
- `crond` – Background daemon responsible for executing scheduled cron jobs.

To check the service status:

```bash
sudo systemctl status crond
```

I also used:

```bash
sudo systemctl is-active crond
```

Expected output:

```text
active
```

## Step 4: Adding a Cron Job for Root

The task specifically required the cron job to be configured for the root user.

I opened root's crontab using:

```bash
sudo crontab -e
```

I added the following entry:

```bash
*/5 * * * * echo hello > /tmp/cron_text
```

Saved the file and exited the editor.

To verify the cron entry:

```bash
sudo crontab -l
```

This displays the scheduled jobs configured for the root user.

## Step 5: Understanding the Cron Expression

A cron expression contains five time fields followed by the command to execute.

```text
*/5 * * * * echo hello > /tmp/cron_text
```

| Field | Value | Meaning |
|---|---|---|
| Minute | `*/5` | Every 5 minutes |
| Hour | `*` | Every hour |
| Day of month | `*` | Every day of the month |
| Month | `*` | Every month |
| Day of week | `*` | Every day of the week |

### Understanding the command

```bash
echo hello > /tmp/cron_text
```

- `echo hello` – Prints the text `hello`.
- `>` – Redirects the output into a file, replacing its previous contents.
- `/tmp/cron_text` – File where the output is stored.

The cron daemon executes this command every five minutes.

## Step 6: Verification

I used the following commands on each application server to verify the configuration.

**Check package installation:**

```bash
rpm -q cronie
```

**Check whether the cron service is running:**

```bash
sudo systemctl is-active crond
```

**Check root's scheduled cron jobs:**

```bash
sudo crontab -l
```

**Check the output file after the job executes:**

```bash
cat /tmp/cron_text
```

Expected file content:

```text
hello
```

## What I Learned

Through this task, I learned:

1. What Cronie is and why it is needed for scheduling tasks in Linux.
2. The difference between installing a package and starting its service.
3. How to manage Linux services using `systemctl`.
4. How to configure scheduled tasks using `crontab -e`.
5. How to configure a cron job specifically for the root user using `sudo crontab -e`.
6. How cron expressions define the execution schedule.
7. How output redirection (`>`) stores command output in a file.

## Real-World DevOps Application

Cron jobs are useful for automating repetitive administrative activities without requiring someone to execute commands manually.

Some practical examples include:

- Scheduling regular backups.
- Cleaning temporary files.
- Running monitoring scripts.
- Generating periodic reports.
- Performing routine maintenance tasks.

This task helped me understand how Linux scheduling works and how cron jobs can be configured across multiple servers.

**Status:** Completed the Cronie installation, service configuration and root cron job setup on all three application servers.
