Let’s go step by step:

* * *

## 🔹 What is Telegraf?

- **Telegraf** is a **lightweight agent** written in Go.
    
- It’s mainly used to **collect, process, aggregate, and send metrics & logs**.
    
- Works with **plugins** (like inputs, outputs, processors, and aggregators).
    
- Commonly paired with:
    
    - **InfluxDB** (time-series DB)
        
    - **Prometheus** (via exporters / remote write)
        
    - **Grafana** (for visualization)
        

* * *

## 🔹 Telegraf Architecture

Telegraf works as a **pipeline**:

```
(Input Plugins) → (Processor Plugins) → (Aggregator Plugins) → (Output Plugins)
```

1.  **Input Plugins** → collect metrics/logs/events  
    (e.g., CPU, memory, Docker stats, MySQL metrics, Prometheus scrape, etc.)
    
2.  **Processor Plugins** → modify or enrich metrics  
    (e.g., regex tagging, filtering, drop/add fields)
    
3.  **Aggregator Plugins** → combine metrics over time  
    (e.g., calculate averages, histograms, min/max)
    
4.  **Output Plugins** → send metrics to destinations  
    (e.g., InfluxDB, Prometheus remote write, Kafka, Loki, Elasticsearch)
    

* * *

## 🔹 Installation

Example (Ubuntu/Debian):

```bash
# Add InfluxData repo
wget -qO- https://repos.influxdata.com/influxdata-archive.key | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/influxdata.gpg
echo "deb https://repos.influxdata.com/debian stable main" | sudo tee /etc/apt/sources.list.d/influxdata.list

# Install telegraf
sudo apt-get update && sudo apt-get install telegraf -y

# Start service
sudo systemctl enable --now telegraf
```

* * *

## 🔹 Telegraf Config File

Default location:  
`/etc/telegraf/telegraf.conf`

It has **4 main sections**:

1.  **Global Agent Settings**
    
    ```toml
    [agent]
      interval = "10s"   # How often to collect metrics
      round_interval = true
    ```
    
2.  **Input Plugins**
    
    ```toml
    [[inputs.cpu]]
      percpu = true
      totalcpu = true
      collect_cpu_time = false
    ```
    
3.  **Processor / Aggregator Plugins**
    
    ```toml
    [[processors.regex]]
      namepass = ["cpu"]
      [[processors.regex.tags]]
        key = "cpu"
        pattern = "cpu0"
        replacement = "main_core"
    ```
    
4.  **Output Plugins**
    
    ```toml
    [[outputs.influxdb]]
      urls = ["http://localhost:8086"]
      database = "telegraf"
    ```
    

* * *

# 🔹 How Telegraf metrics are exposed

1.  **Telegraf has lots of input plugins built-in**, but they are **inactive by default**.
    
2.  **Only when you add a plugin block** (like `[[inputs.net_response]]` or `[[inputs.cpu]]`) to `telegraf.conf` or a `.d` config:
    
    - Telegraf starts collecting data for that plugin.
        
    - Metrics specific to that plugin are generated.
        
3.  **Telegraf outputs metrics** via whatever output plugin you have (e.g., `[[outputs.prometheus_client]]`), so Prometheus can scrape them.
    

## 🔹 Metrics with Telegraf

By default, Telegraf collects **system metrics**:

- CPU, memory, disk, network

But you can also collect from:

- **Applications** → MySQL, PostgreSQL, Nginx, Kafka, Redis
    
- **Cloud Providers** → AWS, GCP, Azure
    
- **Containers** → Docker, Kubernetes
    
- **Logs** → via file tailing or Loki output
    

* * *

## 🔹 Example Use Case

👉 Suppose you want to monitor **MySQL + System metrics**, and send them to **Prometheus**.

**Config Snippet:**

```toml
# Input system metrics
[[inputs.cpu]]
[[inputs.mem]]
[[inputs.disk]]
[[inputs.mysql]]
  servers = ["root:password@tcp(127.0.0.1:3306)/"]

# Output to Prometheus
[[outputs.prometheus_client]]
  listen = ":9273"
```

Now Prometheus can scrape metrics at `http://localhost:9273/metrics`.

