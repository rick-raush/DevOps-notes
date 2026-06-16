&nbsp;

## 📁 **File Management - Linux Commands**

### 🧭 Navigation

| Command | Description |
| --- | --- |
| `pwd` | Show current working directory |
| `cd <dir>` | Change to directory `<dir>` |
| `cd ..` | Go up one level (parent directory) |

* * *

### 📄 File and Folder Creation

| Command | Description |
| --- | --- |
| `touch <file>` | Create an empty file |
| `mkdir <dir>` | Create a directory |
| `mkdir -p <path>` | Create nested directories (parents if needed) |
| `mkdir -v` | Show messages for created directories |
| `mkdir -m 755 <dir>` | Create directory with specific permissions |

* * *

### ❌ Deletion

| Command | Description |
| --- | --- |
| `rm <file>` | Delete a file |
| `rm -r <dir>` | Delete a directory and its contents (recursive) |
| `rm -f <file>` | Force delete (no confirmation) |
| `rm -i` | Prompt before deleting |
| `rm -v` | Show files as they are removed |
| `rmdir <dir>` | Remove empty directory |
| `rmdir -p` | Remove nested empty directories |

&nbsp;

* * *

## **1️⃣ Command: `find . \( -name ".terraform*" -o -name ".terragrunt*" \) -exec rm -rv {} +`**

### 🔹 Breakdown:

- `find .`
    
    - Starts searching in the current directory `.` and all subdirectories recursively.
- `\( -name ".terraform*" -o -name ".terragrunt*" \)`
    
    - `-name ".terraform*"` → matches any file or directory starting with `.terraform`
        
    - `-o` → logical OR
        
    - `-name ".terragrunt*"` → matches any file or directory starting with `.terragrunt`
        
    - The `\(` and `\)` are required to group the OR condition in `find`.
        
- `-exec rm -rv {} +`
    
    - For each match, execute `rm -rv` on it.
        
    - `{}` → placeholder for the matched file/directory
        
    - `+` → runs `rm` on multiple files at once (more efficient than `\;`)
        
    - `rm -rv` flags explained below
        

### 🔹 Flags for `rm -rv`:

| Flag | Meaning |
| --- | --- |
| `-r` | recursive — deletes directories and their contents |
| `-v` | verbose — prints each file/folder as it is removed |

* * *

### 🔹 Key point:

- `find` **traverses all subdirectories** automatically.
    
- It will match **both files and directories** anywhere in the directory tree.
    
- Very safe when you want recursive deletion based on patterns.
    

* * *

## **2️⃣ Command: `rm -rvf ".terraform*" ".terragrunt*"`**

### 🔹 Breakdown:

- `rm` → remove command
    
- Flags:
    
    - `-r` → recursive
        
    - `-v` → verbose
        
    - `-f` → force (ignore nonexistent files, don’t prompt for confirmation)
        
- `".terraform*" ".terragrunt*"` → shell globbing patterns, **expanded by the shell before `rm` runs**.
    

* * *

### 🔹 Behavior:

- Only matches files/directories in **the current directory**.
    
- Does **not traverse subdirectories**.
    
- If no matches are found, zsh (without `nullglob`) will throw an error.
    
- Quoting the pattern prevents the shell from interpreting the glob, but then `rm` itself does not expand globs — it will fail unless you enable glob expansion in `rm` (which it doesn’t by default).
    

* * *

## **3️⃣ Comparison**

| Feature | `rm -rvf ".terraform*" ".terragrunt*"` | `find . -name ... -exec rm -rv {} +` |
| --- | --- | --- |
| Recursive | Only deletes directories recursively if they are in current directory, not inside subdirs | True recursion through all subdirectories |
| Pattern match | Shell globbing, limited to current directory | Uses `find` with `-name` or regex; can search anywhere |
| Verbose | Yes (`-v`) | Yes (`rm -v`) |
| Force deletion | Yes (`-f`) | `rm` can be combined with `-f` if needed |
| Safety | Can fail silently or delete unintended files if glob misused | Safer — only deletes files that actually match pattern |
| Handles dotfiles in subdirs | No  | Yes |
| Works across all shells | Glob expansion can behave differently | Works reliably across bash, zsh, etc. |

* * *

### **✅ TL;DR**

- Use `rm -rvf` for **quick deletion in the current directory**.
    
- Use `find ... -exec rm -rv {} +` for **recursively deleting patterns in all subdirectories** — much safer and reliable.
    

&nbsp;

* * *

### 📋 Copy & Move

