&nbsp;

# **Disk Management Summary**

### 1\. **du** — Disk Usage

- Shows how much disk space files/directories consume.
    
- Example: `du -h /path` (human-readable sizes)
    

### 2\. **df** — Disk Free

- Displays available and used disk space on mounted filesystems.
    
- Example: `df -h` (human-readable output)
    

* * *

### 3\. **mount** — Attach Filesystem

- Connects a device (USB, HDD, ISO, network share) to a directory (mount point) so you can access its files.

**Syntax:**

```bash
mount [OPTIONS] DEVICE MOUNT_POINT
```

**Common options:**

| Option | Purpose | Example |
| --- | --- | --- |
| `-t <fstype>` | Specify filesystem type (ext4, ntfs, etc.) | `mount -t ext4 /dev/sdb1 /mnt/data` |
| `-o ro` | Mount read-only | `mount -o ro /dev/sdb1 /mnt/usb` |
| `-o rw` | Mount read-write | `mount -o rw /dev/sdb1 /mnt/data` |
| `-o loop` | Mount ISO or disk image file | `mount -o loop file.iso /mnt/iso` |
| `-o noexec` | Prevent execution of binaries | `mount -o noexec /dev/sdb1 /mnt/usb` |
| `-o remount` | Remount with new options | `mount -o remount,rw /mnt/data` |

* * *

### 4\. **umount** — Detach Filesystem

- Unmounts a device from its mount point (disconnects it).

**Syntax:**

```bash
umount [OPTIONS] MOUNT_POINT_OR_DEVICE
```

**Common options:**

| Option | Purpose | Example |
| --- | --- | --- |
| (none) | Normal unmount | `umount /mnt/usb` |
| `-f` | Force unmount (risky) | `umount -f /mnt/usb` |
| `-l` | Lazy unmount (detach now, cleanup later) | `umount -l /mnt/usb` |

* * *

### 5\. **parted** — Partition Management CLI

- Tool to create, delete, resize, and manage disk partitions.
    
- Supports MBR and GPT partition tables.
    

**Common commands in parted:**

| Command | Purpose | Example |
| --- | --- | --- |
| `mklabel <type>` | Create partition table (msdos/gpt) | `mklabel gpt` |
| `mkpart <name> <fs-type> <start> <end>` | Create a partition | `mkpart primary ext4 1MiB 10GiB` |
| `print` | Show partition table | `print` |
| `rm <number>` | Remove a partition | `rm 2` |
| `resizepart <number> <end>` | Resize a partition | `resizepart 1 15GiB` |
| `set <number> <flag> on/off` | Set flags like boot | `set 1 boot on` |
| `quit` | Exit parted | `quit` |

* * *

### 6\. **gparted** — Graphical Partition Editor

- GUI frontend for parted.
    
- Easier for beginners.
    
- Allows visual partition creation, deletion, resizing, and moving.
    
- Not scriptable, and requires graphical environment.
    

* * *

# Quick Reference:

| Command | Use | Interface | Notes |
| --- | --- | --- | --- |
| `du` | Check disk usage | CLI | Size of files/folders |
| `df` | Check free/used disk space | CLI | Mounted filesystems info |
| `mount` | Attach a filesystem | CLI | Needs mount point |
| `umount` | Detach a filesystem | CLI | Reverse of mount |
| `parted` | Manage partitions | CLI | Scriptable, powerful |
| `gparted` | Visual partition management | GUI | Easy, beginner-friendly |

* * *

&nbsp;

&nbsp;