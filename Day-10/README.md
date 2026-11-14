# Day 10 - Bash Scripting: Archive Website Media Files and Transfer to Storage Server

## Objective

As part of the KodeKloud 100 Days of DevOps Challenge, I worked on automating the backup of website media files using a Bash script.

The production support team at **xFusionCorp Industries** wanted to automate the process of archiving static website content on App Server 1 and transferring the archive to the Nautilus Storage Server for later validation.

This task helped me understand Bash scripting, ZIP archives, SSH key-based authentication, SCP file transfers, Linux permissions, and basic automation.

---

## 1. Task Description

The production support team needed a Bash script named `media_archive.sh` on **App Server 1** in the Stratos Datacenter.

The script had to perform the following tasks:

1. Create a ZIP archive named `xfusioncorp_media.zip` from `/var/www/html/media`.
2. Store the archive in `/archives/` on App Server 1.
3. Copy the same archive to the Nautilus Storage Server (`ststor01`) under `/archives/`.
4. Configure passwordless SSH authentication from the App Server 1 user (`tony`) to the Storage Server user (`natasha`).
5. Ensure the script runs as `tony` without asking for a password.
6. Do not use `sudo` inside the Bash script.

**Important:** The `zip` package had to be installed manually on App Server 1 before executing the script.

---

## 2. Server Details

| Server | Hostname | User | Purpose |
|---|---|---|---|
| Jump Host | jump-host | thor | Entry point to the lab servers |
| App Server 1 | stapp01 | tony | Creates the ZIP archive and runs the script |
| Storage Server | ststor01 | natasha | Receives and stores the archive |

### Required file locations

**App Server 1**

- Website media directory: `/var/www/html/media`
- Script location: `/scripts/media_archive.sh`
- Local archive: `/archives/xfusioncorp_media.zip`

**Storage Server**

- Remote archive: `/archives/xfusioncorp_media.zip`

---

## 3. Understanding the Workflow

The automation process follows this sequence:

1. Connect to App Server 1 from the Jump Host.
2. Install the ZIP package.
3. Configure passwordless SSH from `tony` to `natasha`.
4. Create the Bash script.
5. Make the script executable.
6. Run the script as `tony`.
7. Create the ZIP archive locally.
8. Transfer the archive to the Storage Server using SCP.
9. Verify the archive exists on both servers.

### Overall architecture

```text
                 Jump Host
                thor@jump-host
                      |
                      | SSH
                      v
          App Server 1 (stapp01)
                  User: tony
                      |
          +-----------+------------+
          |                        |
          v                        v
  Website Media Directory     Bash Script
  /var/www/html/media         /scripts/media_archive.sh
                                   |
                                   |
                            Create ZIP Archive
                                   |
                                   v
                    /archives/xfusioncorp_media.zip
                                   |
                                   |
                             SCP Transfer
                          Passwordless SSH
                                   |
                                   v
                   Storage Server (ststor01)
                         User: natasha
                                   |
                                   v
                    /archives/xfusioncorp_media.zip
```

---

## 4. Step 1 - Connect to App Server 1

From the Jump Host, I connected to App Server 1 using Tony's account.

```bash
ssh tony@stapp01
```

Check the current user and hostname:

```bash
whoami
hostname
```

Expected output:

```text
tony
stapp01
```

**Why is this important?**

The task requires the script to execute as `tony`, not as root or another user.

The SSH key must also belong to Tony because the script will use his SSH credentials when transferring the archive.

---

## 5. Step 2 - Check and Install the ZIP Package

Before creating an archive, I checked whether the ZIP utility was installed.

```bash
zip -v
```

The ZIP package was available on App Server 1.

If it is not installed, install it manually:

```bash
sudo dnf install -y zip
```

Depending on the server's package manager, `yum` can also be used:

```bash
sudo yum install -y zip
```

### What is ZIP?

ZIP is a file compression and archiving utility.

It allows multiple files and directories to be combined into one archive file.

For example:

```text
media/
├── index.html
├── image1.jpg
├── image2.png
└── video.mp4
```

These files can be packaged into:

```text
xfusioncorp_media.zip
```

### Understanding the ZIP command

```bash
zip -r archive.zip directory/
```

- `zip` - Runs the ZIP utility.
- `-r` - Recursively includes files and subdirectories.
- `archive.zip` - Name of the archive to create.
- `directory/` - Directory whose contents should be archived.

**Important:** The ZIP package was installed outside the script, as required by the task.

---

## 6. Step 3 - Understand Passwordless SSH Authentication

The script must copy the archive to the Storage Server without asking for Natasha's password.

To achieve this, I configured SSH key-based authentication from:

```text
tony@stapp01
       |
       v
natasha@ststor01
```

### What is SSH?

