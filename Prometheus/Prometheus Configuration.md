```
global:
  scrape_interval: 15s
  scrape_timeout: 10s
  scrape_protocols:
  - OpenMetricsText1.0.0
  - OpenMetricsText0.0.1
  - PrometheusText0.0.4
  evaluation_interval: 15s
  external_labels:
    prometheus_replica: 10.4.51.111
alerting:
  alert_relabel_configs:
  - separator: ;
    regex: prometheus_replica
    replacement: $1
    action: labeldrop
  alertmanagers:
  - follow_redirects: true
    enable_http2: true
    scheme: http
    timeout: 10s
    api_version: v2
    static_configs:
    - targets:
      - 10.4.51.111:9093
rule_files:
- /etc/prometheus/rules/*.rules.yml
- /etc/prometheus/rules/*.yaml
```

&nbsp;

&nbsp;

# Prometheus scrape configs

👌 Let’s break down the **key components** of a Prometheus relabeling rule so that the whole picture is clear.

# Difference: `relabel_configs` vs `metric_relabel_configs`

- **`relabel_configs`**
    
    - Applied at **target discovery time**.
        
    - Operates on **target labels** (like `__address__`, `__meta_kubernetes_*`, tags from EC2, etc.).
        
    - Controls **what the target looks like to Prometheus** (renaming, adding labels, dropping targets, etc.).
        
- **`metric_relabel_configs`**
    
    - Applied **after a scrape**, on the **metrics inside the target**.
        
    - Operates on **metric labels and names**.
        
    - Useful for dropping unwanted metrics, renaming, or filtering heavy series before ingestion.
        

👉 Think of it like:

- `relabel_configs`: shaping/discovering the **target**
    
- `metric_relabel_configs`: shaping/cleaning the **metrics**
    

* * *

# 🔹 Anatomy of a Relabeling Rule

A relabeling block looks like this:

```yaml
- source_labels: [<list of labels>]
  separator: ;
  regex: (.*)
  target_label: <new_label>
  replacement: $1
  action: replace
```

* * *

## 1\. **`source_labels`**

- Which labels (from the target or metric) should be read.
    
- Their values are concatenated together (with `separator`) → then matched against `regex`.
    

👉 Example:

```yaml
source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_pod_name]
separator: ;
```

If namespace=`default` and pod_name=`nginx-1`, the combined string becomes:

```
"default;nginx-1"
```

* * *

## 2\. **`separator`**

- Used only if multiple `source_labels` are given.
    
- Defines how their values are joined before applying regex.
    
- Default = `;`.
    
- Rarely changed, but you can set to `,` or `|` etc.
    

* * *

## 3\. **`regex`**

- Regular expression applied on the combined source value.
    
- Determines whether the rule applies and what gets captured.
    

👉 Example:

```yaml
regex: (.*)-(.*)
```

On input `nginx-123`, this captures:

- `$1 = nginx`
    
- `$2 = 123`
    

* * *

## 4\. **`target_label`**

- The label to set (or modify).
    
- This is where the result of relabeling ends up.
    

👉 Example:

```yaml
target_label: instance
```

This will set (or overwrite) the `instance` label.

* * *

## 5\. **`replacement`**

- What to write into the `target_label`.
    
- Can use captured groups from regex (`$1`, `$2`, …).
    
- Can be a static string.
    

👉 Example:

```yaml
replacement: $1
```

If regex matched `(.*)-(.*)` and input was `nginx-123`, then `$1=nginx`, so result is `"nginx"`.

👉 Another:

```yaml
replacement: prod
```

Always sets `"prod"` regardless of input.

* * *

## 6\. **`action`**

Defines what to do. Common ones:

- `replace` → (most common) Replace or set `target_label` with `replacement`.
    
- `keep` → Keep the target **only if regex matches**. Otherwise drop.
    
- `drop` → Drop the target if regex matches.
    
- `labelmap` → Rename labels based on regex (useful for prefixing).
    
- `labeldrop` → Drop labels matching regex.
    
