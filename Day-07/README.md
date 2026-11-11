# Day 07 - Configure Password-less SSH Authentication Between Jump Host and App Servers

## Task

The system administrators at xFusionCorp Industries had configured scripts on the Jump Host to perform regular operations on all application servers in the Stratos Datacenter.

For these scripts to work properly, the `thor` user on the Jump Host needed password-less SSH access to all application servers through their respective sudo users.

The requirements were:

1. Configure password-less SSH authentication from `thor` on the Jump Host.
2. Provide access to App Server 1 through `tony`.
3. Provide access to App Server 2 through `steve`.
4. Provide access to App Server 3 through `banner`.

## Servers and Users

| Server | SSH User |
|---|---|
| Jump Host | `thor` |
| App Server 1 (`stapp01`) | `tony` |
| App Server 2 (`stapp02`) | `steve` |
| App Server 3 (`stapp03`) | `banner` |

## Step 1: Verify the Jump Host User

First, I checked that I was logged in as `thor` on the Jump Host.

```bash
whoami
pwd
```

Output:

```text
thor
/home/thor
```

I checked whether the SSH directory already existed:

```bash
ls -la ~/.ssh
```

Initially, the `.ssh` directory did not exist.

### Understanding `~/.ssh`

- `~` represents the current user's home directory.
- `.ssh` is a hidden directory used to store SSH-related files.
- For the `thor` user, the full path is `/home/thor/.ssh`.

## Step 2: Generate an SSH Key Pair

I generated an RSA key pair using:

```bash
ssh-keygen -t rsa
```

I accepted the default file location:

```text
/home/thor/.ssh/id_rsa
```

I left the passphrase empty by pressing Enter twice.

The command generated two files:

| File | Purpose |
|---|---|
| `id_rsa` | Private key – stays on the Jump Host |
| `id_rsa.pub` | Public key – copied to remote servers |

### Understanding the command

- `ssh-keygen` – Generates SSH keys.
- `-t` – Specifies the key type.
- `rsa` – Specifies the RSA algorithm.

The private key must be kept secure and should never be shared.

## Step 3: Copy the Public Key to App Server 1

I used `ssh-copy-id` to install the public key on App Server 1.

```bash
ssh-copy-id tony@stapp01
```

During the first connection, I accepted the server's authenticity prompt by entering `yes` and provided Tony's password when requested.

Successful output:

```text
Number of key(s) added: 1
```

This confirmed that the public key was successfully added to Tony's `authorized_keys` file.

## Step 4: Configure App Server 2

I repeated the same process for App Server 2:

```bash
ssh-copy-id steve@stapp02
```

I authenticated with Steve's password when prompted.

## Step 5: Configure App Server 3

I copied the public key to App Server 3:

```bash
ssh-copy-id banner@stapp03
```

I authenticated with Banner's password when prompted.

## Step 6: Test Password-less SSH Authentication

After copying the public key to all three servers, I tested the SSH connections:

```bash
ssh tony@stapp01
ssh steve@stapp02
ssh banner@stapp03
```

Each connection should log in without asking for the remote user's account password.

To verify the connected user and server:

```bash
whoami
hostname
```

Expected results:

| Connection | `whoami` | `hostname` |
|---|---|---|
| `tony@stapp01` | `tony` | `stapp01` |
| `steve@stapp02` | `steve` | `stapp02` |
| `banner@stapp03` | `banner` | `stapp03` |

## Understanding How Password-less SSH Works

SSH key-based authentication uses a public and private key pair.

1. The private key remains on the Jump Host.
2. The public key is copied to the remote user's `authorized_keys` file.
3. When Thor connects to an application server, SSH uses the private key to prove his identity.
4. The remote server verifies the authentication using the corresponding public key.
5. If authentication succeeds, access is granted without entering the account password.

### Important SSH Files

**On the Jump Host:**

```text
/home/thor/.ssh/id_rsa
/home/thor/.ssh/id_rsa.pub
```

**On App Server 1:**

```text
/home/tony/.ssh/authorized_keys
```

**On App Server 2:**

```text
/home/steve/.ssh/authorized_keys
```

**On App Server 3:**

```text
/home/banner/.ssh/authorized_keys
```

## Difference Between Important SSH Commands

| Command | Purpose |
|---|---|
| `ssh-keygen -t rsa` | Generates an SSH key pair |
| `ssh-copy-id user@server` | Copies the public key to a remote user |
| `ssh user@server` | Connects to a remote server |
| `ls -la ~/.ssh` | Lists files in the SSH directory |
| `cat ~/.ssh/id_rsa.pub` | Displays the public key |

## What I Learned

Through this task, I learned:

1. What SSH key-based authentication is.
2. How to generate public and private keys using `ssh-keygen`.
3. The difference between public and private SSH keys.
4. How to use `ssh-copy-id` to configure password-less authentication.
5. What the `authorized_keys` file does.
6. How to establish secure connections between multiple Linux servers.
7. How password-less SSH helps automate administrative tasks.

## Real-World DevOps Application

Password-less SSH authentication is useful when managing multiple servers and automating repetitive operations.

For example:

- Executing scripts remotely across multiple servers.
- Automating deployments.
- Running configuration management tools.
- Performing scheduled maintenance.
- Managing servers using Ansible and other automation tools.

Instead of manually entering passwords for every connection, SSH key-based authentication allows trusted automation processes to connect securely.

**Status:** Configured SSH key-based password-less authentication from `thor` on the Jump Host to all three application servers using their respective users.
