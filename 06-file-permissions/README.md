# File Permissions Management in Linux

## Introduction to File Permissions
Linux file permissions determine who can read, write, or execute files and directories. Each file and directory has three levels of permission:
- **Owner (User)**: The creator of the file.
- **Group**: Users belonging to the assigned group.
- **Others**: All other users on the system.

Permissions are represented as:
- **Read (`r` or `4`)** – View file contents.
- **Write (`w` or `2`)** – Modify file contents.
- **Execute (`x` or `1`)** – Run scripts or programs.

To check file permissions, use:
```bash
ls -l filename
```
Output example:
```bash
-rwxr--r-- 1 user group 1234 Mar 28 10:00 myfile.sh
```

## Changing Permissions with `chmod`
### Using Symbolic Mode
Modify permissions using symbols:
- Add (`+`), remove (`-`), or set (`=`) permissions.

Examples:
```bash
chmod u+x filename  # Add execute for user
chmod g-w filename  # Remove write for group
chmod o=r filename  # Set read-only for others
chmod u=rwx,g=rx,o= filename  # Set full access for user, read/execute for group, and no access for others
```

### Using Numeric (Octal) Mode
Each permission has a value:
- Read (`4`), Write (`2`), Execute (`1`).

Examples:
```bash
chmod 755 filename  # User (rwx), Group (r-x), Others (r-x)
chmod 644 filename  # User (rw-), Group (r--), Others (r--)
chmod 700 filename  # User (rwx), No access for others
```

## Changing Ownership with `chown`
Modify file owner and group:
```bash
chown newuser filename  # Change owner
chown newuser:newgroup filename  # Change owner and group
chown :newgroup filename  # Change only group
```

Recursively change ownership:
```bash
chown -R newuser:newgroup directory/
```

## Changing Group Ownership with `chgrp`
```bash
chgrp newgroup filename  # Change group
chgrp -R newgroup directory/  # Change group recursively
```

# Execute ("x") Permission on Linux Directories

For a directory, **"x"** means **traverse/search permission**. It allows you to enter the directory, pass through it in a path, and access entries inside it, provided you have the necessary permissions on those entries.

## Example Scenario

### Directory Structure
```text
mydir/
└── file.txt
```

### Running the Command
To execute this command:
```bash
cat mydir/file.txt
```

You need the following permissions:
* **`mydir/`** → `x` (execute/traverse)
* **`file.txt`** → `r` (read)

The directory's **"x"** permission lets you reach `file.txt`. The file's **"r"** permission lets you read its contents.

---

## "r" vs "x" on a Directory

* **`r`** → Read the directory's list of names.
* **`x`** → Traverse/search the directory and access known entries.

With **"x"** but without **"r"**, you may access a known file:
```bash
cat mydir/file.txt
```
But generally cannot list the directory:
```bash
ls mydir/
```

---

## Multiple Directories in a Path

For the path `/home/nahid/mydir/file.txt`, you generally need **"x"** permission on **every directory** in the path:

* **`/`** → `x`
* **`/home`** → `x`
* **`/nahid`** → `x`
* **`/mydir`** → `x`
* **`file.txt`** → Permission required by the operation (e.g., `r` for `cat`)

> 💡 **Key idea:** "x" on a directory means you can traverse/search it and pass through it to access something inside. It does not mean you can execute the files inside.



## Special Permissions
### SetUID (`s` on user execute bit)
Allows users to run a file with the file owner's permissions.
```bash
chmod u+s filename
```
Example: `/usr/bin/passwd` allows users to change their passwords.

# 💡 Understanding SetGID

**SetGID (Set Group ID)** behaves differently depending on whether it is applied to a **directory** or an **executable file**.

---

### 📌 SetGID on a Directory (Shared Workspaces)

When applied to a directory, SetGID ensures collaborative files automatically share the same group ownership.