* * *

&nbsp;

* * *

# 🔹 In Telegraf, every metric has 3 parts:

```
<measurement>,<tags> <fields> <timestamp>
```

Example:

```
cpu,cpu=cpu0,host=server1 usage_user=2.3,usage_idle=97.7 1693559010000000000
```

* * *

## 1\. **Measurement**

- The **name of the metric family**.
    
- Example: `cpu`, `mem`, `disk`, `net`.
    

* * *

## 2\. **Tags (metadata / labels)**

- **Key-value pairs**, always **strings**.
    
- Used for **identification, grouping, filtering** (not math!).
    
- Examples:
    
    - `host=server1`
        
    - `cpu=cpu0`
        
    - `interface=eth0`
        
    - `device=sda1`
        

👉 Think of **tags like SQL `WHERE` filters**. They are indexed and fast for lookups.

* * *

## 3\. **Fields (actual values)**

- **Key-value pairs**, values can be `int`, `float`, `bool`, `string`.
    
- This is the **actual metric data** you want to analyze.
    
- Examples:
    
    - `usage_user=2.3`
        
    - `usage_idle=97.7`
        
    - `bytes_sent=123456`
        
    - `used_percent=55.1`
        

👉 Fields are the **numbers you graph, alert on, and do math with**.

* * *

## ⚖️ Key Difference Between Tags vs Fields

| Feature | Tags | Fields |
| --- | --- | --- |
| Data type | **String only** | Int, Float, Bool, String |
| Indexed? | ✅ Yes (fast queries) | ❌ No (slower for filtering) |
| Use case | Filtering, grouping (like dimensions) | Actual measurement values |
| Example | `cpu=cpu0`, `host=server1` | `usage_user=2.3`, `bytes_sent=12345` |

* * *

## 🔹 Example: `[[inputs.cpu]]`

```toml
[[inputs.cpu]]
  percpu = true
  totalcpu = true
```

Metrics produced (simplified):

```
cpu,cpu=cpu0,host=server1 usage_user=2.3,usage_system=1.1,usage_idle=96.6
cpu,cpu=cpu1,host=server1 usage_user=3.5,usage_system=0.9,usage_idle=95.6
cpu,cpu=cpu-total,host=server1 usage_user=2.9,usage_system=1.0,usage_idle=96.1
```

- **Measurement** = `cpu`
    
- **Tags** = `cpu=cpu0`, `host=server1`
    
- **Fields** = `usage_user=2.3`, `usage_system=1.1`, `usage_idle=96.6`
    

* * *

✅ In short:

- **Tags = metadata / dimensions**
    
- **Fields = numeric values you analyze**
    
- **Measurement = metric family name**
    

* * *

&nbsp;

Here’s the **key idea**:

- In **Telegraf**, the **fields and tags are mostly fixed** by the **plugin developer**.
    
- You **don’t define fields** — those are the actual metric values the plugin collects (like `usage_user`, `bytes_sent`, etc.).
    
- You **can add or modify tags** (extra metadata) using Telegraf’s config or processors.
    

* * *

# 🔹 1. Who defines **fields**?

- Every **input plugin** has a **fixed set of fields** it exports.
    
- Example: `[[inputs.cpu]]` → always exports `usage_user`, `usage_system`, `usage_idle`, etc.
    
- You can’t just “add a new field” inside the plugin config.
    
- If you need a new custom metric, you use:
    
    - `[[inputs.exec]]` (run your script and output metrics), or
        
    - `[[inputs.http]]` (scrape a JSON/metrics endpoint), or
        
    - `processors.converter` (change type/units).
        

So → **fields are plugin-defined**.

* * *

# 🔹 2. Who defines **tags**?

- Plugins provide **some default tags** (`host`, `cpu`, `interface`, `device` etc.).
    
- You can **add your own custom tags** in the config:
    

```toml
[global_tags]
  env = "production"
  region = "us-east-1"
```

This means **every metric collected** will carry those tags.

- You can also add per-plugin tags:

```toml
[[inputs.cpu]]
  percpu = true
  totalcpu = true
  [inputs.cpu.tags]
    service = "backend"
```

