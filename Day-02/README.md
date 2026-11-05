# Day 02 – Linux User Management: Account Expiry

## Challenge Question

A Linux user named `john` needs to be created on App Server 2 within the Stratos Datacenter. Set the account expiry date to **17 February 2027**.

## 1. Server Details

- Server: App Server 2
- Hostname: `stapp02`
- Username used to connect: `steve`
- New user to create: `john`
- Account expiry date: `2027-02-17`

## 2. Connect to App Server 2

From the Jump Host:

```bash
ssh steve@stapp02
```

SSH (Secure Shell) provides secure remote access to another Linux server.

## 3. Create a User with an Expiry Date

```bash
sudo useradd -e 2027-02-17 john
```

### Command Breakdown

- `sudo` – Executes the command with administrator privileges.
- `useradd` – Creates a new Linux user.
- `-e` – Specifies the account expiry date.
- `2027-02-17` – Expiry date in YYYY-MM-DD format.
- `john` – Username being created.

## 4. Verify User Creation

Check whether the user exists:

```bash
getent passwd john
```

Check the account expiry details:

```bash
sudo chage -l john
```

Look for:

```text
Account expires : Feb 17, 2027
```

This confirms that the account expiry date has been configured.

## 5. Important Linux Concepts

| Command / Option | Explanation |
|---|---|
| `useradd` | Creates a Linux user |
| `-e` | Sets account expiry date |
| `getent passwd` | Retrieves user account information |
| `chage -l` | Displays account ageing information |
| `YYYY-MM-DD` | Standard date format used here |

## 6. What Happens When an Account Expires?

When the expiry date is reached, the user cannot normally log in.

Important points:
- The user account is not automatically deleted.
- The user's files remain on the system.
- An administrator can change the expiry date if required.

## 7. Real-World DevOps Connection

In real-world environments, temporary accounts are often created for contractors, external support teams and short-term project members.

Setting account expiry dates helps administrators automatically restrict access after the required period, improving server security.

## 8. Final Outcome

- Connected to App Server 2 using SSH.
- Created the Linux user `john`.
- Configured the account expiry date as `17 February 2027`.
- Verified the account using `getent passwd`.
- Confirmed the expiry date using `chage -l`.

**Status: Completed ✓**

### Key Learning

Learned how to create Linux users with an account expiry date and how to verify account ageing information using the `chage` command.