SSH stands for **Secure Shell**.

It is a network protocol used to securely connect to another machine over a network.

Example:

```bash
ssh natasha@ststor01
```

This connects to the Storage Server using Natasha's account.

### What is SSH key-based authentication?

SSH key-based authentication uses two related cryptographic keys:

1. Private key - Kept securely on the client machine.
2. Public key - Installed on the remote server.

The private key must never be shared.

The public key can be copied to the remote server.

When Tony connects to Natasha's account, SSH uses the matching keys to authenticate the connection without requiring Natasha's account password.

### Step 3.1 - Generate an SSH key pair

While logged in as `tony` on App Server 1:

```bash
ssh-keygen -t rsa
```

I accepted the default key location:

```text
/home/tony/.ssh/id_rsa
```

I left the passphrase empty by pressing Enter twice. This allows the automated script to use the key without requiring an interactive passphrase.

The key pair was generated successfully.

**Files created:**

```text
/home/tony/.ssh/id_rsa
/home/tony/.ssh/id_rsa.pub
```

| File | Purpose |
|---|---|
| `id_rsa` | Private key; must remain on App Server 1 |
| `id_rsa.pub` | Public key; copied to the Storage Server |

### Step 3.2 - Copy the public key to the Storage Server

I ran:

```bash
ssh-copy-id natasha@ststor01
```

The first connection displayed a host authenticity prompt.

```text
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

I entered:

```text
yes
```

Then I entered Natasha's password once.

The command returned:

```text
Number of key(s) added: 1
```

This confirmed that Tony's public key was installed on Natasha's account.

The public key is stored in the remote user's:

```text
~/.ssh/authorized_keys
```

For Natasha, this normally resolves to:

```text
/home/natasha/.ssh/authorized_keys
```

### Step 3.3 - Test passwordless SSH

I tested the connection:

```bash
ssh natasha@ststor01
```

The connection was successful without asking for Natasha's password.

Then I returned to App Server 1:

```bash
exit
```

**Result:** Passwordless SSH authentication was successfully configured from `tony@stapp01` to `natasha@ststor01`.

### Important learning

The key must be generated by the user who runs the script.

Generating the key on the Jump Host as `thor` would not satisfy this requirement because the script runs as `tony` on App Server 1.

---

## 7. Step 4 - Check Required Directories

Before creating the script, I checked the required directories.

```bash
ls -ld /scripts
ls -ld /archives
ls -ld /var/www/html/media
```

The directories were present and accessible.

The observed permissions included:

```text
/scripts                  777
/archives                 777
/var/www/html/media       755
```

### Understanding Linux permissions

Linux permissions determine who can read, write, or execute files and directories.

| Permission | Meaning |
|---|---|
| `r` | Read |
| `w` | Write |
| `x` | Execute |

For directories:

- Read allows listing directory contents.
- Write allows creating, deleting, or renaming entries, subject to other permissions.
- Execute allows accessing or traversing the directory.

In this task, Tony needed:

- Read access to the website media directory.
- Write access to `/archives`.
- Write access to `/scripts`.

---

## 8. Step 5 - Create the Bash Script

The task required the script to be located at:

```text
/scripts/media_archive.sh
```

I created the file using:

```bash
vi /scripts/media_archive.sh
```

I used `vi` because `nano` was not installed on the App Server.

### Final script

```bash
#!/bin/bash

set -e

cd /var/www/html

rm -f /archives/xfusioncorp_media.zip

zip -r /archives/xfusioncorp_media.zip media