- You can also **manipulate tags** using:
    
    - `processors.regex` (extract values into tags)
        
    - `processors.enum` (map codes to tag labels)
        

So → **tags can be customized freely**.

* * *

# 🔹 4. Example Breakdown: `[[inputs.disk]]`

Config:

```toml
[[inputs.disk]]
  ignore_fs = ["tmpfs", "devtmpfs"]
  [inputs.disk.tags]
    role = "database"
```

Metrics produced:

```
disk,host=server1,device=sda1,role=database used=12345678i,free=23456789i,used_percent=55.1 1693559000
```

- **Measurement** = `disk`
    
- **Tags** = `host=server1`, `device=sda1`, `role=database`
    
- **Fields** = `used`, `free`, `used_percent`
    

You **didn’t define `used` or `free`** (they are fixed).  
But you **added `role=database`** as a custom tag.

* * *

# 🔹 5. Rule of Thumb

- **Fields** → fixed by plugin (you consume them).
    
- **Tags** → extendable by you (you decorate metrics with more metadata).
    

* * *

&nbsp;

&nbsp;

* * *

# 🔹 Global Sections

### `[global_tags]`

- Used to define **tags applied to all metrics**.
    
- Example:
    
    ```toml
    [global_tags]
      env = "prod"
      region = "us-east"
    ```
    
    → All metrics will automatically have `env=prod, region=us-east`.
    

* * *

### `[agent]`

Controls how Telegraf collects and sends metrics.

- `interval = "10s"`  
    → Collect metrics every 10 seconds (scrape frequency).
    
- `round_interval = true`  
    → Align collection to exact multiples of interval (e.g., 12:00:00, 12:00:10) instead of just “10s after startup”.
    
- `metric_batch_size = 1000`  
    → When flushing metrics to an output, Telegraf sends them in batches of up to 1000.
    
- `metric_buffer_limit = 10000`  
    → Max number of metrics Telegraf will buffer in memory if the output is slow/unreachable. Beyond this, old metrics get dropped.
    
- `collection_jitter = "0s"`  
    → Adds randomness to collection to avoid thundering herd (multiple Telegraf agents scraping same time).  
    Example: `collection_jitter = "1s"` → adds ±1s randomness.
    
- `flush_interval = "10s"`  
    → How often Telegraf pushes metrics to outputs.
    
- `flush_jitter = "0s"`  
    → Randomness added to flush timing. Helps spread load when many agents flush to same backend.
    
- `precision = "0s"`  
    → Timestamp precision (e.g., `"1s"`, `"1ms"`, `"1us"`). `"0s"` means “don’t modify, keep default precision”.
    
- `hostname = ""`  
    → Override hostname tag. Empty = use system hostname.
    

&nbsp;

- `omit_hostname = false`  
    → If true, no `host` tag will be added to metrics.
    
- `debug = false`  
    → If true, log debug info.
    
- `quiet = false`  
    → If true, suppress logging of successful metrics writes.
    
- `logfile = ""`  
    → If set, write logs to a file instead of stdout.
    

* * *

# 🔹 Input Plugins (Metrics Collection)

* * *

### `[[inputs.cpu]]`

Collects **CPU usage** metrics.

- `percpu = true`  
    → Collect per-CPU core metrics (cpu0, cpu1, …).
    
- `totalcpu = true`  
    → Collect aggregated “all CPUs” metrics.
    
- `collect_cpu_time = false`  
    → If true → export CPU time counters (seconds spent), not percentages.
    
- `report_active = false`  
    → If true → export CPU active time (i.e., non-idle).
    

**Metrics produced** (measurement = `cpu`):

- Tags: `cpu` (cpu0, cpu1, total), `host`
    
- Fields: `usage_user`, `usage_system`, `usage_idle`, `usage_iowait`, etc.
    

* * *

### `[[inputs.disk]]`

Collects **disk usage (filesystem) stats**.

- `ignore_fs = [...]`  
    → Ignore filesystems like `tmpfs`, `devtmpfs`, `overlay` etc. (pseudo/ephemeral filesystems not useful for monitoring).

**Metrics produced** (measurement = `disk`):

- Tags: `device`, `fstype`, `path`, `host`
    
