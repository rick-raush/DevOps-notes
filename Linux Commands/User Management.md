🧑‍💻 Linux User & Permission Management Cheat Sheet

* * *

## 👤 **User Management**

### ✅ `id`

- Show **UID**, **GID**, and group membership of a user.  
    👉 `id alice`

* * *

### ✅ `useradd`

- Add a new user.  
    👉 `useradd [OPTIONS] USERNAME`

| Option | Description | Example |
| --- | --- | --- |
| `-m` | Create home directory | `useradd -m alice` |
| `-d` | Set custom home directory | `useradd -m -d /data/alice alice` |
| `-s` | Set login shell | `useradd -s /bin/zsh alice` |
| `-u` | Set custom UID | `useradd -u 1050 alice` |
| `-g` | Set primary group | `useradd -g devs alice` |
| `-G` | Add to extra groups | `useradd -G sudo,dev alice` |
| `-e` | Set account expiry date | `useradd -e 2025-12-31 alice` |
| `-r` | Create system user (no login) | `useradd -r serviceuser` |

* * *

### ✅ `userdel`

- Delete a user.  
    👉 `userdel [OPTIONS] USERNAME`

| Option | Description | Example |
| --- | --- | --- |
| *(none)* | Delete user only | `userdel alice` |
| `-r` | Delete user + home | `userdel -r alice` |
| `-f` | Force delete (even if logged in) | `userdel -f alice` |

&nbsp;

The content of your `wheel` file line by line:

* * *

### File content:

`## Allow users in the wheel group to execute sudo without password%wheel ALL=(ALL) NOPASSWD: ALL`

* * *

### Explanation:

1.  **`%wheel`**
    
    - The `%` sign means it’s a **group**, not a single user.
        
    - So this line applies to **all users in the `wheel` group**.
        
2.  **`ALL=(ALL)`**
    
    - The first `ALL` = **on all hosts** (relevant if you have multiple machines).
        
    - The `(ALL)` = **as all users** — meaning members can run commands as **any user**, including root.
        
3.  **`NOPASSWD: ALL`**
    
    - `NOPASSWD` = **don’t ask for a password** when running sudo.
        
    - `ALL` = applies to **all commands**.
        

* * *

### ✅ In short:

> Any user who is part of the **wheel group** can run **any command as root** using `sudo` **without being prompted for a password**.

&nbsp;

&nbsp;

* * *

### ✅ `passwd`

- Set or manage passwords.

| Command | Purpose |
| --- | --- |
| `passwd bob` | Change bob's password |
| `passwd -d bob` | Delete password (insecure) |
| `passwd -l bob` | Lock account |
| `passwd -u bob` | Unlock account |
| `passwd -e bob` | Force password change at next login |
| `passwd -n 2 bob` | Min days before change |
| `passwd -x 90 bob` | Max valid days |
| `passwd -w 7 bob` | Warn before expiry |

* * *

## 👥 **Group Management**

### ✅ `groupadd`

- Create a new group.

| Option | Description | Example |
| --- | --- | --- |
| *(none)* | Create normal group | `groupadd devs` |
| `-g` | Set GID | `groupadd -g 1050 devs` |
| `-r` | Create system group | `groupadd -r syslog` |

* * *

### ✅ `groupdel`

- Delete a group.  
    👉 `groupdel devs`

> ⚠️ Cannot delete if it's a user’s primary group.

* * *

### ✅ `gpasswd`

- Manage group membership.

| Command | Purpose |
| --- | --- |
| `gpasswd -a alice devs` | Add user to group |
| `gpasswd -d bob devs` | Remove user from group |
| `gpasswd -A admin devs` | Set group admin |
| `gpasswd -M user1,user2 devs` | Set full member list |

* * *

### ✅ `groups`

- Show groups of current or specific user.  
    👉 `groups alice`

* * *

## 🔐 **Privilege & Access Tools**

### ✅ `sudo`

- Run commands as **root** or another user.

| Option | Description | Example |
| --- | --- | --- |
| *(none)* | Run as root | `sudo apt update` |
| `-u` | Run as another user | `sudo -u postgres psql` |
| `-i` | Root login shell | `sudo -i` |
| `-s` | Root shell (non-login) | `sudo -s` |
| `-k` | Forget sudo password | `sudo -k` |
| `-v` | Extend sudo session | `sudo -v` |
| `-l` | List allowed commands | `sudo -l` |

