&nbsp;

## 🔹 What is Promtail?

Promtail is an **agent for Loki** (the log aggregation system in Grafana’s observability stack).  
It runs on your servers, **collects logs from files (or systemd/journal/syslog), processes them, and pushes them to Loki**.

Think of it like how **Node Exporter** sends system metrics to Prometheus — but here, **Promtail sends logs to Loki**.

* * *

## 🔹 Promtail Architecture

- **Sources** → Where logs come from (e.g., `/var/log/syslog`, `/var/log/nginx/*.log`, journald, syslog).
    
- **Pipeline stages** → Transform logs (parse JSON, regex, add labels, drop unwanted logs).
    
- **Targets** → Loki (most common), but Promtail can also write to stdout, files, etc.
    

* * *

## 🔹 Installing Promtail

### 1\. Download Promtail (Linux example)

```bash
wget https://github.com/grafana/loki/releases/latest/download/promtail-linux-amd64.zip
unzip promtail-linux-amd64.zip
chmod +x promtail-linux-amd64
sudo mv promtail-linux-amd64 /usr/local/bin/promtail
```

### 2\. Create a Config File

Default config: `/etc/promtail/promtail-config.yaml`

Example minimal config:

```yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://localhost:3100/loki/api/v1/push  # Loki endpoint

scrape_configs:
  - job_name: system
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*.log
```

Here:

- **positions.yaml** → Keeps track of last read log line.
    
- **scrape_configs** → Defines where to collect logs from.
    
- ****path**** → Which files to read.
    

* * *

## 🔹 Running Promtail

Run it manually:

```bash
promtail -config.file=/etc/promtail/promtail-config.yaml
```

Or as a **systemd service**:

```ini
# /etc/systemd/system/promtail.service
[Unit]
Description=Promtail service
After=network.target

[Service]
ExecStart=/usr/local/bin/promtail -config.file=/etc/promtail/promtail-config.yaml
Restart=always

[Install]
WantedBy=multi-user.target
```

Then:

```bash
sudo systemctl daemon-reload
sudo systemctl enable promtail
sudo systemctl start promtail
```

* * *

## 🔹 Adding Custom Configs

Promtail is very flexible. Some useful examples:

### 1\. Scrape Nginx logs

```yaml
- job_name: nginx
  static_configs:
    - targets:
        - localhost
      labels:
        job: nginx
        __path__: /var/log/nginx/*.log
```

### 2\. Parse JSON logs

```yaml
pipeline_stages:
  - json:
      expressions:
        level: level
        msg: message
  - labels:
      level:
```

### 3\. Drop unwanted logs

```yaml
pipeline_stages:
  - drop:
      expression: "DEBUG"
```

* * *

## 🔹 Workflow in Real Monitoring Setup

1.  **Promtail** runs on each host → ships logs to **Loki**.
    
2.  **Loki** stores and indexes the logs.
    
3.  **Grafana** queries Loki to visualize logs (with filters, labels, etc.).
    

* * *

&nbsp;

* * *

# 🔹 Promtail Config File – Structure

A Promtail config file is written in **YAML**.  
Main sections:

1.  **server**
    
2.  **positions**
    
3.  **clients**
    
4.  **scrape_configs**
    
5.  **limits_config** (optional)
    
6.  **target_config** (optional)
    
7.  **tracing, analytics, metrics** (optional)
    

* * *

## 1️⃣ **server**

Controls how Promtail itself exposes its **web server** (for metrics, health checks, debug).

```yaml
server:
  http_listen_port: 9080   # Expose metrics, health, and ready endpoints
  grpc_listen_port: 0      # 0 disables gRPC
```

Endpoints exposed:

- `http://localhost:9080/metrics` → Prometheus-style metrics.
    
- `http://localhost:9080/ready` → Readiness probe.
    
- `http://localhost:9080/health` → Health probe.
    

* * *

## 2️⃣ **positions**

Promtail remembers the **last read line** in a file so it doesn’t resend logs after restart.

```yaml
positions:
  filename: /tmp/positions.yaml
```

This YAML file stores offsets like:

```yaml
/var/log/syslog: "12345678"
```