- Fields: `total`, `used`, `free`, `used_percent`, `inodes_total`, etc.
    

* * *

### `[[inputs.diskio]]`

Collects **per-disk I/O stats**.

**Metrics produced** (measurement = `diskio`):

- Tags: `name` (device name), `host`
    
- Fields:
    
    - `reads`, `writes` (number of ops)
        
    - `read_bytes`, `write_bytes`
        
    - `read_time`, `write_time`
        
    - `io_time`, `weighted_io_time`
        

* * *

### `[[inputs.kernel]]`

Collects **Linux kernel stats**.

**Metrics produced** (measurement = `kernel`):

- Fields:
    
    - `boot_time` (epoch)
        
    - `context_switches`
        
    - `interrupts`
        
    - `processes_forked`
        
    - `entropy_avail` (randomness available)
        

* * *

### `[[inputs.mem]]`

Collects **system memory usage**.

**Metrics produced** (measurement = `mem`):

- Fields:
    
    - `total`, `used`, `free`, `available`
        
    - `used_percent`
        
    - `active`, `inactive`
        
    - `cached`, `buffered`
        

* * *

### `[[inputs.processes]]`

Collects **process state counts**.

**Metrics produced** (measurement = `processes`):

- Fields:
    
    - `total`, `sleeping`, `running`, `stopped`, `zombies`, `idle`

* * *

### `[[inputs.swap]]`

Collects **swap memory usage**.

**Metrics produced** (measurement = `swap`):

- Fields:
    
    - `total`, `used`, `free`
        
    - `used_percent`
        
    - `sin` (swap-in bytes)
        
    - `sout` (swap-out bytes)
        

* * *

### `[[inputs.system]]`

Collects **basic system stats**.

**Metrics produced** (measurement = `system`):

- Fields:
    
    - `uptime` (seconds since boot)
        
    - `uptime_format` (formatted string)
        
    - `n_users` (logged in users)
        
    - `load1`, `load5`, `load15` (CPU load averages)
        

* * *

# 🔹 Output Plugin

### `[[outputs.prometheus_client]]`

Exposes metrics in **Prometheus format** on an HTTP endpoint.

- `listen = ":9273"`  
    → Starts an HTTP server on port `9273`  
    → Prometheus scrapes metrics at `http://<host>:9273/metrics`

Example exposed metric:

```
# HELP cpu_usage_user CPU usage in user space
# TYPE cpu_usage_user gauge
cpu_usage_user{cpu="cpu0",host="server1"} 3.5
```

* * *

✅ **Summary**:

- `[agent]` → controls how/when metrics are collected and flushed.
    
- `[[inputs.*]]` → plugins that gather specific system stats.
    
- `[[outputs.*]]` → where metrics are sent (Prometheus in this case).
    

* * *

&nbsp;

&nbsp;

* * *

## 1\. inputs.cpu

From the Telegraf source, available configuration fields include:

- `percpu` *(boolean)*: collect metrics per CPU core (default: `true`)
    
- `totalcpu` *(boolean)*: report aggregated usage across all CPUs (default: `true`)
    
- `collect_cpu_time` *(boolean)*: report raw CPU times (seconds) instead of usage percentages (default: `false`)
    
- `report_active` *(boolean)*: include `usage_active` (non-idle CPU time) (default: `false`)
    