* * *

### ✅ `su`

- Switch user account.  
    👉 `su [OPTIONS] [USERNAME]`

| Option | Description | Example |
| --- | --- | --- |
| *(none)* | Switch to root | `su` |
| `alice` | Switch to user alice | `su alice` |
| `-` or `-l` | Full login | `su - bob` |
| `-c` | Run one command | `su -c "ls /root"` |
| `-s` | Use shell | `su -s /bin/zsh bob` |

* * *

## 🧑‍💻 **Logged-in Users & History**

| Command | Purpose | Example |
| --- | --- | --- |
| `who` | Show logged-in users | `who` |
| `whoami` | Show current username | `whoami` |
| `users` | Show usernames only | `users` |
| `w` | Show users + what they're doing | `w` |
| `last` | Show login history | `last -n 5` |

* * *

## 📁 **File Permissions & Ownership**

* * *

### ✅ `chmod` — Change Permissions

```bash
chmod [OPTIONS] MODE FILE
```

| Mode | Meaning |
| --- | --- |
| `7` | rwx |
| `6` | rw- |
| `5` | r-x |
| `4` | r-- |

**Examples**:

```bash
chmod 755 file.sh         # Numeric mode
chmod u+x file.sh         # Add execute to user
chmod go-w file.sh        # Remove write for group and others
chmod -R 755 /var/www     # Recursive
```

* * *

### ✅ `chown` — Change File Owner

```bash
chown [USER][:GROUP] FILE
```

| Example | Description |
| --- | --- |
| `chown alice file.txt` | Change owner only |
| `chown alice:dev file.txt` | Change owner and group |
| `chown :dev file.txt` | Change group only |

* * *

### ✅ `chgrp` — Change Group Ownership

```bash
chgrp [OPTIONS] GROUP FILE
```

| Example | Description |
| --- | --- |
| `chgrp dev file.txt` | Change group to dev |
| `chgrp -R dev /var/log/` | Recursive group change |

* * *

### ✅ `chmod`, `chown`, and `chgrp` Relationship

| Command | Changes Owner? | Changes Group? |
| --- | --- | --- |
| `chmod` | ❌   | ❌   |
| `chown` | ✅   | ✅ (optional) |
| `chgrp` | ❌   | ✅   |

* * *

&nbsp;

* * *

## 🔧 `usermod` — Modify a User Account

### 📌 Purpose:

Used to change a user's properties like username, home directory, groups, shell, UID, etc.

* * *

### 🧰 Common Syntax:

```bash
usermod [OPTIONS] USERNAME
```

* * *

### ✅ Useful Options:

| Option | Description | Example |
| --- | --- | --- |
| `-l NEWNAME` | Change username | `usermod -l bob alice` → changes `alice` to `bob` |
| `-d /new/home` | Change home directory (does NOT move content) | `usermod -d /home/newdir alice` |
| `-m` | Move old home to new directory (use with `-d`) | `usermod -d /home/newdir -m alice` |
| `-u UID` | Change user ID | `usermod -u 1500 alice` |
| `-g GROUP` | Change primary group | `usermod -g developers alice` |
| `-G GROUPS` | Set additional (secondary) groups (replaces existing) | `usermod -G sudo,docker alice` |
| `-aG GROUPS` | **Append** user to additional groups (safer) | `usermod -aG docker alice` |
| `-s /bin/zsh` | Change login shell | `usermod -s /bin/zsh alice` |
| `-L` | Lock user account | `usermod -L alice` |
| `-U` | Unlock user account | `usermod -U alice` |
| `-e YYYY-MM-DD` | Set account expiry date | `usermod -e 2025-12-31 alice` |
| `-c "Comment"` | Set user description (like full name) | `usermod -c "Alice Cooper" alice` |

* * *

### 🛑 Important Notes:

- Must be run with **sudo** or as **root**
    
- When changing the username (`-l`), home directory and mail spool are not renamed unless handled manually
    
- `-G` **replaces** secondary groups unless combined with `-a`
    
- You can view user info with: `id USER` or `getent passwd USER`
    

* * *

### 🧪 Example Use Case:

```bash
# Add user 'alice' to the docker group
sudo usermod -aG docker alice

# Change alice's shell to zsh
sudo usermod -s /bin/zsh alice
```

* * *

&nbsp;