#### The Problem: Without SetGID
Normally, new files inherit the creator's **primary group**. In a shared project, this creates permission issues:

* **Alice** (primary group: `alice`) creates `/project/alice.txt` → Ownership: `alice:alice`
* **Bob** (primary group: `bob`) creates `/project/bob.txt` → Ownership: `bob:bob`

Because the group owners differ, Alice and Bob may struggle to edit each other's files.

#### The Solution: With SetGID
Enabling SetGID forces all new files to inherit the **parent directory's group**, regardless of who creates them.

```bash
# 1. Create the shared directory
sudo mkdir /project
sudo chown root:developers /project

# 2. Set read/write/execute for owner & group, and enable SetGID (g+s)
sudo chmod 2770 /project
# Or using symbolic permissions: sudo chmod g+s /project
```

#### The Result
The directory permissions will display a lowercase `s` in the group execution slot:
`drwxrws---`

Inside the directory, file ownership automatically aligns:
```text
/project/
├── alice.txt  →  owner: alice | group: developers
└── bob.txt    →  owner: bob   | group: developers
```

> ⚠️ **Note:** SetGID changes **group** inheritance only. The user owner remains the specific person who created the file.

---

### 🔎 SetGID on an Executable File (Process Privileges)

## 1. Normal executable — no SetGID

Suppose Alice has:

```text
User: alice
Primary group: alice
Supplementary groups: users
```

She is **not** a member of `developers`.

We have this executable:

```text
-rwxr-xr-x root developers system_program
```

Alice runs:

```bash
./system_program
```

The resulting process normally has Alice's user identity and group identity:

```text
Effective user ID:  alice
Effective group ID: alice
```

So imagine the program tries to read:

```text
-rw-r----- root developers project.conf
```

The permissions mean:

```text
Owner (root)       → rw-
Group (developers) → r--
Others             → ---
```

Alice cannot normally read this file because:

* She is not `root`
* She is not a member of `developers`
* `others` have no permission

Therefore:

```text
Alice → system_program → project.conf
                         ✗ Permission denied
```

---

## 2. Now add SetGID

We enable SetGID:

```bash
chmod g+s system_program
```

Now:

```text
-rwxr-sr-x root developers system_program
```

Notice the `s`:

```text
-rwxr-sr-x
      ^
      SetGID
```

Alice runs exactly the same command:

```bash
./system_program
```

But now the process has:

```text
Effective user ID:  alice
Effective group ID: developers
```

So when the program tries to read:

```text
-rw-r----- root developers project.conf
```

Linux checks:

```text
Process effective group → developers
File group owner        → developers
Permission              → r
```

Therefore, the program can read the file:

```text
Alice → SetGID program → project.conf
                          ✓ Can read
```

---

## Does Alice herself become a member of `developers`?

**No.**

# Understanding SetUID Through an Example

SetUID works similarly to SetGID, but instead of using the executable file's **group identity**, the program uses the executable file owner's **effective user identity**.

## 1. Normal Executable

Suppose Alice is a regular user:

```text
User: alice
Groups: alice, users
```

There is a program:

```text
-rwxr-xr-x root developers system_program
```

The file is:

```text
Owner: root
Group: developers
```

Alice runs:

```bash
./system_program
```

Without SetUID, the resulting process normally runs with:

```text
Effective user:  alice
Effective group: alice
```

Suppose the program tries to access:

```text
-rw------- root root secret.txt
```

The permissions mean:

```text
Owner (root) → rw-
Group        → ---
Others       → ---
```

Alice cannot read or modify `secret.txt` because the file is accessible only to `root`.

```text
Alice → program → secret.txt
                   ✗ Permission denied
```

## 2. Enable SetUID

Now enable SetUID:

```bash
chmod u+s system_program
```

The permissions may look like:

```text
-rwsr-xr-x root developers system_program
   ^
 SetUID bit active
```

Alice runs:

```bash
./system_program
```