- `labelkeep` → Keep only labels matching regex.
    

👉 Example:

```yaml
action: drop
regex: 10\.0\.0\..*
source_labels: [__address__]
```

Drops any target with IP starting `10.0.0.*`.

* * *

# 🔹 Putting it all together

Example:

```yaml
- source_labels: [__address__]
  regex: (.*):\d+
  target_label: instance
  replacement: $1
  action: replace
```

- Take the target’s `__address__` (say `10.4.51.236:9273`).
    
- Regex `(.*):\d+` → captures `10.4.51.236`.
    
- Replacement `$1` → sets to `10.4.51.236`.
    
- Target label = `instance`.
    

✅ Now your `instance` label no longer has the port.

* * *

# ⚡ Quick analogy

- `source_labels`: "Where do I read from?"
    
- `separator`: "How do I glue values together?"
    
- `regex`: "What pattern am I looking for?"
    
- `replacement`: "What should I write back?"
    
- `target_label`: "Where should I put the result?"
    
- `action`: "What type of operation do I want (replace, drop, keep, …)?"
    

* * *

&nbsp;

Got it 👍 Let’s break down **Service Discovery (SD)** in Prometheus for **Kubernetes** and **EC2**.  
Both are about: *“How does Prometheus automatically find scrape targets, instead of me hardcoding IPs?”*

* * *

# 🔹 1. Kubernetes Service Discovery

Prometheus integrates tightly with Kubernetes.  
When you enable `kubernetes_sd_configs`, Prometheus queries the **Kubernetes API server** to discover resources dynamically.

Example:

```yaml
scrape_configs:
- job_name: 'kubernetes-service-endpoints'
  kubernetes_sd_configs:
  - role: endpoints
    kubeconfig_file: /data/kubeconfig
```

### 🔑 `role` options

- `node` → discovers all K8s nodes
    
    - Gives you labels like `__meta_kubernetes_node_name`.
- `pod` → discovers all pods
    
    - Useful when you want to scrape metrics directly from app pods.
- `service` → discovers services (cluster IPs).
    
- `endpoints` → discovers endpoints behind services (best for scraping exporters, apps).
    
- `ingress` → discovers ingresses.
    

### Labels you get

Prometheus attaches tons of labels from Kubernetes:

- `__meta_kubernetes_namespace`
    
- `__meta_kubernetes_pod_name`
    
- `__meta_kubernetes_service_name`
    
- `__meta_kubernetes_node_name`
    
- etc.
    

You then use **`relabel_configs`** to turn those into useful labels (`namespace`, `pod`, `service`, `node`, `environment=prod`, etc.).

👉 Example: scrape all pods with annotation `prometheus.io/scrape: "true"`

```yaml
relabel_configs:
- source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
  action: keep
  regex: true
```

✅ This way, Prometheus dynamically follows pods/services as they scale, die, or move.

* * *

# 🔹 2. EC2 Service Discovery

When running Prometheus in AWS, you can use **`ec2_sd_configs`**.  
Prometheus queries the **EC2 API** (via IAM credentials) and discovers instances based on filters.

Example:

```yaml
ec2_sd_configs:
  - endpoint: ""
    region: ap-south-1
    profile: arn:aws:iam::337891198417:role/ec2/cst-prometheus-prod-iam-role
    refresh_interval: 1m40s
    port: 9999
    filters:
    - name: instance-state-name
      values:
      - running
    - name: tag:environment
      values:
      - prod
    - name: tag:role
      values:
      - kafka
    follow_redirects: true
    enable_http2: true
```

### What happens:

- Prometheus asks EC2 API: *“Give me all running instances in ap-south-1 with tag Environment=prod.”*
    
- It automatically discovers their private/public IPs.
    
- Prometheus attaches EC2 metadata as labels:
    
    - `__meta_ec2_instance_id`
        
    - `__meta_ec2_private_ip`
        
    - `__meta_ec2_tag_Name` (any EC2 tag comes as a label)
        
    - `__meta_ec2_availability_zone`
        
