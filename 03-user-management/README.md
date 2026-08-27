# User & Group Management in Linux

## 👤 Introduction to Linux Identity Management

Linux is a true **multi-user operating system**. This means multiple people can log in and run programs on the exact same computer simultaneously without interfering with one another. To keep the system secure, Linux assigns every person an identity and controls exactly what files and commands they are allowed to touch.

The entire user infrastructure relies on four critical system configuration files located in the `/etc` directory:

*   **`/etc/passwd`** – The master directory of user accounts. It stores public details like usernames, numerical User IDs (UIDs), primary Group IDs (GIDs), home directory paths, and default login shells. Contrary to its historic name, it does *not* contain passwords.
*   **`/etc/shadow`** – A highly secure file readable *only* by the root administrator. It contains the actual encrypted (hashed) user passwords along with password security expiration dates.
*   **`/etc/group`** – Defines groups, which are collections of users clustered together to share file access permissions.
*   **`/etc/gshadow`** – Stores secure administrative information and group-specific passwords.

---

## 🆕 Creating Users in Linux

There are two primary tools used to build a new user account. Understanding the difference between them prevents common setup mistakes.

### 1. The `useradd` Command (Low-Level / All Distros)
`useradd` is a low-level utility built into every Linux system. By default, running it raw creates a "hidden" user profile that lacks a workspace or a password. To make a standard human user account, you must explicitly pass flags:

*   **Create user with a home workspace directory:**
    ```bash
    useradd -m username
    ```
*   **Specify a default command shell (like Bash):**
    ```bash
    useradd -m -s /bin/bash username
    ```

### 2. The `adduser` Command (High-Level / Debian & Ubuntu)
`adduser` is a beginner-friendly script wrapped around the raw `useradd` tool. It handles the manual steps for you automatically. When you run it, the system starts a friendly interactive conversation asking you to type a password, confirm full names, and construct a home directory automatically:
```bash
adduser username
```
*(Note: If you are practicing inside a minimal Docker container, `adduser` might not be pre-installed. Stick to `useradd -m -s /bin/bash` or install it manually).*

---

## 🔒 Managing User Passwords & Security Policies

Creating an account does not activate it. You must secure it by assigning or modifying passwords.

### Setting or Changing a Password
To set a brand new password or change an old one, type the command followed by the username. *(If you drop the username, it changes your own password)*:
```bash
sudo passwd username
```

### Enforcing Enterprise Password Policies
*   **Set Password Expiration:** Force a user to change their password every 90 days to prevent credentials from growing stale:
    ```bash
    sudo chage -M 90 username
    ```
*   **Lock an Account Instantly:** Temporarily block a user from logging in without destroying their files (ideal for security incidents or offboarding):
    ```bash
    sudo passwd -l username
    ```
*   **Unlock an Account:** Restore full login permissions to a locked profile:
    ```bash
    sudo passwd -u username
    ```

---

## ✏️ Modifying Existing Users

You can safely reconfigure user properties on the fly using the `usermod` (User Modify) utility.

*   **Change a Username:** Swap an old username for a new one:
    ```bash
    sudo usermod -l new_username old_username
    ```
*   **Change Home Directory Path:** Move a user's home folder location and safely copy over all their existing personal files (`-m` ensures files are migrated):
    ```bash
    sudo usermod -d /home/new_home_directory -m username
    ```
*   **Change Default Shell:** Swap a user's working terminal framework (e.g., upgrading from standard Bash to Zsh):
    ```bash
    sudo usermod -s /bin/zsh username
    ```

---

## ❌ Deleting Users Safely

When an account is no longer needed, you must decide what happens to their personal documents.

*   **Delete the user profile but KEEP their files:** Deletes login privileges, but leaves their home directory intact so an administrator can audit their work later:
    ```bash
    sudo userdel username
    ```
*   **Delete the user AND completely wipe their files:** Purges the user account and destroys their home workspace folder and mailbox completely:
    ```bash
    sudo userdel -r username
    ```

---

## 👥 Managing Collaborative Groups

Groups make it simple to manage security permissions for teams. Instead of changing file settings for 50 separate engineers, you grant access to a single `devs` group and place those users inside it.

### 1. Creating a New Group
```bash
sudo groupadd groupname
```

### 2. Adding Users to a Group (The Append Rule)
> ⚠️ **Critical Alert for Beginners:** When modifying group memberships, you **must** include the `-a` (append) flag. Running `usermod -G groupname username` without `-a` will violently kick the user out of every single secondary group they belong to, which can break their system permissions. Always link them together as `-aG`:

```bash
sudo usermod -aG groupname username
```

### 3. Checking Group Memberships
To view exactly what groups a specific user belongs to, run:
```bash
groups username
id username
```

### 4. Changing a User's Primary Group
Every user has exactly one "Primary Group" linked directly to their login profile (assigned automatically upon account creation). Any new file that user builds will automatically belong to this primary group. To change it:
```bash
sudo usermod -g new_primary_group username
```

---

## 👑 Sudo Privileges & Administrative Access

By default, normal users are isolated inside their personal sandbox and blocked from running commands that alter the core OS. To let a trusted user run administrative tasks, you grant them **Sudo** (`superuser do`) access.

### Adding a User to the Admin Group
Linux operating systems look for a specific administrative group. Anyone inside this group can use the `sudo` prefix to execute commands as the system root administrator.

*   **On Debian / Ubuntu systems:** The admin cluster is named `sudo`:
    ```bash
    sudo usermod -aG sudo username
    ```
*   **On RHEL / Fedora / CentOS systems:** The admin cluster is named `wheel`:
    ```bash
    sudo usermod -aG wheel username
    ```

### Fine-Grained Controls: Granting Specific Commands
Sometimes you want to let a user run *one specific tool* as an administrator without giving them complete control over the entire operating system. 

1. Never open or edit the configuration files manually with standard text editors. Always use the safe built-in editor utility, which checks your file for syntax errors before saving to prevent lockouts:
   ```bash
   sudo visudo
   ```
2. Scroll to the bottom and append a rule. For example, to allow a user named `alex` to restart the Nginx web server without being forced to type their password every time:
   ```plaintext
   alex ALL=(ALL) NOPASSWD: /usr/sbin/systemctl restart nginx
   ```
