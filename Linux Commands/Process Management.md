🧠 **Linux Command Summary: Process, Service, System & Disk Management**

* * *

## 🔹 `ps` – Process Status

| Option | Description | Example |
| --- | --- | --- |
| `ps` | Show user’s current shell processes | `ps` |
| `-e` or `-A` | Show all processes | `ps -e` |
| `-f` | Full format listing | `ps -ef` |
| `-u <user>` | Processes by user | `ps -u root` |
| `-p <pid>` | Process with specific PID | `ps -p 1234` |
| `-x` | Show background (no terminal) processes | `ps -x` |
| `-H` | Show process hierarchy | `ps -eH` |
| `-o` | Custom output columns | `ps -eo pid,cmd,%cpu` |
| `--sort` | Sort output | `ps -eo pid,cmd --sort=-%cpu` |

🔁 **Related Tools**

- `top`, `htop` → Realtime monitoring
    
- `pgrep` → Find processes by name
    
- `kill` → Terminate by PID
    

* * *

## 🔹 `top` – Realtime System Monitor

| Option | Description | Example |
| --- | --- | --- |
| `-n <num>` | Number of refreshes | `top -n 3` |
| `-d <sec>` | Delay between refreshes | `top -d 5` |
| `-u <user>` | Show only user's processes | `top -u alice` |
| `-p <pid>` | Monitor specific PID | `top -p 1234` |
| `-b` | Batch mode (non-interactive, loggable) | `top -b -n 1 > log.txt` |
| `-i` | Ignore idle processes | `top -i` |

* * *

## 🔹 `kill` – Send Signals to Processes

| Signal | Number | Meaning |
| --- | --- | --- |
| `SIGTERM` | 15  | Graceful termination (default) |
| `SIGKILL` | 9   | Force kill (no cleanup) |
| `SIGSTOP` | 19  | Pause a process |
| `SIGCONT` | 18  | Resume a paused process |
| `SIGHUP` | 1   | Reload/restart daemon config |

📌 **Usage:**

```bash
kill -9 1234     # Force kill
kill -15 1234    # Graceful kill
```

* * *

## 🔹 `systemctl` – Systemd Service Management

## ⚙️ **Creating and Managing a Custom systemd Service**

### ✅ **1\. Create Service File**

```ini
# /etc/systemd/system/myservice.service
[Unit]
Description=My Custom Service
After=network.target

[Service]
ExecStart=/usr/bin/python3 /home/user/myscript.py
Restart=always
User=user
Group=user
WorkingDirectory=/home/user
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

* * *

### 🔄 **2\. Reload systemd to recognize new service**

```bash
sudo systemctl daemon-reload
```

* * *

### ▶️ **3\. Enable and Start the Service**

```bash
sudo systemctl enable myservice      # Start on boot
sudo systemctl start myservice       # Start now
```

* * *

### 📌 **4\. Check Status**

```bash
sudo systemctl status myservice
```

* * *

### ⛔ **5\. Stop and Disable**

```bash
sudo systemctl stop myservice
sudo systemctl disable myservice
```

* * *

| Command | Description |
| --- | --- |
| `start <service>` | Start service |
| `stop <service>` | Stop service |
| `restart <service>` | Restart service |
| `reload <service>` | Reload config |
| `status <service>` | Show status |
| `enable <service>` | Start on boot |
| `disable <service>` | Don't start on boot |
| `is-active <service>` | Is service running? |
| `is-enabled <service>` | Is service enabled? |
| `mask <service>` | Prevent any starts (even manual) |
| `unmask <service>` | Re-enable service |
| `daemon-reload` | Reload after editing unit files |

🧱 **Systemd Units**

- `.service`, `.socket`, `.target`, `.mount`, `.timer`, etc.

* * *

## 🔹 `df` – Disk Space (Filesystem Level)

| Option | Description | Example |
| --- | --- | --- |
| `-h` | Human-readable (KB, MB, GB) | `df -h` |
| `-T` | Show filesystem type | `df -T` |
| `-i` | Show inode usage | `df -i` |
| `--total` | Show total at end | `df -h --total` |

* * *

## 🔹 `du` – Disk Usage (File/Directory Level)

| Option | Description | Example |
| --- | --- | --- |
| `-h` | Human-readable | `du -h` |
| `-s` | Summary for a folder | `du -sh /var/log` |
| `-a` | Show all files | `du -ah` |
| `--max-depth=N` | Limit folder depth | `du -h --max-depth=1` |
| `-c` | Grand total | `du -ch` |
| `--exclude=*.log` | Exclude files | `du -h --exclude=*.log` |

* * *

## 🔹 `free` – Memory Usage

| Option | Description | Example |
| --- | --- | --- |
| `-h` | Human-readable | `free -h` |
| `-m` | Show in MB | `free -m` |
| `-g` | Show in GB | `free -g` |
| `-s N` | Refresh every N seconds | `free -h -s 2` |
| `-t` | Show total line (RAM + Swap) | `free -ht` |

* * *

## 🔹 `uptime` – System Uptime & Load

| Option | Description | Example |
| --- | --- | --- |
| `-p` | Pretty format | `uptime -p` |
| `-s` | Show system boot time | `uptime -s` |

### 🔁 Load Average

Example: `load average: 0.50, 0.40, 0.35`

- 1 min, 5 min, 15 min average of processes **waiting for CPU**
    
- On a **4-core** system:
    
    - `4.0` = full load
        
    - `>4.0` = overloaded
        

* * *

## 🔹 `nice` – Set Process Priority

| Option | Description | Example |
| --- | --- | --- |
| `-n <value>` | Set niceness (-20 to 19) | `nice -n 5 backup.sh` |
| *No option* | Default niceness (usually 10) | `nice script.sh` |

- Lower value = higher priority
    
- Use **positive** values for background jobs
    

* * *

## 📌 Summary Table

| Command | Focus | Use It For |
| --- | --- | --- |
| `ps` | Snapshot of current processes | Find, filter, and list running processes |
| `top` | Live process monitor | Watch CPU/memory usage in real-time |
| `kill` | Send signal to a process | Stop or restart a process |
| `systemctl` | Manage system services | Start, stop, enable, monitor services |
| `df` | Disk space (mounted FS) | Check free space on partitions |
| `du` | Directory/file size usage | Find what’s using disk space |
| `free` | RAM and swap usage | View memory consumption |
| `uptime` | How long system is running | Check system load and uptime |
| `nice` | Set job priority | Run less-important jobs with lower CPU |

* * *

&nbsp;

* * *

## 🖥️ **`uname` – System Information**

| Command | Description | Example Output |
| --- | --- | --- |
| `uname` | Kernel name | `Linux` |
| `uname -r` | Kernel version | `5.15.0-84-generic` |
| `uname -n` | Hostname | `myhostname` |
| `uname -a` | All info: kernel, machine, OS, etc. | Full kernel/system info |
| `uname -m` | Machine hardware name (architecture) | `x86_64` |