| Command | Description |
| --- | --- |
| `cp <src> <dest>` | Copy file |
| `cp -r <src_dir> <dest>` | Copy directory recursively |
| `cp -i` | Prompt before overwrite |
| `cp -u` | Copy only if source is newer |
| `cp -a` | Archive (preserves structure, links, permissions) |
| `mv <src> <dest>` | Move or rename file/directory |
| `mv -i` | Prompt before overwrite |
| `mv -f` | Force move (overwrite without prompt) |
| `mv -v` | Verbose (show moved files) |

* * *

### 🔎 Searching

| Command | Description |
| --- | --- |
| `find <path> -name "<pattern>"` | Find files by name |
| `find . -type f -size +10M` | Find files larger than 10MB |
| `find <path> -exec <cmd> {} \;` | Run command on found files |

* * *

### 🌳 Structure & Links

| Command | Description |
| --- | --- |
| `tree` | Show directory structure (needs install) |
| `tree -L 2` | Limit depth to 2 levels |
| `tree -a` | Include hidden files |
| `tree -d` | Show only directories |
| `tree -I "*.log"` | Ignore `.log` files |

* * *

### 🔗 Links

| Command | Description |
| --- | --- |
| `ln <file> <link>` | Create a hard link |
| `ln -s <target> <link>` | Create a symbolic (soft) link |
| `ln -f` | Force overwrite existing links |
| `ln -t <dir> <target>` | Link to target into given directory |

&nbsp;

* * *

## 🔗 **Links in Linux**

Linux filesystems allow **links**, which are basically shortcuts or references to files or directories. There are two main types:

### 1\. **Hard Links**

- A **hard link** is like an additional name for the same file.
    
- Both the original file and hard link point to the same data (inode) on disk.
    
- Deleting one hard link **does not delete the actual file data** as long as one link remains.
    
- Hard links **cannot span across different filesystems** (partitions).
    
- You **cannot create hard links to directories** (usually restricted to avoid loops).
    

**Example:**

```bash
ln file.txt file_hardlink.txt
```

- `file_hardlink.txt` is now another name for `file.txt`.
    
- Changes to either affect the same file content.
    

* * *

### 2\. **Symbolic (Soft) Links**

- A **symbolic link (symlink)** is like a shortcut or pointer to another file or directory.
    
- It stores the **path** to the target file or directory.
    
- If the target is deleted or moved, the symlink becomes broken (dangling link).
    
- Symlinks **can cross filesystem boundaries**.
    
- Symlinks **can point to directories**.
    

**Example:**

```bash
ln -s /path/to/original/file.txt shortcut.txt
```

- `shortcut.txt` points to `/path/to/original/file.txt`.
    
- Opening `shortcut.txt` opens the target file.
    

* * *

### Differences at a glance

| Feature | Hard Link | Symbolic Link |
| --- | --- | --- |
| Points to | Same inode (file data) | Path to file/directory |
| Can link directories? | No (usually restricted) | Yes |
| Can cross filesystems? | No  | Yes |
| Breaks if target deleted? | No (file data remains if at least one hard link exists) | Yes (dangling link) |
| Uses disk space? | No additional (just directory entry) | Small space for storing path |

* * *

### How to check links?

Use `ls -l` to see links:

```bash
ls -l file.txt file_hardlink.txt shortcut.txt
```

- Hard links have the same inode number (first column).
    
- Symbolic links show `->` followed by the target.
    

* * *

&nbsp;

&nbsp;

| **Command** | **Purpose** | **Output** | **Key Flags Explained** |
| --- | --- | --- | --- |
| **`du -csh`** | **Calculate Directory Size** | Provides the **cumulative size** of a directory (or multiple files/directories) in human-readable format. | **`-c`**: Provides a **grand total** line (cumulative).  <br><br/><br/><br/>**`-s`**: **Summary** (shows only the total, not subdirectories).  <br><br/><br/><br/>**`-h`**: **Human-readable** (K, M, G). |
| **`ls -lh`** | **List File Details & Size** | Lists the contents of a directory, showing the **size of individual files** in a human-readable format. | **`-l`**: **Long listing** (shows permissions, owner, date, and size).  <br><br/><br/><br/>**`-h`**: **Human-readable** (K, M, G). |
| **`ls -i`** | **List Inode Numbers** | Lists the contents of a directory, showing the unique **inode number** associated with each file or directory. | **`-i`**: Displays the **inode number** in the first column. |
| **`df -ih`** | **Check Disk Partition Usage** | Displays the amount of **free and used disk space** and **inode usage** across all mounted file systems (partitions). | **`-i`**: Reports **inode usage** (instead of block usage).  <br><br/><br/><br/>**`-h`**: **Human-readable** (K, M, G). |