* * *

## 3️⃣ **clients**

Where Promtail should send logs (**Loki endpoint**).

```yaml
clients:
  - url: http://localhost:3100/loki/api/v1/push
    batchwait: 1s
    batchsize: 1048576  # 1 MB
    tenant_id: "team-a"
```

- **url** → Loki push endpoint.
    
- **batchwait** → Time to wait before sending logs batch.
    
- **batchsize** → Max batch size before pushing.
    
- **tenant_id** → For multi-tenancy setups.
    

* * *

## 4️⃣ **scrape_configs**

This is the **heart** of Promtail – defines what logs to scrape and how to process them.

```yaml
scrape_configs:
  - job_name: varlogs
    static_configs:
      - targets:
          - localhost
        labels:
          job: varlogs
          __path__: /var/log/*.log
    pipeline_stages:
      - regex:
          expression: "level=(?P<level>[A-Z]+)"
      - labels:
          level:
```

Breakdown:

- **job_name** → Logical name for this scrape.
    
- **static_configs** → Defines sources (log paths).
    
    - `__path__` → File pattern to watch.
        
    - `labels` → Tags attached to logs (used in queries in Grafana/Loki).
        
- **pipeline_stages** → Transformation steps:
    
    - **regex/json** → Extract fields.
        
    - **labels** → Convert fields into Loki labels.
        
    - **drop** → Drop unwanted logs.
        

* * *

## 5️⃣ **limits_config** (optional)

Controls memory, log line size, and ingestion rate.

```yaml
limits_config:
  readline_rate: 100   # Max lines per second per target
  max_line_size: 102400 # 100 KB
```

* * *

## 6️⃣ **target_config** (optional)

Controls how log targets are watched.

```yaml
target_config:
  sync_period: 10s   # How often files are scanned for changes
```

* * *

## 7️⃣ **tracing / analytics**

For distributed tracing or telemetry. Usually disabled in small setups.

* * *

# 🔹 Promtail Metrics

Promtail itself exports **Prometheus metrics** (so you can monitor Promtail with Prometheus).

Visit → `http://localhost:9080/metrics`

Common metrics include:

### 1\. Log ingestion

- `promtail_read_bytes_total` → Total bytes read from files.
    
- `promtail_read_lines_total` → Total log lines read.
    
- `promtail_dropped_lines_total` → Lines dropped due to config rules.
    
- `promtail_parsed_lines_total` → Lines successfully parsed by pipelines.
    

### 2\. Client metrics

- `promtail_client_sent_bytes_total` → Bytes sent to Loki.
    
- `promtail_client_requests_total` → Number of push requests sent.
    
- `promtail_client_errors_total` → Failed pushes to Loki.
    

### 3\. File tracking

- `promtail_targets_active_total` → Number of active targets (files).
    
- `promtail_positions_lines_total` → Number of lines tracked in positions file.
    

### 4\. Resource usage

- `promtail_go_memstats_alloc_bytes` → Memory allocated.
    
- `promtail_go_goroutines` → Number of goroutines.
    

* * *

# 🔹 Summary

- **Config file parts:** `server`, `positions`, `clients`, `scrape_configs`, (optional: limits, target, tracing).
    
- **Key metrics:** ingestion (`read_lines_total`), client pushes (`client_sent_bytes_total`), drops/errors, active targets.
    

* * *

&nbsp;

* * *

## 🔹 1. Extract Metrics from Logs (Log-based Metrics)

Promtail has **pipeline stages** where you can parse logs and turn fields into **labels**.  
Once logs are in **Loki**, you can create **metrics in Grafana** using **LogQL**.

For example, suppose your app writes logs like:

```
2025-09-01 12:00:01 INFO Request processed in 120ms
2025-09-01 12:00:02 INFO Request processed in 95ms
```

Promtail config:

```yaml
pipeline_stages:
  - regex:
      expression: "processed in (?P<duration_ms>[0-9]+)ms"
  - metrics:
      request_duration:
        type: histogram
        description: "Request duration from logs"
        source: duration_ms
```