Now the resulting process can have:

```text
Effective user:  root
Effective group: alice
```

Because the executable is owned by `root`, the SetUID program runs with `root` as its effective user.

If the program accesses:

```text
-rw------- root root secret.txt
```

Linux checks:

```text
Process effective user → root
File owner             → root
Permission             → rw
```

Therefore, the program may be able to read or modify the file:

```text
Alice → SetUID program → secret.txt
                          ✓ Can access it
```

## Does Alice Become Root?

**No.**

Alice herself does not become root.

Before running the program:

```text
Alice's effective user → alice
```

While the SetUID program is running:

```text
Program's effective user → root
```

After the program exits:

```text
Alice's effective user → alice
```

Only the **process created by the program** temporarily has the executable owner's effective user identity.

## SetUID vs SetGID

```text
SetUID → Process uses the executable owner's effective user identity.

SetGID → Process uses the executable's group owner's effective group identity.
```

### Key Idea

> **SetUID allows an executable to run with the effective user identity of the file's owner, rather than the user who launched it.**

This is why SetUID programs must be written carefully: if an executable owned by `root` has SetUID enabled, a vulnerability in that program could potentially allow a regular user to perform operations with elevated privileges.




### Sticky Bit (`t` on others execute bit)
Used on directories to allow only the owner to delete/rename  their files. Sticky bit = users sharing a writable directory cannot normally delete or rename each other's files. The file owner and directory owner can, and root can.
```bash
chmod +t directory/
```
Example: `/tmp` directory.

## 🛡️ Default Permissions: `umask`

Whenever you create a brand new file or directory, Linux automatically assigns it a default set of read, write, and execute permissions. The tool that calculates and restricts these initial permissions is called the **User Mask (`umask`)**.

Think of `umask` as a **subtraction mask** or a permission filter. Instead of granting permissions, it *blocks* specific permissions from being given to new files and folders for security reasons.

---

### 1. Default Base Permissions (Before the Mask)
Before the `umask` filter is applied, the Linux kernel assigns maximum base permissions to new items:
*   📁 **New Directories:** Started with a baseline score of **`777`** (Full Read, Write, and Execute permissions for everyone).
*   📄 **New Files:** Started with a baseline score of **`666`** (Read and Write permissions for everyone. *Note: Files never get Execute permissions by default for security reasons*).

---

### 2. Checking Your Active Mask
To see your system's current permission filter, run:
```bash
umask
```
*Expected Output:* `0022` (or simply `022`). You can ignore the leading zero; the critical values are the last three digits (`022`), which correspond to **Owner**, **Group**, and **Others**.

---

### 3. How the Math Works (The Subtraction)
The `umask` value is subtracted directly from the baseline permissions to determine the final permissions.

If your `umask` is set to **`022`**, here is what happens when you make a new folder or file:

*   **For Directories (`777` baseline):**
    ```plaintext
      777  (Full Access Baseline)
    - 022  (Your umask Filter)
    -----
      755  (Final Permission: Owner can do everything; Group & Others can only read/enter)
    ```

*   **For Files (`666` baseline):**
    ```plaintext
      666  (Read/Write Baseline)
    - 022  (Your umask Filter)
    -----
      644  (Final Permission: Owner can read/write; Group & Others can only read)
    ```

---

### 4. Changing Your Default Mask
You can instantly alter the default safety filter for your current terminal session by passing a new three-digit mask:

```bash
# Sets a highly secure mask
umask 077
```

#### 🔍 What does `umask 077` do?
*   **Directories:** `777 - 077 = 700` (Only **you** can access the folder. Groups and strangers are completely blocked).
*   **Files:** `666 - 077 = 600` (Only **you** can read or modify the file. Everyone else sees permission denied).


## Conclusion
Understanding file permissions is essential for system security and proper file management. Using `chmod`, `chown`, and `chgrp`, you can control access to files and directories efficiently.