&nbsp;

&nbsp;

### ✅ Filesystem and Mounted On

- **Filesystem** = *where storage comes from*
    
    - Disk → `/dev/nvme0n1p1`
        
    - Memory → `tmpfs`, `devtmpfs`
        
    - Virtual → `proc`, `sysfs`
        
- **Mounted on** = *directory path where that storage is attached*
    

* * *

### 🔹 Key idea

Linux has **one directory tree (`/`)**.  
Different filesystems are **mounted into directories** inside it.

* * *

### 🔹 Examples

- `tmpfs` → memory → mounted on `/run`
    
- `devtmpfs` → memory → mounted on `/dev`
    
- `/dev/nvme0n1p1` → disk → mounted on `/`
    

* * *

### 🔑 One-line rule

> **Filesystem = storage source | Mounted on = directory location**

That’s it.

Here is a concise summary of our deep dive into Linux storage, moving from the physical hardware up to the usable folder.

* * *

## 🏗️ The Linux Storage Stack: A Summary

The process of making storage usable follows a strict 3-step hierarchy: **Partitioning → Formatting → Mounting.**

### 1\. The Device Portal (`/dev`)

In Linux, hardware is represented as files in the `/dev` directory.

- **Disk:** The physical device (e.g., `/dev/nvme1n1`).
    
- **Partition:** A logical slice of that disk (e.g., `/dev/nvme1n1p1`).
    

### 2\. Building the Structure (`fdisk`)

Before you can store data, you must define the boundaries of the disk using `fdisk`.

- **Role:** Creates the partition table (MBR or GPT).
    
- **Analogy:** Building the "walls" of a room within a house.
    
- **Action:** It creates the device file name like `p1` (partition 1).
    

### 3\. Adding the Logic (`mkfs`)

A partition is just empty space until you "format" it with a filesystem.

- **Role:** To "add" a filesystem (like **ext4** or **XFS**) to a partition.
    
- **Analogy:** Putting "shelving" and an "indexing system" inside the room so you can find your items.
    
- **Result:** The partition is now ready to hold files and folders.
    

### 4\. Making it Accessible (`mount`)

Finally, you must attach that formatted partition to your existing file tree.

- **Role:** Maps the **filesystem** on a **partition** to a **mount point** (a directory like `/data`).
    
- **Analogy:** Opening the "door" to the room so you can walk in and use it.
    
- **Command:** `mount /dev/nvme1n1p1 /data`
    

* * *

### 📋 Technical Quick-Reference

| **Layer** | **Component** | **Tool** | **Purpose** |
| --- | --- | --- | --- |
| **Physical** | Disk (`nvme1n1`) | —   | The actual hardware. |
| **Structural** | Partition (`p1`) | `fdisk` | Divides the disk into usable sections. |
| **Logical** | Filesystem (`ext4`) | `mkfs` | Organizes how data is stored/retrieved. |
| **Access** | Mount Point (`/data`) | `mount` | Links the hardware to a usable folder. |

**The Big Picture:** You don't just "mount a disk." You **mount a filesystem** that lives on a **partition** which is part of a **disk**.

&nbsp;

* * *

# 🧾 EBS Resize Flow – Summary

## 🧱 Disk Structure

```text
Disk        → /dev/nvme0n1
Partition   → /dev/nvme0n1p1
Filesystem  → XFS / EXT4 (inside partition)
Mount       → /
```

* * *

# 🔄 What happens when you increase EBS size

👉 Only **disk size increases**  
Partition + filesystem remain unchanged

* * *

# ⚙️ Required steps to use new space

## 1\. Expand partition

```bash
growpart /dev/<disk> <partition_number>
```

✔ Expands partition to full disk  
❌ Does NOT touch filesystem

* * *

## 2\. Expand filesystem

### For XFS:

```bash
xfs_growfs /
```

### For EXT4:

```bash
resize2fs /dev/<partition>
```

✔ Makes filesystem use new space

* * *

# 📍 Mounting – Do you need it?

👉 **No manual mount needed**

- Root (`/`) is already mounted by OS (via `/etc/fstab`)
    
- You are resizing a **live mounted filesystem**
    
- `xfs_growfs /` uses the existing mount
    

* * *

# 🔍 Key Commands Mapping

| Purpose | Command |
| --- | --- |
| Find root partition | `findmnt -n -o SOURCE /` |
| Find disk | `lsblk -no PKNAME <partition>` |
| Check FS type | `findmnt -n -o FSTYPE /` |
| Verify size | `df -h /` |