👉 This creates a **Prometheus metric** (`request_duration_bucket`, etc.) from your logs, exposed at Promtail’s `/metrics`.  
Now Prometheus can scrape it!

* * *

## 🔹 2. Use Loki + LogQL to Create Metrics

Even if you don’t configure Promtail pipelines, you can use **Loki queries** in Grafana to turn logs into metrics.

Example (count all ERROR logs per second):

```logql
count_over_time({job="myapp"} |= "ERROR" [1m])
```

This turns logs into a Prometheus-style metric in Grafana dashboards.

* * *

## 🔹 3. Export Metrics From Your App → Scrape Separately

Sometimes it’s better to instrument your application directly:

- Use **Prometheus client libraries** (`prometheus_client` in Python, `prom-client` in Node.js, `prometheus-net` in C#, etc.).
    
- Expose metrics at `/metrics`.
    
- Let **Prometheus scrape it directly**, instead of Promtail.
    

This is preferred when you want **true numeric metrics** (CPU usage, request counts, durations), instead of parsing logs.

* * *

## 🔹 Example: Custom Metric from Logs in Promtail

Say your app writes login attempts like:

```
2025-09-01 12:00:00 LOGIN user=alice status=success
2025-09-01 12:00:01 LOGIN user=bob status=failed
```

Promtail config:

```yaml
scrape_configs:
  - job_name: app_logs
    static_configs:
      - targets:
          - localhost
        labels:
          job: app
          __path__: /var/log/app.log
    pipeline_stages:
      - regex:
          expression: "LOGIN user=(?P<user>\\w+) status=(?P<status>\\w+)"
      - metrics:
          login_attempts_total:
            type: counter
            description: "Total login attempts"
            source: status
            config:
              match_all: true
```

This creates a **Prometheus counter metric** `login_attempts_total` that increases every time a log line matches.

* * *

✅ **Summary:**

- Default metrics = Promtail’s own ingestion stats.
    
- Custom metrics = You can extract fields from logs with `metrics` pipeline stage.
    
- Advanced metrics = Better to instrument your app directly + scrape with Prometheus.
    

* * *

&nbsp;

Great, let’s deep dive into **Promtail pipeline metrics** — because that’s the feature that lets you turn **log content into real Prometheus metrics** 🚀.

* * *

# 🔹 What is the `metrics` Pipeline Stage?

The `metrics` stage allows you to **extract values from logs** (using regex/json stages earlier in the pipeline) and then **export them as Prometheus metrics**.

These metrics are exposed on Promtail’s `/metrics` endpoint so Prometheus can scrape them.

* * *

# 🔹 Structure of the `metrics` Stage

The stage looks like this:

```yaml
pipeline_stages:
  - metrics:
      <metric_name>:
        type: <counter|gauge|histogram>
        description: "<text describing metric>"
        source: <log field>
        config:   # (optional) type-specific configs
          <options>
```

* * *

# 🔹 Metric Types & Options

### 1️⃣ **Counter**

- **Monotonically increasing** (like `http_requests_total`).
    
- Good for counting occurrences of events.
    

Example:

```yaml
- metrics:
    login_attempts_total:
      type: counter
      description: "Total number of login attempts"
      source: status
      config:
        match_all: true
```

Here:

- Every log with `status` increases the counter by `1`.
    
- If `status` is numeric, it increases by that value.
    

* * *

### 2️⃣ **Gauge**

- Stores a **value that can go up or down**.
    
- Good for tracking current states (temperature, queue length).
    

Example:

```yaml
- metrics:
    queue_size:
      type: gauge
      description: "Current queue size"
      source: queue_length
```

If a log line has `queue_length=42`, the gauge will be set to `42`.

* * *

### 3️⃣ **Histogram**

- Buckets values into ranges.
    
- Good for durations, sizes, or distributions.
    

Example:

```yaml
- metrics:
    request_duration_ms:
      type: histogram
      description: "Request duration in milliseconds"
      source: duration_ms
      config:
        buckets: [50, 100, 200, 500, 1000]  # ms
```

If logs contain `duration_ms=120`, Promtail puts it in the right bucket.

* * *

# 🔹 Workflow of a Metrics Pipeline

1.  **Log line ingested** → Promtail reads a line.
    
2.  **Parsing stage** (regex/json) extracts fields.
    
    ```yaml
    - regex:
        expression: "user=(?P<user>\\w+) duration=(?P<duration_ms>[0-9]+)"
    ```
    
3.  **Metrics stage** takes those fields and updates Prometheus-style metrics.
    
4.  Prometheus scrapes `/metrics` from Promtail.
    

* * *

# 🔹 Example – Full Config with Metrics

Suppose logs look like this:

```
2025-09-01 INFO user=alice duration=95ms status=success
2025-09-01 INFO user=bob duration=200ms status=failed
```

Promtail config:

```yaml
scrape_configs:
  - job_name: app_logs
    static_configs:
      - targets:
          - localhost
        labels:
          job: app
          __path__: /var/log/app.log
    pipeline_stages:
      - regex:
          expression: "user=(?P<user>\\w+) duration=(?P<duration_ms>[0-9]+)ms status=(?P<status>\\w+)"
      - metrics:
          login_attempts_total:
            type: counter
            description: "Total login attempts"
            source: status
            config:
              match_all: true
          request_duration_ms:
            type: histogram
            description: "Request duration"
            source: duration_ms
            config:
              buckets: [50, 100, 200, 500, 1000]
```

Promtail will expose:

- `login_attempts_total` → counter
    
- `request_duration_ms_bucket` → histogram buckets
    
- `request_duration_ms_sum` → total duration
    
- `request_duration_ms_count` → number of samples
    

* * *

# 🔹 Limitations

- Metrics are **in Promtail, not in Loki** (they go to Prometheus, not Loki).
    
- Too many custom metrics can bloat Promtail.
    
- If you need **high-cardinality metrics** (like per-user), better to use logs in Loki or instrument your app directly.
    

* * *

✅ **In summary**:  
The `metrics` pipeline stage lets you **define custom counters, gauges, and histograms** from log fields. It takes `type`, `description`, `source` (field name), and `config` (buckets or match rules).

* * *

👉 **Regex is usually the stage before `metrics`**, because `metrics` needs structured fields, not raw log lines.

Let’s break it down.

* * *

# 🔹 Why Regex Before Metrics?

- Logs are **unstructured text** (e.g., `2025-09-01 LOGIN user=alice status=failed`).
    
- The `regex` stage extracts **named fields** (`user`, `status`, etc.).
    
- The `metrics` stage then takes those fields and **turns them into counters, gauges, or histograms**.
    

* * *

# 🔹 Example 1: Counting Failed Logins

Log line:

```
2025-09-01 12:00:00 LOGIN user=alice status=failed
```

Promtail pipeline:

```yaml
pipeline_stages:
  - regex:
      expression: "user=(?P<user>\\w+) status=(?P<status>\\w+)"
  - metrics:
      failed_logins_total:
        type: counter
        description: "Number of failed login attempts"
        source: status
        config:
          value: "failed"   # only increment when status == failed
```

### How it works:

1.  Regex extracts → `user="alice"`, `status="failed"`.
    
2.  Metrics stage checks → is `status="failed"`? Yes → increment counter.
    
3.  Prometheus sees →
    
    ```
    failed_logins_total 1
    ```
    

* * *

# 🔹 Example 2: Measuring Request Duration

Log line:

```
2025-09-01 INFO request_id=99 duration=120ms
```

Pipeline:

```yaml
pipeline_stages:
  - regex:
      expression: "duration=(?P<duration_ms>[0-9]+)ms"
  - metrics:
      request_duration_ms:
        type: histogram
        description: "Request duration in ms"
        source: duration_ms
        config:
          buckets: [50, 100, 200, 500, 1000]
```

### How it works:

1.  Regex extracts → `duration_ms=120`.
    
2.  Metrics stage takes `duration_ms` → puts it in histogram.
    
3.  Prometheus scrapes:
    
    ```
    request_duration_ms_bucket{le="200"} 1
    request_duration_ms_count 1
    request_duration_ms_sum 120
    ```
    

* * *

# 🔹 Example 3: Regex with Multiple Fields

Log line:

```
2025-09-01 WARN service=payment user=bob error=timeout
```

Pipeline:

```yaml
pipeline_stages:
  - regex:
      expression: "service=(?P<service>\\w+) user=(?P<user>\\w+) error=(?P<error>\\w+)"
  - metrics:
      service_errors_total:
        type: counter
        description: "Errors per service"
        source: error
        config:
          match_all: true
```

Now Prometheus will see:

```
service_errors_total{service="payment", user="bob", error="timeout"} 1
```

* * *

# 🔹 Key Points About Regex + Metrics

- **Regex must define named capture groups** (`?P<fieldname>`) → these become fields.
    
- `metrics.source` must refer to one of those extracted fields.
    
- `config.value` (for counters) or `buckets` (for histograms) refine how the metric behaves.
    
- Labels can also be applied → you can use regex fields as **labels**.
    

* * *

✅ **In short**:

- **Regex extracts fields from raw logs.**
    
- **Metrics stage converts extracted fields into Prometheus metrics.**
    

* * *

&nbsp;

# Promtail as a DaemonSet (Kubernetes):

- **Ensures per-node deployment:**
    
    A DaemonSet guarantees that a single instance of the Promtail Pod runs on every eligible node in the Kubernetes cluster. This is ideal for log collection, as Promtail can then access and ship logs from all applications and system components on that specific node.
    
- **Automatic scaling and self-healing:**
    
    When new nodes are added to the cluster, the DaemonSet automatically deploys Promtail to them. If a Promtail Pod fails or is terminated, the DaemonSet ensures it's restarted.
    
- **Centralized management:**
    
    DaemonSets are managed through Kubernetes manifests, allowing for declarative configuration and easy updates or rollbacks across the entire cluster.
    
- **Designed for cluster-wide services:**
    
    DaemonSets are specifically designed for running cluster-level services like log collectors (Promtail, Fluentd), monitoring agents (Node Exporter), or network proxies on every node.
    

# Promtail as a Standalone Container (or "on container" in a non-Kubernetes context):

- **Manual deployment and management:**
    
    Running Promtail as a standalone container means you are responsible for deploying, managing, and ensuring its availability on each machine where you want to collect logs. This typically involves using tools like Docker Compose, systemd services, or manual `docker run` commands.
    
- **Node-specific configuration:**
    
    Each standalone container instance needs to be configured individually for its specific host, including log paths and Loki endpoint.
    
- **No inherent self-healing:**
    
    If the Promtail container crashes, you would need external mechanisms (e.g., a process manager) to restart it.
    
- **Suitable for non-Kubernetes environments:**
    
    This approach is necessary when you are not using Kubernetes and need to collect logs from individual servers or virtual machines.
    

In summary:

- **DaemonSet**
    
    is the preferred and most efficient way to run Promtail within a Kubernetes cluster, ensuring comprehensive log collection and simplified management.
    
- **Standalone container**
    
    is used outside of Kubernetes or for specific, isolated use cases where per-node deployment via a DaemonSet is not applicable or desired.
    

&nbsp;

&nbsp;

# ✅ File System Type - **Rule of thumb in DevOps:**

- **ext4** → default / safe choice.
    
- **xfs** → best for databases & big volumes.
    
- **tmpfs** → for speed & ephemeral data.
    
- **nfs** → for shared storage in clusters.
    
- **overlayfs** → container internals.
    
- ntfs -> in windows

&nbsp;

# **Block Storage vs Object Storage**.

|     | **Block Storage** (like EBS) | **Object Storage** (like S3) |
| --- | --- | --- |
| Data unit | Blocks (fixed-size chunks) | Objects (file + metadata + ID) |
| Access | Via filesystem (ext4, NTFS, etc.) | Via REST API / HTTP |
| Latency | Low (fast I/O) | Higher (API-based) |
| Scalability | Limited per volume | Virtually unlimited |
| Best for | Databases, VM disks, apps needing fast read/write | Backups, archives, static content, logs |
| Examples | AWS EBS, GCP Persistent Disk | AWS S3, Azure Blob, GCP Cloud Storage |