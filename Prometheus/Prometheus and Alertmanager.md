&nbsp;

# 🚀 Prometheus 

![c7a76825f7e50598e3e3667077108385.png](../_resources/c7a76825f7e50598e3e3667077108385.png)

This guide covers installing **Prometheus**, running it with the default config, installing **Node Exporter** on an app server, and creating **recording rules**.

* * *

## 1\. Install Prometheus

### Step 1: Download Prometheus

Go to the [Prometheus official downloads page](https://prometheus.io/download/) and copy the latest `wget` link for Linux. Example:

```bash
wget https://github.com/prometheus/prometheus/releases/download/v3.0.0/prometheus-3.0.0.linux-amd64.tar.gz
```

### Step 2: Extract the archive

```bash
tar -xvf prometheus-3.0.0.linux-amd64.tar.gz
cd prometheus-3.0.0.linux-amd64
```

### Step 3: Contents of the extracted folder

You’ll see something like:

```
-rwxr-xr-x. 1 ec2-user ec2-user 159542981 Aug 21 13:34 prometheus
-rwxr-xr-x. 1 ec2-user ec2-user 150853025 Aug 21 13:34 promtool
-rw-r--r--. 1 ec2-user ec2-user 1093 Aug 23 08:25 prometheus.yml
-rw-r--r--. 1 ec2-user ec2-user 3773 Aug 23 08:25 NOTICE
-rw-r--r--. 1 ec2-user ec2-user 11357 Aug 23 08:25 LICENSE
```

- **prometheus** → main Prometheus binary
    
- **promtool** → config validation tool
    
- **prometheus.yml** → default config file
    
- **NOTICE / LICENSE** → docs
    

### Step 4: Run Prometheus with default config

```bash
./prometheus --config.file=prometheus.yml
```

✅ Access the Prometheus UI at:  
`http://localhost:9090`

* * *

## 2\. Install Node Exporter (on app server)

Node Exporter exposes system metrics (CPU, memory, disk, etc.).

### Step 1: Download Node Exporter

From [Node Exporter official downloads](https://prometheus.io/download/):

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
```

### Step 2: Extract and run

```bash
tar -xvf node_exporter-1.8.1.linux-amd64.tar.gz
cd node_exporter-1.8.1.linux-amd64
```

Run Node Exporter on your private IP and port `9100`:

```bash
./node_exporter --web.listen-address="PRIVATE_IP:9100"
```

👉 Get private IP:

```bash
hostname -i
```

### Step 3: Add Node Exporter to Prometheus config

Edit `prometheus.yml` on the Prometheus server:

```yaml
scrape_configs:
  - job_name: "node"
    static_configs:
      - targets: ["PRIVATE_IP:9100"]
```

Restart Prometheus, and now you’ll see Node metrics in the Prometheus UI.

* * *

## 3\. Recording Rules in Prometheus

Recording rules **precompute queries** and save them as new metrics, improving performance for dashboards and alerts.

### Example: CPU Usage Recording Rule

```yaml
groups:
- name: cpu-node
  rules:
  - record: job_instance_mode:node_cpu_seconds:avg_rate5m
    expr: avg by (job, instance, mode) (
            rate(node_cpu_seconds_total[5m])
          )
```

🔎 **Explanation (line by line):**

- **record:**  
    `job_instance_mode:node_cpu_seconds:avg_rate5m` → new metric name that will be stored.
    
- **expr:**  
    `rate(node_cpu_seconds_total[5m])` → calculates per-second CPU usage rate from the counter.
    
- **avg by (job, instance, mode)** → averages across all CPU cores, grouped by job, instance, and CPU mode.
    

👉 Why useful?  
Instead of calculating CPU usage every time in a dashboard or alert, Prometheus precomputes it. This saves resources and makes queries faster.

* * *

# ✅ Summary

1.  Download and extract **Prometheus** → `prometheus`, `promtool`, `prometheus.yml`.
    
2.  Run Prometheus with → `./prometheus --config.file=prometheus.yml`.
    
3.  Download and run **Node Exporter** on app server → `./node_exporter --web.listen-address="PRIVATE_IP:9100"`.
    
4.  Add Node Exporter target in Prometheus config.
    
5.  Use **recording rules** to precompute metrics (like CPU usage).
    

* * *

# 🧠 Full Breakdown of the Recording Rule

```yaml
groups:
- name: cpu-node
  rules:
  - record: job_instance_mode:node_cpu_seconds:avg_rate5m
    expr: avg by (job, instance, mode) (
            rate(node_cpu_seconds_total[5m])
          )
```

* * *

## 🔹 `groups:` and `name: cpu-node`

- **groups**:
    
    - A collection of related recording rules.
        
    - You can have multiple groups inside one rules file.
        
- **name: cpu-node**:
    
    - A human-readable label for this group.
        
    - Helps when debugging or organizing rules (e.g., separate groups for CPU, memory, network).
        

* * *

## 🔹 `record: job_instance_mode:node_cpu_seconds:avg_rate5m`

- **record** = the name of the new metric Prometheus will generate.
    
- This custom metric name is descriptive and structured:
    
    - **job** → which scrape job (e.g., `node_exporter`)
        
    - **instance** → which host/IP
        
    - **mode** → CPU mode (`user`, `system`, `idle`, etc.)
        
    - **avg_rate5m** → tells you it’s the average rate over 5 minutes
        

✅ This is now queryable just like a built-in metric.

* * *

## 🔹 `expr: avg by (job, instance, mode) (rate(node_cpu_seconds_total[5m]))`

The PromQL expression that generates values for the new metric.

### Step 1: `node_cpu_seconds_total`

- Built-in Node Exporter metric.
    
- It’s a **cumulative counter**: total seconds CPU spent in each mode.
    
- Example modes:
    
    - `user` → time spent running user processes
        
    - `system` → time spent on kernel/system processes
        
    - `idle` → CPU idle time
        
    - `iowait` → waiting on I/O
        

* * *

### Step 2: `rate(...[5m])`

- Converts the cumulative counter into a **per-second rate**.
    
- Looks at the last **5 minutes** of data.
    
- Tells us *how fast the counter is increasing per second*.
    
- Example: “CPU spent 0.25s per second in user mode over the last 5 minutes” (≈25% usage if across 1 core).
    

* * *

### Step 3: `avg by (job, instance, mode)`

- Takes the average across all CPU cores on the machine.
    
- Groups results by:
    
    - **job** → scrape job name (e.g., `node`)
        
    - **instance** → server hostname/IP
        
    - **mode** → CPU mode
        

✅ Result: Average CPU usage per mode per instance over the last 5 minutes.

* * *

# 🧪 Real-World Use Case

- Dashboards: Plot how much CPU time is spent in `user`, `system`, `idle`, etc.
    
- Alerts: Detect high `system` CPU or low `idle` CPU.
    
- Performance: Precomputing avoids running a heavy query repeatedly in Grafana/alerts.
    

* * *

# 📊 Extending to CPU Utilization %

The above rule gives raw rates. To get actual utilization %, we normalize against **all modes**.

```yaml
- record: job_instance:cpu_usage:percent
  expr: 100 * (1 - avg by (job, instance) (
                  rate(node_cpu_seconds_total{mode="idle"}[5m])
                ))
```

### 🔍 What this does:

- Takes `idle` CPU time per instance.
    
- `1 - idle` = busy CPU time (user + system + other).
    
- Multiply by `100` = percentage CPU usage.
    

✅ This gives you **CPU utilization % per instance**.

* * *

# 🚨 Example Alert Rule (High CPU Usage)

```yaml
groups:
- name: cpu-alerts
  rules:
  - alert: HighCPUUsage
    expr: job_instance:cpu_usage:percent > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High CPU usage on {{ $labels.instance }}"
      description: "CPU usage is above 80% for more than 5 minutes."
```

- Triggers if CPU usage > 80% for 5 minutes.
    
- Can be hooked into Alertmanager for notifications (Slack, email, PagerDuty, etc.).
    

* * *

✅ So now you’ve got:

1.  **Recording Rule** (raw CPU per mode)
    
2.  **Derived Metric** (CPU utilization %)
    
3.  **Alert Rule** (fires if too high)
    

* * *

&nbsp;

# 📂 Structure of `prometheus.yml`

Prometheus config usually has 5 big sections:

1.  **global** → defaults (scrape/eval intervals)
    
2.  **alerting** → where Alertmanager lives
    
3.  **rule_files** → alerting/recording rules to load
    
4.  **scrape_configs** → what targets to scrape
    
5.  (optional) **remote_write/remote_read** → external storage
    

* * *

## 1️⃣ Global Config

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
```

- `scrape_interval`: how often Prometheus scrapes metrics from all targets (default = 1m, here = 15s).
    
- `evaluation_interval`: how often rule expressions (`.rules.yml`) are evaluated.
    

👉 Think of **scrape_interval** = "how often to collect data",  
and **evaluation_interval** = "how often to check alerts/record rules."

* * *

## 2️⃣ Alertmanager Config

```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - 'localhost:9093'
```

- Tells Prometheus where to send firing alerts.
    
- Here, Alertmanager is running locally on port **9093**.
    
- `static_configs` just means we define the addresses manually.
    

📌 If you had multiple Alertmanagers (HA setup), you’d list them all under `targets`.

* * *

## 3️⃣ Rule Files

```yaml
rule_files:
  - "prometheus.rules.yml"
  - "alert.rules.yml"
```

- Prometheus loads these YAML files that define **alerting rules** and **recording rules**.
    
- Example inside `alert.rules.yml`:
    

```yaml
groups:
  - name: instance_down
    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} is down"
```

👉 These rules generate alerts that are sent to Alertmanager.

* * *

## 4️⃣ Scrape Configs

This is where Prometheus learns what to scrape.

```yaml
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
        labels:
          app: "prometheus"
      - targets: ["172.31.12.108:9100"]
        labels:
          app: "appserver"
      - targets: ["172.31.14.189:9100"]
        labels:
          app: "appserver"
```

### 🔍 Breakdown:

- **job_name**: `"prometheus"`
    
    - Every scraped metric gets a `job="prometheus"` label.
        
    - Useful for grouping queries and alerts.
        
- **static_configs**:
    
    - Defines static target addresses to scrape.
        
    - Each target automatically gets an **`instance` label** (equal to its `host:port`), plus any extra labels you define here.
        

👉 Example:  
Scraping `172.31.12.108:9100` with labels →

```
job="prometheus"
instance="172.31.12.108:9100"
app="appserver"
```

- **labels** (inside static_configs):
    
    - Add extra labels to all metrics from this target.
        
    - In your case, metrics from `172.31.12.108:9100` and `172.31.14.189:9100` both get `app="appserver"`.
        

* * *

# 🧩 How Labels Build Up

For target `172.31.12.108:9100`:

- Prometheus auto-assigns:  
    `job="prometheus"`, `instance="172.31.12.108:9100"`
    
- You add:  
    `app="appserver"`
    

Final metric labels look like:

```
{job="prometheus", instance="172.31.12.108:9100", app="appserver"}
```

* * *

# ✅ Summary Table

| Section | Purpose |
| --- | --- |
| **global** | Default scrape & evaluation timings |
| **alerting** | Defines Alertmanager targets |
| **rule_files** | Loads alerting & recording rules |
| **scrape_configs** | Defines what targets to scrape and with what labels |

* * *

&nbsp;

# 🔔 Prometheus + Alertmanager Alerting Setup

## 1\. Prometheus Rules (Alerting Rules)

Prometheus doesn’t send notifications directly — instead, it **evaluates alerting rules** and forwards them to Alertmanager.

👉 Example: `alert.rules.yml`

```yaml
groups:
- name: Instance
  rules:
  - alert: InstanceDown
    expr: up == 0
    for: 5m
    labels:
      severity: page
```

### 🔍 What it does:

- **alert: InstanceDown** → Alert name.
    
- **expr: up == 0** → Triggers if the `up` metric reports the instance is down.
    
- **for: 5m** → Condition must be true for 5 minutes continuously before alert fires (avoids false alarms).
    
- **labels** → Extra metadata (e.g., severity).
    

✅ Validate rules with:

```bash
./promtool check rules alert.rules.yml
```

* * *

Perfect 👍 Let’s go step by step and structure the **Alertmanager setup** clearly.

* * *

# 🚨 Setting up Alertmanager

Alertmanager works with **Prometheus** to handle alerts: grouping, deduplication, routing, and sending notifications (email, Slack, etc.).

* * *

## 1️⃣ Download and Install Alertmanager

```bash
# Go to Prometheus downloads page or use wget
wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz

# Extract
tar -xvf alertmanager-0.27.0.linux-amd64.tar.gz

# Move into directory
cd alertmanager-0.27.0.linux-amd64
```

Inside you’ll see:

- `alertmanager` → binary
    
- `amtool` → CLI tool to check configs
    
- `alertmanager.yml` → default config file
    

* * *

## 2️⃣ Create Alertmanager User (Security Best Practice)

```bash
sudo useradd --no-create-home --shell /bin/false alertmanager
```

* * *

## 3️⃣ Move Binaries and Set Permissions

```bash
# Move binaries
sudo cp alertmanager /usr/local/bin/
sudo cp amtool /usr/local/bin/

# Create dirs for data and config
sudo mkdir /etc/alertmanager
sudo mkdir /var/lib/alertmanager

# Copy config
sudo cp alertmanager.yml /etc/alertmanager/

# Set permissions
sudo chown -R alertmanager:alertmanager /etc/alertmanager /var/lib/alertmanager
```

* * *

## 4️⃣ Configure `alertmanager.yml`

Example **basic config** with email + Slack:

```yaml
global:
  resolve_timeout: 5m
  smtp_smarthost: 'smtp.example.com:587'
  smtp_from: 'alertmanager@example.com'
  smtp_auth_username: 'alertmanager'
  smtp_auth_password: 'your_password'

route:
  receiver: 'default-receiver'
  group_by: ['alertname', 'instance']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h

receivers:
  - name: 'default-receiver'
    email_configs:
      - to: 'ops@example.com'

  - name: 'slack-notifications'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/XXX/YYY/ZZZ'
        channel: '#alerts'

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'instance']
```

✅ This means:

- Alerts group by `alertname` + `instance`
    
- Email goes to `ops@example.com`
    
- Slack alerts go to `#alerts`
    
- Warnings are suppressed if a Critical is already firing
    

* * *

## 5️⃣ Create Systemd Service

`/etc/systemd/system/alertmanager.service`

```ini
[Unit]
Description=Alertmanager
After=network-online.target
Wants=network-online.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
ExecStart=/usr/local/bin/alertmanager \
  --config.file=/etc/alertmanager/alertmanager.yml \
  --storage.path=/var/lib/alertmanager
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

* * *

## 6️⃣ Start and Enable Service

```bash
sudo systemctl daemon-reload
sudo systemctl start alertmanager
sudo systemctl enable alertmanager
```

Check status:

```bash
systemctl status alertmanager
```

* * *

## 7️⃣ Verify Setup

- Web UI:  
    👉 `http://<server-ip>:9093`
    
- Validate config:
    

```bash
amtool check-config /etc/alertmanager/alertmanager.yml
```

* * *

## 8️⃣ Connect Prometheus to Alertmanager

In `prometheus.yml`:

```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']
```

Reload Prometheus:

```bash
sudo systemctl reload prometheus
```

* * *

## 9️⃣ Test Alerts

Example alert rule (`alert.rules.yml`):

```yaml
groups:
- name: InstanceDown
  rules:
  - alert: InstanceDown
    expr: up == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} has been down for more than 1 minute."
```

Reload Prometheus to load new rules:

```bash
sudo systemctl reload prometheus
```

* * *

## 🔑 Summary

| Step | Purpose |
| --- | --- |
| Install binaries | Get Alertmanager |
| Create user + dirs | Security & data paths |
| Configure alertmanager.yml | Routing, receivers, inhibition |
| Create systemd service | Auto-start at boot |
| Start & enable | Run Alertmanager |
| Connect with Prometheus | Enable alert flow |
| Test rules | Verify alerts |

* * *

# 🎯 What Are Alert Labels?

- **Labels** are **key-value pairs** attached to alerts.
    
- They come from **Prometheus alerting rules**.
    
- Alertmanager uses these labels for **matching, routing, grouping, and inhibiting** alerts.
    

* * *

## 1️⃣ Defining Labels in Prometheus Rules

Example Prometheus alert rule:

```yaml
groups:
- name: instance-rules
  rules:
  - alert: InstanceDown
    expr: up == 0
    for: 1m
    labels:
      severity: critical
      team: backend
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "The instance {{ $labels.instance }} of job {{ $labels.job }} has been down for 1 minute."
```

### Breakdown:

- `alert: InstanceDown` → alert name (special label: `alertname`).
    
- `labels:` → custom labels (e.g., `severity`, `team`).
    
- `annotations:` → human-readable context, not used for routing.
    

* * *

## 2️⃣ Example Alert Sent to Alertmanager

When Prometheus triggers, it sends something like:

```json
{
  "labels": {
    "alertname": "InstanceDown",
    "instance": "10.0.0.1:9100",
    "job": "node_exporter",
    "severity": "critical",
    "team": "backend"
  },
  "annotations": {
    "summary": "Instance 10.0.0.1:9100 down",
    "description": "The instance 10.0.0.1 of job node_exporter has been down for 1 minute."
  },
  "startsAt": "2025-09-01T05:35:00Z"
}
```

* * *

## 3️⃣ How Labels Are Used in Alertmanager

- **Routing (match/match_re)**
    
    ```yaml
    route:
      receiver: "default"
      routes:
        - match:
            severity: critical
          receiver: "slack-critical"
        - match:
            team: backend
          receiver: "backend-email"
    ```
    
- **Grouping (group_by)**
    
    ```yaml
    route:
      group_by: ['alertname', 'team']
    ```
    
    → All alerts with the same `alertname` + `team` are grouped into one notification.
    
- **Inhibition (suppress lower severity)**
    
    ```yaml
    inhibit_rules:
      - source_match:
          severity: critical
        target_match:
          severity: warning
        equal: ['alertname', 'instance']
    ```
    

* * *

## 4️⃣ Labels vs Annotations

| Feature | Labels ✅ | Annotations 📝 |
| --- | --- | --- |
| Machine-usable | Yes (used for routing, grouping, matching) | No (only metadata for humans) |
| Shown in Alertmanager | Yes | Yes |
| Example | `severity=critical`, `team=backend` | `summary=Instance down`, `description=...` |

* * *

## 🔑 Example End-to-End Flow

1.  Prometheus fires an alert:  
    `{alertname="InstanceDown", instance="10.0.0.1:9100", severity="critical", team="backend"}`
    
2.  Alertmanager checks routes:
    
    - `severity=critical → receiver=slack-critical`
        
    - `team=backend → receiver=backend-email`
        
3.  If both match, it sends to **Slack + Email**.
    
4.  Annotations (summary/description) appear in the alert message.
    

* * *

# 🔍 Where Do Labels Like `instance` Come From?

Prometheus alerts are built on **time series data**, and **every time series already has labels**.  
When you write an alert rule, Prometheus **copies the labels from the matching series** into the alert.

* * *

## 1️⃣ Example Time Series

Suppose we scrape a Node Exporter running on `10.0.0.1:9100`.  
The metric `up` looks like this in Prometheus:

```
up{job="node_exporter", instance="10.0.0.1:9100"} = 1
```

- `job="node_exporter"` → comes from scrape config in `prometheus.yml`
    
- `instance="10.0.0.1:9100"` → automatically added by Prometheus (target address)
    

So the **`instance` label is always attached to scraped metrics**.

* * *

## 2️⃣ When Alert Rules Fire

Alert rule:

```yaml
- alert: InstanceDown
  expr: up == 0
  for: 1m
  labels:
    severity: critical
    team: backend
```

If `up{job="node_exporter", instance="10.0.0.1:9100"} == 0`,  
Prometheus fires an alert with **all the metric’s labels + custom labels**.

* * *

## 3️⃣ The Alert That Gets Sent

```json
{
  "labels": {
    "alertname": "InstanceDown",
    "instance": "10.0.0.1:9100",   👈 from scraped metric
    "job": "node_exporter",        👈 from scrape config
    "severity": "critical",        👈 from alert rule
    "team": "backend"              👈 from alert rule
  }
}
```

* * *

## 4️⃣ Summary

- **Scrape configs (`prometheus.yml`)** define `job` and target addresses → Prometheus adds `instance` & `job`.
    
- **Metrics themselves** may have labels (`mode`, `device`, etc.).
    
- **Alert rules** add extra labels (`severity`, `team`).
    
- Final alert = **union of all these labels**.
    

* * *

✅ So the `instance` label wasn’t missing — it came directly from the **scraped target**, not the alert rule.

* * *

## 2\. Alertmanager Configuration (alertmanager.yml)

Once Prometheus fires an alert, it’s handled by **Alertmanager**, which groups, routes, and sends notifications.

### Example: `alertmanager.yml`

```yaml
global:
  resolve_timeout: 3m
  smtp_smarthost: 'smtp.example.com:587'
  smtp_from: 'alertmanager@example.com'
  smtp_auth_username: 'alertmanager'
  smtp_auth_password: 'your_password'

route:
  receiver: 'default-receiver'
  group_by: ['alertname', 'instance']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h

receivers:
- name: 'default-receiver'
  email_configs:
  - to: 'ops@example.com'

- name: 'slack-notifications'
  slack_configs:
  - api_url: 'https://hooks.slack.com/services/XXX/YYY/ZZZ'
    channel: '#alerts'

inhibit_rules:
- source_match:
    severity: 'critical'
  target_match:
    severity: 'warning'
  equal: ['alertname', 'instance']

templates:
- '/etc/alertmanager/templates/*.tmpl'
```

### 🔍 Key Sections:

1.  **global** → Default settings (SMTP, timeout, etc.).
    
2.  **route** → Defines routing logic (grouping, retries, receivers).
    
3.  **receivers** → Notification destinations (email, Slack, webhook, etc.).
    
4.  **inhibit_rules** → Suppresses less severe alerts when a more severe one is firing.
    
5.  **templates** → Custom alert messages.
    

✅ Validate config with:

```bash
./amtool check-config alertmanager.yml
```

* * *

## 3\. `resolve_timeout` Explained

```yaml
global:
  resolve_timeout: 3m
```

- **Purpose:**  
    If Prometheus (or another alert source) stops sending updates for an alert, Alertmanager waits **3 minutes** before auto-resolving it.
    
- **How it works:**
    
    - Normally, Prometheus includes an **EndsAt timestamp** in alerts.
        
    - If that’s missing, `resolve_timeout` is used as a fallback.
        
- **Real-world effect:**  
    Prevents alerts from **lingering forever** in Alertmanager if the source never tells them to resolve.
    
- **Best practice:**  
    Keep between **3m–5m**, unless you expect long delays in updates.
    

* * *

## 4\. End-to-End Flow

1.  **Prometheus evaluates rules** (`alert.rules.yml`).
    
    - If `up == 0` for 5m → fires `InstanceDown`.
2.  **Alert sent to Alertmanager**.
    
    - Alertmanager applies `route` rules.
        
    - Groups by `alertname, instance`.
        
    - Sends to email + Slack.
        
3.  **Alert resolves** when:
    
    - Prometheus sends `EndsAt` update.
        
    - OR Alertmanager auto-resolves after `resolve_timeout: 3m`.
        

* * *

✅ With this setup, you now have:

- A working **InstanceDown alert** in Prometheus.
    
- **Alertmanager configured** to notify via Email + Slack.
    
- **Noise suppression** with `inhibit_rules`.
    
- **Auto-cleanup** of stale alerts via `resolve_timeout`.
    

* * *

Got it 👍 Let’s make this crystal clear and structured for you, Raushan. I’ll break it into **4 parts**: Receiver Types, Top-Level Routing, Nested Routes, and End-to-End Flow.

* * *

# 🧠 Alertmanager Configuration Breakdown

## 1️⃣ Receiver Types

Receivers define **where** alerts are sent. Common types:

| Receiver Type | Purpose | Example Use Case | Key Fields |
| --- | --- | --- | --- |
| **slack_configs** | Send alerts to Slack | Team chat alerts | `api_url`, `channel`, `username`, `http_config`, `send_resolved` |
| **webhook_configs** | Send alerts as HTTP POST requests | Custom dashboards, APIs, automation | `url`, `send_resolved` |
| **pagerduty_configs** | Send alerts to PagerDuty | Incident escalation, on-call rotation | `service_key`, `severity`, `class`, `group` |
| **email_configs** | Send alerts via SMTP email | Simple email notifications | `to`, `from`, `smarthost`, `auth_username`, `auth_password`, `require_tls` |

* * *

# 🧠 Receiver Types in Alertmanager

## 🔹 1. `slack_configs`

**Purpose:** Send alerts to **Slack channels or groups** for team visibility.  
**Best For:** Operational teams who want real-time notifications in chat.

✅ **Key Fields:**

- `api_url`: Slack Incoming Webhook URL OR Slack API endpoint
    
- `channel`: Target Slack channel (e.g., `#alerts` or channel ID like `C03HF6J97LH`)
    
- `username`: Display name of the sender (e.g., "AlertManager")
    
- `http_config`: Auth settings (e.g., bearer token for Slack API)
    
- `send_resolved`: Send a message when an alert resolves
    

📌 **Example:**

```yaml
receivers:
- name: 'slack-notifier'
  slack_configs:
  - api_url: https://slack.com/api/chat.postMessage
    channel: 'C03HF6J97LH'
    username: 'AlertManager'
    send_resolved: true
    http_config:
      bearer_token: 'xoxb-xxxxxxxx-xxxxxxxx-xxxxxxxx'
```

* * *

## 🔹 2. `webhook_configs`

**Purpose:** Send alerts as **HTTP POST requests** to a custom service.  
**Best For:** Triggering automation, updating dashboards, sending to internal APIs.

✅ **Key Fields:**

- `url`: Endpoint to receive the POST request
    
- `send_resolved`: Whether to notify when alert resolves
    

📌 **Example:**

```yaml
receivers:
- name: 'webhook-notifier'
  webhook_configs:
  - url: 'http://internal-dashboard.example.com/alerts'
    send_resolved: true
```

* * *

## 🔹 3. `pagerduty_configs`

**Purpose:** Send alerts to **PagerDuty** for **incident management & escalation**.  
**Best For:** Critical production issues requiring on-call response.

✅ **Key Fields:**

- `service_key`: PagerDuty integration/service key
    
- `severity`: Alert severity (info, warning, error, critical)
    
- `class`, `component`, `group`: Optional context for incidents
    

📌 **Example:**

```yaml
receivers:
- name: 'pagerduty-notifier'
  pagerduty_configs:
  - service_key: '7b673512c62b45e9922c33e9fb698ce8'
    severity: critical
    class: 'infrastructure'
    component: 'payment-api'
    group: 'backend'
```

* * *

## 🔹 4. `email_configs`

**Purpose:** Send alerts via **SMTP email**.  
**Best For:** Teams who prefer email-based alerting.

✅ **Key Fields:**

- `to`: Recipient email
    
- `from`: Sender email
    
- `smarthost`: SMTP server + port (e.g., `smtp.gmail.com:587`)
    
- `auth_username`: SMTP login username
    
- `auth_password`: SMTP login password
    
- `require_tls`: Ensure TLS is used when sending (recommended)
    
- `send_resolved`: Notify when alert resolves
    

📌 **Example:**

```yaml
receivers:
- name: 'email-notifier'
  email_configs:
  - to: 'devops-team@example.com'
    from: 'alerts@example.com'
    smarthost: 'smtp.gmail.com:587'
    auth_username: 'alerts@example.com'
    auth_password: 'super-secret-password'
    require_tls: true
    send_resolved: true
```

* * *

# 🧩 Summary

| Receiver Type | Use Case | Example Destination |
| --- | --- | --- |
| `slack_configs` | Team visibility & chat alerts | Slack channel `#alerts` |
| `webhook_configs` | Automation, dashboards, APIs | Internal service endpoint |
| `pagerduty_configs` | On-call incident escalation | PagerDuty incident service |
| `email_configs` | Simple email notifications | [devops-team@example.com](mailto:devops-team@example.com) |

* * *

## 2️⃣ Top-Level `route` Block

Defines **global routing logic**.

```yaml
route:
  group_by: ['alertname', 'instance', 'service']
  group_wait: 30s
  group_interval: 2m
  repeat_interval: 20m
  receiver: oauth-devops-slack-notifier
```

🔍 Meaning:

- **group_by**: Alerts with the same labels (`alertname`, `instance`, `service`) are grouped.
    
- **group_wait: 30s**: Wait before sending first notification (avoids flapping).
    
- **group_interval: 2m**: Time between grouped notifications.
    
- **repeat_interval: 20m**: Resend alert if still firing.
    
- **receiver**: Default receiver if no sub-route matches.
    

* * *

## 3️⃣ Nested `routes` Block

Sub-routes refine alert delivery based on labels.

### Example

```yaml
routes:
  - match:
      techteam: OauthTechops_ClientDev
    receiver: oauth-devops-slack-notifier-for-oauth-alert-group
    continue: true

  - match:
      severity: critical
      techteam: oauthdevops
    receiver: oauth-devops-pagerduty-notifier

  - match:
      category: oauth-info
    receiver: oauth-devops-slack-info-notifier
```

🔍 Meaning:

- **Label-based routing**: Alerts are matched by labels (`techteam`, `severity`, `category`).
    
- **continue: true**: After sending to one receiver, Alertmanager keeps checking further routes.
    
- **Examples**:
    
    - Alerts with `techteam: OauthTechops_ClientDev` → Sent to team’s Slack.
        
    - Alerts with `severity: critical` + `techteam: oauthdevops` → Sent to PagerDuty.
        
    - Alerts with `category: oauth-info` → Sent to info-only Slack channel.
        

* * *

## 4️⃣ End-to-End Flow (Prometheus → Alertmanager → Receiver)

1.  **Prometheus Fires Alert**

```yaml
- alert: InstanceDown
  expr: up == 0
  for: 1m
  labels:
    severity: critical
    techteam: oauthdevops
  annotations:
    summary: "Instance {{ $labels.instance }} is down"
```

Labels:

- `alertname: InstanceDown`
    
- `severity: critical`
    
- `techteam: oauthdevops`
    

* * *

2.  **Prometheus Pushes Alert → Alertmanager**  
    Defined in `prometheus.yml`:

```yaml
alerting:
  alertmanagers:
  - static_configs:
      - targets: ['localhost:9093']
```

* * *

3.  **Alertmanager Routes Alert**

- Matches sub-route with `severity: critical` + `techteam: oauthdevops`
    
- → Sends to **PagerDuty** via `oauth-devops-pagerduty-notifier`.
    

* * *

4.  **Receiver Sends Notification**  
    Example Slack receiver:

```yaml
receivers:
- name: oauth-devops-slack-notifier
  slack_configs:
  - api_url: https://slack.com/api/chat.postMessage
    channel: 'C03HF6J97LH'
    username: 'AlertManager'
    http_config:
      bearer_token: 'xoxb-...'
    send_resolved: true
```

* * *

## ✅ Summary

| Stage | What Happens |
| --- | --- |
| **Prometheus Rule** | Alert fires with labels (severity, team, etc.) |
| **Alert → Alertmanager** | Sent to Alertmanager API |
| **Routing** | Alertmanager matches labels → decides receiver |
| **Receiver** | Slack, Email, PagerDuty, or Webhook delivers alert |

* * *

&nbsp;

# 🧠 systemctl start vs systemctl enable

| Command | What It Does |
| --- | --- |
| `systemctl start` | Starts the service **immediately** in the current session |
| `systemctl enable` | Configures the service to **start automatically on boot** |

## 🔧 `systemctl start`

- Activates the service **right now**
    
- **Does not** persist across reboots
    
- **Example:**
    
    ```bash
    sudo systemctl start prometheus
    ```
    
- **Analogy:** Manual ignition — you start the engine, but it won’t restart automatically next time.
    

## 🔧 `systemctl enable`

- Creates systemd **symlinks** so the service starts on **boot**
    
- **Doesn’t** start the service immediately
    
- **Example:**
    
    ```bash
    sudo systemctl enable prometheus
    ```
    
- **Analogy:** Set your car to auto-start every morning — it won’t start **now**, but it will on next boot.
    

## 🧪 Want Both?

Start **now** and enable on **boot** in one go:

```bash
sudo systemctl enable --now prometheus
```

* * *

# ⚙️ Setting Prometheus as a systemd Service

## 🔎 Confirm full path to config

```bash
readlink -f prometheus.yml
# → /home/ec2-user/prometheus-3.6.0-rc.0.linux-amd64/prometheus.yml
```

## 🗂️ Unit file: `/etc/systemd/system/prometheus.service`

```ini
[Unit]
Description=Prometheus
After=network-online.target
Wants=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/home/ec2-user/prometheus-3.6.0-rc.0.linux-amd64/prometheus --config.file=/home/ec2-user/prometheus-3.6.0-rc.0.linux-amd64/prometheus.yml
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

* * *

# 🔐 Service Account & File Ownership

## 1) Create a non-login service user

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```

**Flags explained**

- `--no-create-home`: Don’t create `/home/prometheus` (service accounts don’t need it)
    
- `--shell /bin/false`: Disables interactive login
    
- `prometheus`: Username
    

✅ **Why:** Security best practice — Prometheus runs as its own user but cannot log in.

## 2) Give Prometheus ownership of its files

```bash
sudo chown -R prometheus:prometheus /home/ec2-user/prometheus-3.6.0-rc.0.linux-amd64
```

**What it does**

- `chown -R`: Recursively change owner/group
    
- Sets both **user** and **group** to `prometheus`
    
- Ensures the service can read/execute its binaries and configs
    

### 🧠 Why this combo matters

- Creates a dedicated, non-login service identity
    
- Ensures proper file permissions
    
- Prepares for secure systemd execution as `prometheus`
    

* * *

# 🧩 `[Install]` Section & `WantedBy=multi-user.target`

## 🧠 What is `[Install]`?

The `[Install]` section defines **how** a service is **enabled** (linked into the boot process).  
It answers: *“When someone runs `systemctl enable`, where should this service be hooked in?”*

## 🔗 What does `WantedBy=multi-user.target` mean?

“Start this service when the system reaches the **multi-user** state.”

### 🔍 What is `multi-user.target`?

- Equivalent to **runlevel 3** (classic SysVinit)
    
- Fully booted system **with networking & logins**, **no GUI**
    
- Typical for **servers/headless** setups
    

## 🧪 What happens when you run:

```bash
sudo systemctl enable prometheus
```

Systemd creates a symlink:

```
/etc/systemd/system/multi-user.target.wants/prometheus.service
  → /etc/systemd/system/prometheus.service
```

➡️ Prometheus will **auto-start** when the system boots into **multi-user** mode.

## 🧩 Quick Summary

| Directive | Purpose |
| --- | --- |
| `[Install]` | Defines how the service is **enabled** |
| `WantedBy=multi-user.target` | Hooks service into **server/CLI** boot target |

* * *

# 🧠 Common systemd Target Modes

| Target Unit | Equivalent Runlevel | Description |
| --- | --- | --- |
| `poweroff.target` | Runlevel 0 | Shuts down and powers off the system |
| `rescue.target` | Runlevel 1 | Single-user (maintenance); minimal services; no networking |
| `multi-user.target` | Runlevel 3 | CLI-only mode with networking; typical for servers |
| `graphical.target` | Runlevel 5 | Multi-user + graphical UI (X11/Wayland) |
| `reboot.target` | Runlevel 6 | Reboots the system |
| `default.target` | —   | Symlink to the default boot target (often multi-user/graphical) |
| `network.target` | —   | Basic network is up (not guaranteed usable) |
| `network-online.target` | —   | Network is fully **connected and usable** |

> These targets control when services start, what dependencies are required, and how the system behaves during boot/shutdown.

* * *

# 🔧 How to View & Switch Targets

## 🔍 View current default target

```bash
systemctl get-default
```

## 🔄 Change default boot mode (persistent)

```bash
sudo systemctl set-default multi-user.target
```

## 🚀 Boot into a specific mode (temporary)

At the **GRUB** prompt, append:

```
systemd.unit=rescue.target
```

## 🛠️ Custom Targets

You can create your own targets to define custom system states (e.g., **monitoring-only** with Prometheus/Alertmanager/Node Exporter).

* * *

# ✅ Quick “Next Steps” (optional)

- Enable & start the service: `sudo systemctl enable --now prometheus`
    
- Check status/logs: `systemctl status prometheus` / `journalctl -u prometheus -f`
    
- Reload after edits: `sudo systemctl daemon-reload && sudo systemctl restart prometheus`
    

&nbsp;

&nbsp;

* * *

# 🔎 Prometheus Service Discovery

## 🧠 What Is Service Discovery?

Service discovery answers the question:  
👉 *“How does Prometheus know which servers/applications (targets) to scrape metrics from — and how does it keep that list updated automatically?”*

* * *

## ❓ Why Do We Need It?

- In small setups → You can **hardcode IPs/ports** in `prometheus.yml`.
    
- In real-world setups (cloud, Kubernetes, autoscaling) → Servers are **dynamic** (added/removed frequently).
    
- Manually updating configs is **impossible and error-prone**.  
    ✅ Service discovery makes Prometheus automatically detect changes.
    

* * *

## ⚙️ How It Works

Prometheus integrates with your infrastructure provider or orchestrator to fetch the list of active targets.

* * *

## 📌 Types of Service Discovery

### 1\. **Static Config (Manual)**

Good for **small environments**.

```yaml
scrape_configs:
  - job_name: "node"
    static_configs:
      - targets: ["192.168.1.10:9100", "192.168.1.11:9100"]
```

👉 You list all targets manually.

* * *

### 2\. **Cloud-Based (EC2, GCP, Azure)**

Prometheus queries your **cloud provider API** for instances with specific tags.  
Example (AWS EC2): Scrape all instances tagged `monitoring:enabled`.

👉 Auto-updates when instances are added/removed.

* * *

### 3\. **Kubernetes**

Prometheus integrates with the **Kubernetes API**.  
It can discover:

- **Pods**
    
- **Nodes**
    
- **Services**
    
- **Ingress** endpoints
    

👉 Perfect for dynamic container environments.

* * *

### 4\. **Other Systems**

- **Consul** → Uses service registry.
    
- **Etcd, Zookeeper** → Distributed key-value stores for discovery.
    
- **DNS-based SD** → Prometheus resolves DNS names dynamically.
    

* * *

## 📝 Analogy

Think of Prometheus as a **delivery boy**:

- **Without service discovery** → You give him a handwritten list of addresses (static configs).
    
- **With service discovery** → He asks the **housing society’s directory** (Kubernetes API, AWS EC2 API, Consul) and automatically knows who lives there today.
    

* * *

✅ **In short:**  
Service discovery = *Prometheus staying in sync with your infrastructure so you don’t manually chase IPs.*

* * *

# 🧠 Prometheus: Jobs, Service Discovery & Relabeling

* * *

## 🔹 1. What is a **Job** in Prometheus?

A **job** is a logical group of scrape targets that share the same scrape configuration.

- Defined under `scrape_configs` → `job_name`
    
- Prometheus automatically adds:
    
    - `job` = job name
        
    - `instance` = target scraped (ip:port)
        

📌 **Example:**

```yaml
scrape_configs:
- job_name: "node_exporter"
  static_configs:
  - targets: ["10.0.0.1:9100", "10.0.0.2:9100"]

- job_name: "nginx"
  static_configs:
  - targets: ["10.0.0.1:9113"]
```

✅ Here:

- Metrics from first block get `job="node_exporter"`
    
- Metrics from second block get `job="nginx"`
    

👉 Think of `job` like a **folder name** that groups targets together.

* * *

## 🔹 2. Why Use **Labels** in Prometheus?

Labels add **context** to metrics → making queries in Grafana/alerts much more powerful.

- **Auto labels:**
    
    - `job` (from job_name)
        
    - `instance` (from target ip:port)
        
- **Custom labels:** via `relabel_configs` or `static_configs`
    

📌 **Example (custom env label):**

```yaml
scrape_configs:
- job_name: "ec2-nodes"
  ec2_sd_configs:
  - region: us-east-1
  relabel_configs:
  - target_label: "env"
    replacement: "production"
```

✅ Now every metric from these EC2 nodes will have `env="production"`.

* * *

## 🔹 3. Static Discovery (small setups)

Manually define targets.

📌 **Example:**

```yaml
scrape_configs:
- job_name: "node"
  static_configs:
  - targets: ["192.168.1.10:9100", "192.168.1.11:9100"]
```

👉 Best for small, fixed environments.

* * *

## 🔹 4. EC2 Service Discovery (`ec2_sd_configs`)

Prometheus queries **AWS EC2 API** for instances.

📌 **Example:**

```yaml
scrape_configs:
- job_name: "ec2-nodes"
  ec2_sd_configs:
  - region: us-east-1
    port: 9100
    filters:
    - name: tag:Role
      values: ["monitoring"]
  relabel_configs:
  - source_labels: [__meta_ec2_private_ip]
    target_label: instance
  - source_labels: [__meta_ec2_tag_Name]
    target_label: name
```

✅ Breakdown:

- `region`: AWS region (e.g., us-east-1)
    
- `port`: scrape port (Node Exporter = 9100)
    
- `filters`: only select EC2s with tag `Role=monitoring`
    
- `relabel_configs`:
    
    - Map AWS private IP → `instance`
        
    - Map AWS tag `Name` → `name`
        

👉 Prometheus auto-exposes metadata as `__meta_ec2_*` labels → you relabel them into usable labels.

* * *

## 🔹 5. Kubernetes Service Discovery (`kubernetes_sd_configs`)

Prometheus queries **Kubernetes API**.

📌 **Example (pods):**

```yaml
scrape_configs:
- job_name: "kubernetes-pods"
  kubernetes_sd_configs:
  - role: pod
  relabel_configs:
  - source_labels: [__meta_kubernetes_namespace]
    target_label: namespace
  - source_labels: [__meta_kubernetes_pod_name]
    target_label: pod
```

✅ Breakdown:

- `role`: what to discover:
    
    - `node` → cluster nodes
        
    - `pod` → pods
        
    - `service` → services
        
    - `endpoints` → service endpoints
        
- `relabel_configs`:
    
    - Convert `__meta_kubernetes_namespace` → `namespace`
        
    - Convert `__meta_kubernetes_pod_name` → `pod`
        

👉 This way you can query:

```promql
sum(rate(container_cpu_usage_seconds_total{namespace="prod"}[5m]))
```

* * *

## 🔹 6. Relabeling — The Glue

Relabeling **transforms discovery metadata into Prometheus labels**.

📌 Common patterns:

```yaml
relabel_configs:
- source_labels: [__meta_ec2_private_ip]
  target_label: instance

- source_labels: [__meta_ec2_tag_Env]
  target_label: env

- source_labels: [__meta_kubernetes_pod_label_app]
  target_label: app
```

✅ Examples:

- Turn EC2 `__meta_ec2_private_ip` → `instance=10.0.0.5`
    
- Turn AWS Tag `Env=prod` → `env="prod"`
    
- Turn K8s Pod Label `app=nginx` → `app="nginx"`
    

* * *

## 🧩 Summary

| Concept | Purpose | Example Label/Config |
| --- | --- | --- |
| **Job** | Logical group of scrape targets | `job="node_exporter"` |
| **Labels** | Metadata for filtering/grouping metrics | `instance="10.0.0.5:9100"`, `env="prod"` |
| **Static Discovery** | Manual target list | `targets: ["192.168.1.10:9100"]` |
| **EC2 SD** | Discover AWS EC2 instances by region/tags | `__meta_ec2_tag_Name → name` |
| **K8s SD** | Discover Pods, Nodes, Services via K8s API | `__meta_kubernetes_namespace → namespace` |
| **Relabeling** | Convert metadata → usable Prometheus labels | `__meta_* → app, env, name` |

&nbsp;

* * *

# 🧠 Prometheus Metrics, Queries & Service Discovery – Full Structured Guide

* * *

## 1\. Metric Types in Prometheus

Prometheus has 4 core metric types, each with distinct semantics and use cases.

### 🔹 Gauge

- **Definition**: A value that can go up *and* down.
    
- **Examples**:
    
    - Memory usage
        
    - CPU temperature
        
    - Number of items in a queue
        
- **Updates in code**:
    
    - `Set(value)` → set directly
        
    - `Inc()` / `Dec()` → increase/decrease by 1
        
    - `Add(n)` / `Sub(n)` → increase/decrease by n
        
- **Use in queries**:
    
    - Directly read (`memory_usage_bytes`)
        
    - Can calculate time since event: `time() - gauge`
        

* * *

### 🔹 Counter

- **Definition**: A value that only increases (resets on restart).
    
- **Examples**:
    
    - Total HTTP requests
        
    - Total errors logged
        
    - Total seconds process has run
        
- **Updates in code**:
    
    - `Inc()` or `Add(n)`
- **Important**: Counters *never* go down (drop = reset).
    
- **Use in queries**:
    
    - `rate(http_requests_total[5m])` → requests/sec
        
    - `increase(http_requests_total[1h])` → requests in last hour
        
- **Prometheus handles resets** automatically.
    

* * *

### 🔹 Summary

- **Definition**: Tracks quantiles (percentiles) of observations.
    
- **Example use case**: Request latency (e.g., 50%, 90%, 99% percentiles).
    
- **Exposed metrics**:
    
    - `http_request_duration_seconds{quantile="0.9"}`
        
    - `_count` → number of observations
        
    - `_sum` → total observed values
        
- **Limitation**:
    
    - Per-instance only → cannot aggregate across instances.

* * *

### 🔹 Histogram

- **Definition**: Buckets values into ranges to calculate distributions.
    
- **Example use case**: Request latency across services.
    
- **Exposed metrics**:
    
    - `_bucket{le="0.1"}`, `_bucket{le="0.5"}` etc.
        
    - `_count`
        
    - `_sum`
        
- **Use in queries**:
    
    - `histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))` → 95th percentile latency across services.
- **Tradeoff**:
    
    - More buckets = more accuracy but higher storage cost.
- **Advantage over Summary**: Aggregatable across instances.
    

* * *

### 🔹 Native Histograms (Experimental)

- Stores entire histogram in a single sample.
    
- Combines benefits of summaries and histograms.
    
- More efficient + aggregatable.
    

* * *

### ✅ Rule of Thumb

- **Gauge** → "Current value"
    
- **Counter** → "How many times something happened"
    
- **Summary** → "Per-instance quantiles only"
    
- **Histogram** → "Distributions and aggregatable percentiles"
    

* * *

## 2\. PromQL Data Selectors

Prometheus query language (PromQL) has multiple ways to select and manipulate time series data.

* * *

### 🔹 Instant Vector Selectors

- **Definition**: Selects the latest value at query evaluation time.
    
- **Example**: `demo_memory_usage_bytes`
    
- **Graph vs Table**:
    
    - **Table view** → one value per series
        
    - **Graph view** → multiple values plotted across evaluation timestamps
        

* * *

### 🔹 Label Matchers

- **Definition**: Filter time series by labels.
    
- **Types**:
    
    - `=` equality
        
    - `!=` inequality
        
    - `=~` regex match
        
    - `!~` negative regex
        
- **Examples**:
    
    - `{type="buffers"}`
        
    - `{type!="free"}`
        
    - `{type=~"buffers|cached"}`
        
    - `{type!~"buffers|cached"}`
        
- Multiple matchers can be combined.
    

* * *

### 🔹 Lookback Delta

- **Default**: 5 minutes
    
- **Purpose**: Prometheus looks back this long to find a sample if none exists at query timestamp.
    
- **If no sample** → series is dropped → graph gap.
    
- **Prevents** showing stale data as fresh.
    

* * *

### 🔹 Staleness Markers

- Prometheus writes ❌ when it detects a series has ended (e.g., scrape failure).
    
- Effect: Series stops appearing immediately, even within 5m lookback.
    
- If series reappears → it resumes.
    

* * *

### 🔹 Range Vector Selectors

- **Definition**: Selects multiple samples over a duration.
    
- **Syntax**: `[duration]`
    
- **Example**: `demo_memory_usage_bytes[1m]` → last 1 min of samples.
    
- **Important**: Cannot graph directly → must be reduced via functions like:
    
    - `rate()`
        
    - `avg_over_time()`
        
    - `increase()`
        

* * *

### 🔹 Offset Modifier

- **Definition**: Shifts query evaluation back in time.
    
- **Example**:
    
    - `demo_memory_usage_bytes offset 2m` → value as of 2m ago.
- **Use case**: Compare current vs past:
    
    ```
    demo_memory_usage_bytes - demo_memory_usage_bytes offset 1w
    ```
    
    → Difference vs one week ago.
    

* * *

### 🔹 Absolute Evaluation Timestamp (@ modifier)

- **Definition**: Fixes query evaluation to a specific time.
    
- **Example**:
    
    - `demo_memory_usage_bytes @ 1672531200` → evaluate at that UNIX timestamp.
- **Special keywords**:
    
    - `@ start` → start of graph range
        
    - `@ end` → end of graph range
        
- **Use case**: Show stable top-k series across a graph window.
    

* * *

### 🔹 Combining Modifiers

- **Order**:
    
    1.  Metric name
        
    2.  Label matchers `{}`
        
    3.  Range `[duration]`
        
    4.  Offset
        
    5.  @ absolute timestamp
        
- **Example**:
    
    ```
    demo_memory_usage_bytes{instance="inst1"}[1m] offset 1h @ 1672531200
    ```
    
    - Anchor = UNIX timestamp
        
    - Offset = shift 1h back
        
    - Range = 1 min samples
        
    - Final = samples between (t - 1h - 1m) and (t - 1h)
        

* * *

### ✅ Big Picture

- **Instant vector** → one value per series (now).
    
- **Range vector** → multiple values, needs function.
    
- **Lookback delta** → freshness guarantee.
    
- **Staleness markers** → explicit "end of series".
    
- **Offset** → relative time shift.
    
- **@** → absolute time anchor.
    
- **Combined** → full control over querying data in time.
    

* * *

&nbsp;