- `core_tags` *(boolean)*: add `core_id` and `physical_id` tags (default: `false`) ([GitHub](https://github.com/influxdata/telegraf/blob/master/plugins/inputs/cpu/cpu.go?utm_source=chatgpt.com "telegraf/plugins/inputs/cpu/cpu.go at master"))
    

* * *

## 2\. inputs.disk

According to the documentation, key configuration options include:

- `mount_points` *(array of strings)*: restrict monitoring to specific mount points
    
- `ignore_fs` *(array of fs types)*: ignore specific filesystem types (e.g., `tmpfs`, `overlay`)
    
- `ignore_mount_opts` *(array of strings)*: ignore mounts based on mount options (e.g., `bind`) ([Grafana Labs](https://grafana.com/grafana/dashboards/20165-server-stats/?utm_source=chatgpt.com "Telegraf InfluxDB Server stats"))
    

* * *

## 3\. inputs.diskio

Default Telegraf behavior is to gather I/O stats for all devices. Common config options:

- `devices` *(array of strings)*: restrict to specific disks
    
- `skip_serial_number` *(boolean)*: skip serial number lookup (improves performance)
    
- `skip_udev` *(boolean)*: skip udev calls (improves performance)
    
- `udev_data` *(boolean)*: include additional metadata from udev ([Grafana Labs](https://grafana.com/grafana/dashboards/20165-server-stats/?utm_source=chatgpt.com "Telegraf InfluxDB Server stats"), [Reddit](https://www.reddit.com/r/unRAID/comments/18481a7/telegraf_influx_how_to_collect_cpu_usage_by/?utm_source=chatgpt.com "Telegraf & Influx: How to collect CPU usage by Process?"))
    

* * *

## 4\. inputs.kernel

This plugin collects kernel statistics from `/proc/stat`. A key config option:

- `collect` *(array of strings)*: specify additional stats like `ksm` (Kernel Same-page Merging) ([Grafana Labs](https://grafana.com/grafana/dashboards/20165-server-stats/?utm_source=chatgpt.com "Telegraf InfluxDB Server stats"))

* * *

## 5\. inputs.mem & inputs.swap & inputs.system

These core system-level plugins have **no configurable attributes**—they run with defaults:

- `mem`: collects memory usage stats
    
- `swap`: collects swap usage stats
    
- `system`: collects system metrics like uptime, load averages ([Grafana Labs](https://grafana.com/grafana/dashboards/20165-server-stats/?utm_source=chatgpt.com "Telegraf InfluxDB Server stats"))
    

* * *

## 6\. inputs.processes

Collects process state metrics (running, sleeping, etc.):

- `use_sudo` *(boolean)*: on BSD, use `sudo` for `ps` command (not needed on Linux) ([Grafana Labs](https://grafana.com/grafana/dashboards/20165-server-stats/?utm_source=chatgpt.com "Telegraf InfluxDB Server stats"))

* * *

## Summary Table

| Plugin | Config Attributes |
| --- | --- |
| `inputs.cpu` | `percpu`, `totalcpu`, `collect_cpu_time`, `report_active`, `core_tags` |
| `inputs.disk` | `mount_points`, `ignore_fs`, `ignore_mount_opts` |
| `inputs.diskio` | `devices`, `skip_serial_number`, `skip_udev`, `udev_data` |
| `inputs.kernel` | `collect` (e.g., "ksm") |
| `inputs.mem`, `inputs.swap`, `inputs.system` | No config options available |
| `inputs.processes` | `use_sudo` (BSD systems only) |

* * *

## TL;DR

- **Fields and tags are fixed per plugin** – you can’t arbitrarily change them via config.
    
- **Config attributes control behavior** like what to collect or how to collect it.
    
- Some lightweight plugins—like `mem`, `swap`, and `system`—have no configuration options.
    
- Complex plugins (e.g., `disk`, `diskio`, `cpu`) have many toggles for fine control.
    

* * *

&nbsp;

## `[[inputs.http]]` Plugin

### 🔹 Purpose

- Fetches data from an HTTP(S) endpoint.
    
- Can collect JSON, XML, or plain text metrics.
    
- Useful for scraping metrics from custom apps, APIs, or web services.
    
    * * *
    
    🔹 `[[inputs.http]]` Plugin Options
    
    ### 1\. **`urls`**
    
    - **Type:** array of strings
        
    - **Required:** Yes
        
    - **Default:** `[]`
        
    - **Description:** List of HTTP(S) endpoints to scrape.
        
    - **Example:**
        
    
    ```toml
    urls = ["http://localhost:8080/metrics", "https://api.example.com/stats"]
    ```
    
    > Telegraf will send requests to each URL in this list.
    
    * * *
    
    ### 2\. **`method`**
    
    - **Type:** string
        
    - **Default:** `"GET"`
        
    - **Description:** HTTP method used to fetch the data (`GET`, `POST`, `PUT`, `DELETE`).
        
    - **Example:**
        
    
    ```toml
    method = "POST"
    ```
    
    > Useful if the endpoint requires a POST request with a payload.
    
    * * *
    
    ### 3\. **`headers`**
    
    - **Type:** table (key-value)
        
    - **Default:** `{}` (none)
        
    - **Description:** Custom HTTP headers to include in the request.
        
    - **Example:**
        
    
    ```toml
    headers = { Authorization = "Bearer TOKEN", "Content-Type" = "application/json" }
    ```
    
    * * *
    
    ### 4\. **`body`**
    
    - **Type:** string
        
    - **Default:** `""`
        
    - **Description:** HTTP request body used for POST/PUT requests.
        
    - **Example:**
        
    
    ```toml
    body = '{"query": "select * from metrics"}'
    ```
    
    * * *
    
    ### 5\. **`timeout`**
    
    - **Type:** duration
        
    - **Default:** `"5s"`
        
    - **Description:** Maximum time to wait for a response from the server.
        
    - **Example:**
        
    
    ```toml
    timeout = "10s"
    ```
    
    > Prevents Telegraf from hanging if the endpoint is slow.
    
    * * *
    
    ### 6\. **`follow_redirects`**
    
    - **Type:** boolean
        
    - **Default:** `true`
        
    - **Description:** Whether HTTP redirects (3xx responses) should be automatically followed.
        
    - **Example:**
        
    
    ```toml
    follow_redirects = false
    ```
    
    * * *
    
    ### 7\. **`name_override`**
    
    - **Type:** string
        
    - **Default:** `""`
        
    - **Description:** Override the measurement name for the metrics collected from this endpoint.
        
    - **Example:**
        
    
    ```toml
    name_override = "custom_metrics"
    ```
    
    > Useful when the endpoint’s default name is too generic.
    
    * * *
    
    ### 8\. **`data_format`**
    
    - **Type:** string
        
    - **Default:** `"json"`
        
    - **Description:** How Telegraf interprets the response. Options:
        
        - `"json"` → parse JSON
            
        - `"value"` → simple numeric value
            
        - `"graphite"` → Graphite format
            
        - `"influx"` → Influx line protocol
            
        - `"nagios"` → Nagios plugin output
            
        - `"prometheus"` → Prometheus metrics format
            
    - **Example:**
        
    
    ```toml
    data_format = "prometheus"
    ```
    
    * * *
    
    ### 9\. **`json_name_key`**
    
    - **Type:** string
        
    - **Default:** `""`
        
    - **Description:** For JSON responses, specify a key to use as the measurement name.
        
    - **Example:**
        
    
    ```toml
    json_name_key = "system.cpu"
    ```
    
    * * *
    
    ### 10\. **`json_query`**
    
    - **Type:** string
        
    - **Default:** `""`
        
    - **Description:** Extract specific nested JSON values using JMESPath syntax.
        
    - **Example:**
        
    
    ```toml
    json_query = "system.memory"
    ```
    
    > Only metrics under `system.memory` will be collected.
    
    * * *
    
    ### 11\. **`response_timeout`**
    
    - **Type:** duration
        
    - **Default:** `"5s"`
        
    - **Description:** Maximum time to wait for a response after connection is established.
        
    - **Example:**
        
    
    ```toml
    response_timeout = "10s"
    ```
    
    * * *
    
    ### 12\. **`tags`**
    
    - **Type:** table (key-value)
        
    - **Default:** `{}`
        
    - **Description:** Add static tags to all metrics collected from this HTTP endpoint.
        
    - **Example:**
        
    
    ```toml
    tags = { env = "prod", region = "us-east-1" }
    ```
    
    * * *
    
    ✅ **Summary:**  
    `[[inputs.http]]` is flexible and lets you fetch metrics via HTTP. Key options let you:
    
    1.  Specify which **URLs** to scrape (`urls`)
        
    2.  Choose **HTTP method** (`method`) and add **headers/body**
        
    3.  Control **timeouts** and redirects
        
    4.  Configure how the **data is parsed** (`data_format`, `json_query`)
        
    5.  Add **custom measurement names** (`name_override`) and **tags**
        
    
    * * *
    
    &nbsp;
    
    &nbsp;