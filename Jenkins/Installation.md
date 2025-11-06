## 🔹 Jenkins Installation on Amazon Linux 2023 (EC2)

1.  **Update system packages**
    
    ```bash
    sudo dnf update -y
    ```
    
2.  **Install Java (required by Jenkins)**
    
    ```bash
    sudo dnf install java-17-amazon-corretto -y
    java -version
    ```
    
    (Jenkins supports Java 11 & 17; we used 17.)
    
3.  **Add Jenkins repo & import key**
    
    ```bash
    sudo curl -o /etc/yum.repos.d/jenkins.repo \
        https://pkg.jenkins.io/redhat-stable/jenkins.repo
    sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
    ```
    
4.  **Install Jenkins**
    
    ```bash
    sudo dnf install jenkins -y
    ```
    
5.  **Enable & start Jenkins service**
    
    ```bash
    sudo systemctl enable jenkins
    sudo systemctl start jenkins
    sudo systemctl status jenkins
    ```
    
6.  **Open port 8080 in AWS Security Group**
    
    - Inbound Rule → Custom TCP → Port `8080` → Source: `0.0.0.0/0` (or your IP).
7.  **Access Jenkins in browser**
    
    ```
    http://<EC2-Public-IP>:8080
    ```
    
8.  **Unlock Jenkins**
    
    ```bash
    sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    ```
    
    - Paste key into setup page.
        
    - Install **Suggested Plugins**.
        
    - Create first admin user.
        
    - Confirm Jenkins URL (use Public IP or Elastic IP).
        

* * *

&nbsp;

&nbsp;

&nbsp;

Awesome! Here’s a clear summary of what we did to switch `/tmp` from RAM-based tmpfs to disk-based storage:

* * *

## Summary: Switching `/tmp` from tmpfs (RAM) to disk-backed storage

### Problem:

- Your `/tmp` was mounted as a **tmpfs**, meaning it used RAM (and swap) and was limited in size (~453MB).
    
- You got errors because the free space in `/tmp` was below your 1GB threshold.
    
- You wanted `/tmp` to use **disk space** instead of RAM, so it can have more space and avoid memory pressure.
    

* * *

### What we did step-by-step:

1.  **Confirmed `/tmp` is tmpfs:**
    
    ```bash
    mount | grep /tmp
    # Showed tmpfs on /tmp
    ```
    
2.  **Disabled systemd tmpfs mounting on `/tmp` permanently:**
    
    ```bash
    sudo systemctl mask tmp.mount
    # Created symlink /etc/systemd/system/tmp.mount → /dev/null
    ```
    
    - This prevents systemd from mounting `/tmp` as tmpfs on every boot.
3.  **Unmounted the current tmpfs `/tmp`:**
    
    - You tried:
        
        ```bash
        sudo umount /tmp
        ```
        
        but got “target is busy”.
        
    - Then identified processes using `/tmp` (like Jenkins) with:
        
        ```bash
        sudo lsof +D /tmp
        ```
        
    - Stopped those services:
        
        ```bash
        sudo systemctl stop jenkins
        ```
        
    - Then successfully unmounted `/tmp`.
        
4.  **Ensured `/tmp` directory exists on disk and has correct permissions:**
    
    ```bash
    sudo mkdir -p /tmp
    sudo chmod 1777 /tmp
    ```
    
5.  **Rebooted the system:**
    
    - On reboot, systemd did NOT mount `/tmp` as tmpfs.
        
    - `/tmp` is now a normal directory on your root disk partition (`/dev/nvme0n1p1`), with plenty of disk space.
        
6.  **Verified `/tmp` is disk-backed:**
    
    ```bash
    mount | grep /tmp
    # No tmpfs entry for /tmp now
    df -h /tmp
    # Shows /tmp on disk partition with sufficient space
    ```
    

* * *

### Result:

- `/tmp` now uses disk space (your root partition), so you have plenty of space (>1GB).
    
- You freed RAM that was previously reserved for tmpfs.
    
- You solved the “disk space below threshold” error.
    

* * *

&nbsp;

Great question!

* * *

### Why use `chmod 1777` on `/tmp`?

- `/tmp` is a **shared temporary directory** where all users and many system processes create temporary files.
    
- **Permissions need to allow everyone to read, write, and execute (access) the directory.**
    

* * *

### Breaking down `1777`:

- The **`1`** at the beginning is the **sticky bit**.
    
- The **`777`** means:
    
    - Owner: read (4) + write (2) + execute (1) = 7
        
    - Group: read + write + execute = 7
        
    - Others: read + write + execute = 7
        

So, **everyone can read/write/execute** in `/tmp`.

* * *

### What does the sticky bit (`1` at the front) do?

- It **prevents users from deleting or renaming files that they don’t own** in `/tmp`.
    
- Without the sticky bit, any user could delete or rename anyone else’s temporary files, which is a security risk.
    
- With sticky bit set, users can only delete or rename files they own.
    

* * *

### In short:

- `chmod 1777 /tmp` means:
    
    - **Everyone can create and access files in `/tmp`.**
        
    - **But only file owners (or root) can delete or rename their own files.**
        

This is the **standard and secure permission** setting for `/tmp`.

* * *

&nbsp;

&nbsp;

Great follow-up! Here's how the sticky bit (the leading `1` in `1777`) **prevents users from deleting or renaming files they don’t own in `/tmp`**:

* * *

### How deletion and renaming work on Unix/Linux:

- Deleting or renaming a file is actually an operation on the **directory** containing the file, not on the file itself.
    
- To delete or rename a file, you need **write permission on the directory** that holds it.
    

* * *

### What happens without the sticky bit?

- `/tmp` is usually `rwxrwxrwx` (777), meaning **any user can write (modify) the directory**.
    
- Without the sticky bit, **any user can delete or rename any file in `/tmp`**, even if they don’t own it, because they have write permission on `/tmp` itself.
    

* * *

### What the sticky bit changes:

- When the sticky bit is set on a directory (`drwxrwxrwt`), the system **restricts who can delete or rename files inside** it:
    
    - Only the **file owner**, the **directory owner**, or **root** can delete or rename files in that directory.
- Other users, even if they have write access to the directory, **cannot delete or rename files owned by someone else**.
    

* * *

### In other words:

- Sticky bit **adds an extra check** on top of the directory write permission:
    
    - If you try to delete/rename a file **you don’t own** inside a sticky bit directory, the system denies the operation.
- This protects everyone's temporary files from being accidentally or maliciously deleted by others.
    

* * *

### You can see this with:

```bash
ls -ld /tmp
# drwxrwxrwt  15 root root 4096 Oct 16 12:00 /tmp
```

- The final `t` in the permission string means sticky bit is set.
    
- Without `t` it would be just `rwxrwxrwx`.
    

* * *

&nbsp;

&nbsp;

&nbsp;

&nbsp;