scp -o BatchMode=yes /archives/xfusioncorp_media.zip natasha@ststor01:/archives
```

### Explanation of every line

#### Line 1

```bash
#!/bin/bash
```

This is called the shebang.

It tells Linux to execute the script using the Bash shell.

#### Line 2

```bash
set -e
```

This tells Bash to exit when a command fails with a non-zero exit status.

Why is it useful?

Suppose ZIP creation fails. Without error handling, the script might continue and attempt to copy an archive that was never created.

With `set -e`, execution stops when a command fails.

#### Line 3

```bash
cd /var/www/html
```

`cd` means change directory.

This moves the working directory to:

```text
/var/www/html
```

The website media directory is inside this location.

#### Line 4

```bash
rm -f /archives/xfusioncorp_media.zip
```

`rm` removes a file.

`-f` means force:

- Do not ask for confirmation.
- Do not report an error if the file does not exist.

This removes an older archive so that the script creates a fresh one each time.

#### Line 5

```bash
zip -r /archives/xfusioncorp_media.zip media
```

This creates the ZIP archive.

Breakdown:

| Part | Meaning |
|---|---|
| `zip` | ZIP utility |
| `-r` | Include directory contents recursively |
| `/archives/xfusioncorp_media.zip` | Destination archive |
| `media` | Source directory |

Because the current directory is `/var/www/html`, the command archives:

```text
/var/www/html/media
```

The archive contains the `media` directory and its contents.

#### Line 6

```bash
scp -o BatchMode=yes /archives/xfusioncorp_media.zip natasha@ststor01:/archives
```

This transfers the ZIP archive to the Storage Server.

Breakdown:

| Part | Meaning |
|---|---|
| `scp` | Secure Copy utility |
| `-o` | Pass an option to SSH |
| `BatchMode=yes` | Disable interactive password/passphrase prompts |
| `/archives/xfusioncorp_media.zip` | Source archive |
| `natasha@ststor01` | Remote username and hostname |
| `:/archives` | Destination directory on the remote server |

The archive is copied to:

```text
ststor01:/archives/xfusioncorp_media.zip
```

**Why use `BatchMode=yes`?**

The script must run automatically without waiting for a password.

If key-based authentication is not working, BatchMode makes the SSH connection fail instead of prompting for a password.

---

## 9. Step 6 - Make the Script Executable

After saving the script, I added execute permission:

```bash
chmod +x /scripts/media_archive.sh
```

### Understanding chmod

`chmod` means change mode.

It is used to modify file permissions.

- `+` means add a permission.
- `x` means execute.
- `+x` adds execute permission.

I verified the permissions:

```bash
ls -l /scripts/media_archive.sh
```

Output:

```text
-rwxr-xr-x 1 tony tony 199 Nov 14 18:06 /scripts/media_archive.sh
```

The `x` permissions indicate that the script is executable.

---

## 10. Step 7 - Execute the Script

I ran the script as Tony:

```bash
/scripts/media_archive.sh
```

I did not use `sudo`.

The script executed successfully.

### Actual execution output

```text
adding: media/ (stored 0%)
adding: media/.gitkeep (stored 0%)
adding: media/index.html (stored 0%)

xfusioncorp_media.zip 100% 517 1.1MB/s 00:00
```

### Understanding the output

The ZIP utility added the following entries:

```text
media/
media/.gitkeep
media/index.html
```

The output confirms that the media directory and its files were included in the archive.

The SCP output showed:

```text
xfusioncorp_media.zip 100%
```

This indicates that the file transfer completed.

---

## 11. Step 8 - Verify the Archive on App Server 1

I checked whether the ZIP archive existed locally.

Command:

```bash
ls -lh /archives/xfusioncorp_media.zip
```

Actual output:

```text
-rw-r--r-- 1 tony tony 517 Nov 14 18:11 /archives/xfusioncorp_media.zip
```

This confirmed that the archive was created successfully on App Server 1.

### Understanding the output

| Field | Meaning |
|---|---|
| `-rw-r--r--` | File permissions |
| `1` | Number of hard links |
| `tony` | File owner |
| `tony` | File group |
| `517` | File size in bytes |
| `Nov 14 18:11` | Last modification time |
| `xfusioncorp_media.zip` | Archive filename |

---

## 12. Step 9 - Verify the Archive on the Storage Server

Next, I verified that the archive was transferred to the Storage Server.

Command:

```bash
ssh natasha@ststor01 'ls -lh /archives/xfusioncorp_media.zip'
```

Actual output:

```text
-rw-r--r-- 1 natasha natasha 517 Nov 14 18:11 /archives/xfusioncorp_media.zip
```

This confirmed that the archive was present on the Storage Server.

### Final verification

| Requirement | Location | Result |
|---|---|---|
| Create ZIP archive | App Server 1 | Completed |
| Save local archive | `/archives/` on App Server 1 | Verified |
| Configure passwordless SSH | Tony to Natasha | Verified |
| Transfer archive using SCP | App Server 1 to Storage Server | Completed |
| Save remote archive | `/archives/` on Storage Server | Verified |
| Execute without sudo | App Server 1 as Tony | Completed |

Both servers contained the archive:

```text
App Server 1:
 /archives/xfusioncorp_media.zip

Storage Server:
 /archives/xfusioncorp_media.zip