- Then you relabel those into useful labels (`instance`, `environment`, `app`, etc.).
    

👉 Example: set environment label from EC2 tag

```yaml
relabel_configs:
    - source_labels: [__meta_ec2_tag_Environment]
      target_label: environment
```

✅ This way, new EC2s with tag `Environment=prod` are automatically monitored, no config change needed.

* * *

# 🔹 Comparison: Kubernetes vs EC2 SD

| Feature | Kubernetes SD | EC2 SD |
| --- | --- | --- |
| Source | Kubernetes API | AWS EC2 API |
| Resources discovered | Pods, Nodes, Services, Endpoints, Ingress | EC2 instances |
| Metadata available | namespace, pod name, node, labels, annotations | instance ID, tags, AZ, IP addresses |
| Typical usage | Dynamic pods & services monitoring | Node Exporters / App instances in AWS |
| Scaling | Perfect for autoscaling workloads in K8s | Perfect for autoscaling EC2 groups |

* * *

⚡ TL;DR:

- **K8s SD**: auto-discovers pods, services, nodes from the cluster → great for container workloads.
    
- **EC2 SD**: auto-discovers AWS EC2 instances with filters/tags → great for VM-based workloads.
    

* * *

&nbsp;

Ahh nice observation 👌 — you noticed that in most **EC2 service discovery** configs we just map tags directly, while in **Kubernetes SD** we often use regex + replacement. Let’s dig into why.

* * *

# 🔹 Why difference between EC2 SD and Kubernetes SD?

### 1\. **EC2 metadata is already clean and label-like**

- In EC2 SD, Prometheus automatically exposes each **tag** as `__meta_ec2_tag_<TagName>`.
    
- Example: EC2 tag `Environment=prod` → Prometheus label `__meta_ec2_tag_Environment="prod"`.
    
- You usually just copy this over directly:
    

```yaml
- source_labels: [__meta_ec2_tag_Environment]
  target_label: environment
```

👉 No need for regex or replacement, because you already have clean string values (`prod`, `staging`, etc.).

* * *

### 2\. **Kubernetes metadata is often messy or nested**

- K8s labels/annotations come in more complex forms, like:
    
    - `__meta_kubernetes_pod_annotation_prometheus_io_path=/metrics`
        
    - `__meta_kubernetes_namespace=default`
        
    - `__meta_kubernetes_pod_name=nginx-1234`
        
- Many times you only want *part* of the value.
    
    - Example: Pod name `nginx-1234` → you only want `nginx`.
        
    - Here you’d need regex:
        

```yaml
- source_labels: [__meta_kubernetes_pod_name]
  regex: (.*)-[0-9]+
  target_label: app
  replacement: $1
```

✅ Now `nginx-1234` → `app=nginx`.

* * *

### 3\. **EC2 values are static, K8s values are dynamic**

- EC2 tags are human-assigned (`Environment=prod`, `Role=app`).
    
- Kubernetes generates a lot of system values (`pod_name`, `service_name`) where regex helps you trim or normalize them.
    

* * *

### 4\. **Regex in EC2 relabeling is rare but possible**

Even in EC2 SD, you *can* use regex when needed. Example:  
Suppose your EC2 instances are tagged like `Environment=prod-us-east-1`, and you only want the first part (`prod`).

```yaml
- source_labels: [__meta_ec2_tag_Environment]
  regex: (.*?)-.*
  target_label: environment
  replacement: $1
```

✅ If tag is `prod-us-east-1`, result → `environment=prod`.

So regex is not forbidden in EC2 — it’s just **less often required** because tags are usually already clean.

* * *

# ⚡ TL;DR

- **EC2 SD**: Tags are already structured key-value pairs → usually just `source_labels` + `target_label`.
    
- **K8s SD**: Metadata (names, annotations) often needs trimming, parsing, or normalizing → regex + replacement is common.
    
- Regex can be used in **both**, but it’s more *necessary* in K8s.
    

* * *

&nbsp;