* * *

# ⚠️ Common Mistakes

❌ Assuming disk name (`nvme0n1`)  
❌ Assuming partition number (`1`)  
❌ Assuming filesystem is XFS  
❌ Using `lsblk | head` (unreliable)

* * *

# ✅ Production Best Practices

✔ Detect everything dynamically  
✔ Handle both XFS & EXT4  
✔ Add retries (AWS delay)  
✔ Use `set -euo pipefail`  
✔ Verify after resize (`df -h`)

* * *

# 🚀 One-line takeaway

👉  
**EBS resize = grow partition (`growpart`) + grow filesystem (`xfs_growfs` / `resize2fs`), no remount needed.**

* * *

&nbsp;

&nbsp;

* * *

# 🧾 What is `/etc/fstab`?

👉 It tells Linux:

> “Which filesystems to mount, where to mount them, and how to mount them — automatically at boot.”

* * *

# 📄 Your file

```bash
UUID=552af251-8f34-4207-8c34-97c8f38c6833     /           xfs    defaults,noatime  1   1
UUID=9026-B5B4        /boot/efi       vfat    defaults,noatime,uid=0,gid=0,umask=0077,shortname=winnt,x-systemd.automount 0 2
```

* * *

# 🧱 General format

```text
<device>   <mount_point>   <filesystem>   <options>   <dump>   <fsck_order>
```

* * *

# 🔍 Line 1 (ROOT filesystem)

```bash
UUID=552af251-8f34-4207-8c34-97c8f38c6833   /   xfs   defaults,noatime   1   1
```

### 🔹 Breakdown:

### 1\. `UUID=552af251-...`

- Unique identifier of the partition
    
- Used instead of `/dev/nvme0n1p1` (safer — device names can change)
    

* * *

### 2\. `/`

- Mount point
    
- This is your **root filesystem**
    

* * *

### 3\. `xfs`

- Filesystem type

* * *

### 4\. `defaults,noatime`

- `defaults` → standard options (rw, suid, dev, exec, auto, etc.)
    
- `noatime` → disables updating file access time
    

👉 Why `noatime`?

- Improves performance
    
- Reduces disk writes
    

* * *

### 5\. `1` (dump)

- Used by old backup tool `dump`
    
- `1` = include in backup (rarely used nowadays)
    

* * *

### 6\. `1` (fsck order)

- Filesystem check order during boot

👉 `1` = check first (root FS always first)

* * *

# 🔍 Line 2 (EFI boot partition)

```bash
UUID=9026-B5B4   /boot/efi   vfat   defaults,noatime,uid=0,gid=0,umask=0077,shortname=winnt,x-systemd.automount   0   2
```

### 🔹 Breakdown:

### 1\. `UUID=9026-B5B4`

- EFI partition identifier

* * *

### 2\. `/boot/efi`

- Mount point for bootloader files

* * *

### 3\. `vfat`

- Filesystem type (FAT — required for UEFI)

* * *

### 4\. Options explained:

| Option | Meaning |
| --- | --- |
| `defaults` | standard mount options |
| `noatime` | no access time updates |
| `uid=0,gid=0` | owned by root |
| `umask=0077` | only root can access |
| `shortname=winnt` | filename compatibility |
| `x-systemd.automount` | mount on demand (lazy mount) |

* * *

### 5\. `0` (dump)

- Not included in dump backup

* * *

### 6\. `2` (fsck order)

- Checked after root filesystem

* * *

# 🧠 Key Insight

👉 This file is why:

- Your system **automatically mounts `/`**
    
- You **don’t need to mount manually**
    
- `xfs_growfs /` works directly
    

* * *

# 🔍 How this connects to your earlier question

Because of this line:

```bash
UUID=552af251-...   /   xfs ...
```

👉 At boot:

```text
Partition → mounted to /
```

So your resize script works **without any mount command**

* * *

# ⚠️ Important Notes

### ✔️ Always use UUID (not device names)

Devices can change:

```text
/dev/nvme0n1 → /dev/nvme1n1 (after reboot)
```

UUID stays constant ✅

* * *

### ✔️ Wrong fstab entry = system won’t boot

Always test with:

```bash
mount -a
```

* * *

# 🚀 One-line summary

👉  
**`/etc/fstab` defines how disks (via UUID) are automatically mounted (like `/`) at boot, which is why you don’t need to mount manually.**

* * *

If you want, I can show:  
✅ how to map UUID → actual device  
✅ how to safely edit fstab in production (very important skill)