```

Both files showed a size of 517 bytes.

---

## 13. Troubleshooting and Mistakes I Encountered

This task also helped me understand some common Linux command mistakes.

### Issue 1 - Incorrect SSH hostname

I initially typed:

```bash
ssh tony@atapp01
```

The hostname was incorrect.

Correct command:

```bash
ssh tony@stapp01
```

**Learning:** Always verify hostnames before connecting to remote servers.

### Issue 2 - Nano was not installed

I tried:

```bash
nano /scripts/media_archive.sh
```

Output:

```text
-bash: nano: command not found
```

I used `vi` instead:

```bash
vi /scripts/media_archive.sh
```

**Learning:** Not every Linux server has the same text editors installed. Knowing how to use `vi` is useful when working with minimal server environments.

### Issue 3 - Missing space in chmod

Incorrect command:

```bash
chmod+x /scripts/media_archive.sh
```

Correct command:

```bash
chmod +x /scripts/media_archive.sh
```

**Learning:** Linux commands require correct spacing between the command, options, and arguments.

### Issue 4 - Incorrect file path

I accidentally entered:

```bash
ls -l /scripts/media)archive.sh
```

and:

```bash
ls -l /scripts/media/archive.sh
```

The correct path was:

```bash
ls -l /scripts/media_archive.sh
```

**Learning:** An underscore `_` in a filename is different from a slash `/`, which separates directories.

### Issue 5 - Archive did not exist before script execution

Initially, I checked:

```bash
ls -lh /archives/xfusioncorp_media.zip
```

The output showed:

```text
No such file or directory
```

This was expected because the script had not yet been executed successfully.

After running:

```bash
/scripts/media_archive.sh
```

the archive was created and transferred successfully.

**Learning:** Creating a script does not automatically execute it. The script must be run to perform its tasks.

### Issue 6 - SSH host key verification

When I attempted to connect from App Server 1 to App Server 2, SSH displayed a host-key verification error because the host key was associated with another hostname in the known-hosts file.

This connection was not required for Day 10. The actual task only needed App Server 1 and the Storage Server.

**Learning:** Understand which servers are involved in a task before making unnecessary connections.

---

## 14. Important Linux and DevOps Concepts Learned

### 1. Bash scripting

Bash scripting allows us to combine Linux commands into one executable file.

Instead of manually running several commands, we can automate them through a script.

### 2. ZIP archiving

ZIP combines files and directories into one archive.

Useful for backups, file transfers, and storing website content.

### 3. SSH

SSH provides secure remote access to another Linux server.

### 4. SSH key-based authentication

Uses a public/private key pair to authenticate without repeatedly entering a password.

### 5. SCP

SCP securely copies files between systems over SSH.

Example:

```bash
scp file.txt user@server:/destination/
```

### 6. chmod

Changes file permissions.

Example:

```bash
chmod +x script.sh
```

### 7. rm

Removes files or directories.

Example:

```bash
rm -f old_archive.zip
```

### 8. set -e

Stops script execution when a command fails.

### 9. BatchMode

Prevents SSH from prompting for interactive credentials, which is useful in automation.

### 10. Linux file permissions

Understanding read, write, and execute permissions is important when scripts need to access files and directories.

---

## 15. Real-World Application

This task represents a simple automated backup workflow.

In a real production environment, website media files may contain images, documents, videos, and other static content.

Instead of manually compressing and transferring files, a Bash script can automate the process.

A similar workflow could be used for:

- Website backups.
- Application log archiving.
- Scheduled file transfers.
- Moving backup files to a central storage server.
- Retaining copies for auditing and validation.
- Automating repetitive server administration tasks.

For production use, additional improvements could include timestamps in archive filenames, backup retention policies, logging, error notifications, and checksum validation.

---

## 16. Key Commands Reference

| Command | Purpose |
|---|---|
| `ssh tony@stapp01` | Connect to App Server 1 |
| `whoami` | Display current user |
| `hostname` | Display server hostname |
| `zip -v` | Check ZIP utility |
| `sudo dnf install -y zip` | Install ZIP package |
| `ssh-keygen -t rsa` | Generate SSH key pair |
| `ssh-copy-id natasha@ststor01` | Copy public key to Storage Server |
| `ssh natasha@ststor01` | Test SSH connection |
| `ls -ld /archives` | Check directory permissions |
| `vi /scripts/media_archive.sh` | Create or edit the script |
| `chmod +x /scripts/media_archive.sh` | Make script executable |
| `cat /scripts/media_archive.sh` | Display script contents |
| `/scripts/media_archive.sh` | Execute the script |
| `ls -lh /archives/xfusioncorp_media.zip` | Verify local archive |
| `ssh natasha@ststor01 'ls -lh /archives/xfusioncorp_media.zip'` | Verify remote archive |

---

## 17. Final Outcome

**Day 10 completed successfully!**

I created a Bash script named `media_archive.sh` on App Server 1 that:

- Creates a ZIP archive of the website media directory.
- Stores the archive locally under `/archives/`.
- Uses passwordless SSH authentication.
- Transfers the archive securely to the Nautilus Storage Server using SCP.
- Runs as the required user `tony`, without using sudo inside the script.

This task helped me understand how Bash scripting, SSH, SCP, Linux permissions, and archiving utilities work together to automate a real-world server administration task.

**Main takeaway:** Automation is not just about writing commands in a script. It is also about configuring the right permissions, authentication, file paths, and server access so the entire process works without manual intervention.
