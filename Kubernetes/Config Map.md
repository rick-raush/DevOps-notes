A **ConfigMap** is a Kubernetes object used to store **non-confidential configuration data** in key-value pairs. It allows you to **decouple configuration artifacts** from application code, making it easier to manage and update configurations without rebuilding your container images.

* * *

### ✅ What is ConfigMap Used For?

- Storing application configuration settings
    
- Keeping environment-specific values
    
- Injecting configuration into Pods at runtime
    
- Sharing the same config among multiple Pods
    

* * *

### 🔧 Example Use Cases:

1.  Set environment variables in a Pod
    
2.  Mount configuration files into containers
    
3.  Command-line arguments for containers
    

* * *

### 🧱 Basic Structure of a ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  APP_ENV: production
  APP_DEBUG: "false"
  DATABASE_URL: postgres://user:pass@db:5432/mydb
```

* * *

# 📦 Ways to Create a ConfigMap:

**4 ways to create ConfigMaps in Kubernetes**:

1.  **From a file**:
    
    ```bash
    kubectl create configmap my-config --from-file=config.txt
    ```
    
2.  **From a directory**:
    
    ```bash
    kubectl create configmap my-config --from-file=./config-dir
    ```
    
3.  **From literal values**:
    
    ```bash
    kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2
    ```
    
4.  **From env file:**
    
    ```bash
    kubectl create configmap my-config --from-env-file=app.env
    ```
    

* * *

### 1\. **From Literal Values**

- You specify key-value pairs directly on the command line.
    
- Useful for small, simple configurations.
    

**Example:**

```bash
kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2
```

Creates a ConfigMap with:

```yaml
data:
  key1: value1
  key2: value2
  
# kubectl get = quick overview
# kubectl describe = deep dive
```

* * *

### 2\. **From an Environment File**

- Use a file containing `KEY=VALUE` pairs, like a `.env` file.
    
- The ConfigMap reads each line and converts them to key-value pairs.
    

**Example file `app.env`:**

```
APP_ENV=production
APP_DEBUG=false
```

**Create ConfigMap:**

```bash
kubectl create configmap my-config --from-env-file=app.env
```

* * *

### 3\. **From a File**

- You provide a single file as the source.
    
- The filename becomes the key, and the file’s content becomes the value.
    

**Example:**

```bash
kubectl create configmap my-config --from-file=config.txt
```

If `config.txt` contains:

```
database_url=postgres://user:pass@db:5432/mydb
```

The ConfigMap will have:

```yaml
data:
  config.txt: |
    database_url=postgres://user:pass@db:5432/mydb
```

* * *

### 4\. **From a Directory**

- You provide a directory containing multiple files.
    
- Each filename becomes a key, and the content of each file becomes the value for that key.
    

**Example:**

If the directory `config-dir/` contains:

- `app.properties`
    
- `logging.conf`
    

Run:

```bash
kubectl create configmap my-config --from-file=config-dir/
```

ConfigMap data will include keys `app.properties` and `logging.conf` with corresponding file contents.

* * *

### Summary Table

| Method | Input Type | Key in ConfigMap | Value in ConfigMap | Use Case |
| --- | --- | --- | --- | --- |
| From literal | CLI key=value pairs | Keys as specified | Values as specified | Quick, small configs |
| From env file | `.env` file | Keys from file keys | Values from file values | Environment variables config |
| From single file | One file | Filename | File content | Single config file |
| From directory | Directory of files | Each filename | Each file content | Multiple config files |

&nbsp;

# **Create a ConfigMap from a YAML manifest**

You can **create a ConfigMap from a YAML manifest** in Kubernetes! In fact, using a YAML file is the **most flexible and declarative way** to define a ConfigMap, especially for GitOps or Infrastructure-as-Code workflows.

* * *

## 📄 Example: ConfigMap from YAML

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
  namespace: default
data:
  APP_NAME: MyAwesomeApp
  APP_ENV: production
  APP_DEBUG: "false"
  SERVER_PORT: "8080"
```

- `kind: ConfigMap` – Tells Kubernetes this is a ConfigMap.
    
- `data:` – Contains the key-value pairs (all values must be **strings**).
    

* * *

### 🛠 How to Apply It:

1.  Save it as a file (e.g., `configmap.yaml`)
    
2.  Apply with `kubectl`:
    

```bash
kubectl apply -f configmap.yaml
```

You can also create it with `kubectl create -f` (but `apply` is better for updates).

* * *

## 🔁 Equivalent to:

```bash
kubectl create configmap my-config \
  --from-literal=APP_NAME=MyAwesomeApp \
  --from-literal=APP_ENV=production \
  --from-literal=APP_DEBUG=false \
  --from-literal=SERVER_PORT=8080
```

But using YAML:

- Is easier to manage at scale
    
- Can be version-controlled
    
- Supports annotations, labels, and structured metadata
    

* * *

## 🧠 Pro Tip:

If you want to load the content of a file (like `app.properties`) into a ConfigMap via YAML:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  app.properties: |
    APP_NAME=MyAwesomeApp
    APP_ENV=production
    APP_DEBUG=false
```

This stores the entire file under the key `app.properties`.

* * *

### ✅ TL;DR:

Yes, you can — and **should** — create ConfigMaps using YAML when possible. It's the cleanest, most maintainable way in production environments.

* * *

## Kubernetes ConfigMap Creation Using `kubectl` with Dry-Run and YAML:

1.  **Get existing ConfigMap `cm1` in YAML:**

```bash
kubectl get cm cm1 -o yaml
```

2.  **Generate YAML manifest for new ConfigMap `cm6` (dry-run, client-side):**

```bash
kubectl create cm cm6 --from-literal=key=value --dry-run=client -o yaml > cm.yml
```

3.  **Preview generated YAML manifest for `cm6`:**

```bash
kubectl create cm cm6 --from-literal=key=value --dry-run=client -o yaml
```

4.  **Apply the new ConfigMap from YAML to the `raushan` namespace:**

```bash
kubectl apply -f cm.yml -n raushan
```

&nbsp;

## 🧾 Your Scenario:

- You **created a ConfigMap** named `cm6` (using dry-run + `apply`).
    
- Then you **created another ConfigMap** (or edited the YAML) and **applied it again** — also with name `cm6`.
    

* * *

## ✅ What Happens:

- **Kubernetes updates the existing ConfigMap** named `cm6`.
    
- It **does NOT create a second ConfigMap** with the same name.
    
- ConfigMap names must be **unique within a namespace**.
    

* * *

&nbsp;

&nbsp;

&nbsp;

* * *

&nbsp;

# 🚀 Using ConfigMap in a Pod-(Inject data from a ConfigMap into a Pod):

## 1\. **As environment variables**:

&nbsp;

### ✅ **Inject individual keys using `env` and `valueFrom.configMapKeyRef`**

This is used when you want to expose **specific keys** from a ConfigMap as **environment variables** in a container.

### Example:

```yaml
env:
  - name: APP_NAME
    valueFrom:
      configMapKeyRef:
        name: cm5
        key: APP_NAME
```

📌 This tells Kubernetes:

> "From ConfigMap `cm5`, take the value of key `APP_NAME` and inject it as the environment variable `APP_NAME`."

* * *

### ✅ **Inject all keys using `envFrom`**

This is a **shorthand** way to load **all key-value pairs** from a ConfigMap into the Pod as environment variables.

### Example:

```yaml
envFrom:
  - configMapRef:
      name: cm5
```

📌 This tells Kubernetes:

> "Take all keys in ConfigMap `cm5` and create environment variables for each."

So, if `cm5` has:

```yaml
data:
  APP_NAME: MyApp
  APP_ENV: production
```

The container will get:

```bash
APP_NAME=MyApp
APP_ENV=production
```

* * *

## 🆚 Summary: `env` vs `envFrom`

| Feature | `env` with `valueFrom` | `envFrom` |
| --- | --- | --- |
| Inject specific key | ✅ Yes | ❌ No (injects all keys) |
| Inject all keys | ❌ No | ✅ Yes |
| Rename env var | ✅ Yes (`name:` can differ from key) | ❌ No (env var name = key name) |
| Fine-grained control | ✅ Yes | ❌ No |

* * *

## ✅ Pro Tip:

You can **use both `envFrom` and `env` together** in the same container — useful if you want to inject most of the keys but override or rename one or two.

* * *

&nbsp;

## 2\. **As files in a volume**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-cm-file
  labels:
    class: app
spec:
  containers:
    - name: nginx
      image: nginxdemos/hello:plain-text
      volumeMounts:
        - name: configmap-volume
          mountPath: /etc/config
          readOnly: true

  volumes:
    - name: configmap-volume
      configMap:
        name: cm5 #when we need all the key: value pair in the config file
        
  volumes:
    - name: configmap-volume
      configMap:
        name: cm5
        items: 	#<-- when we need specific key: value pairs form the configMap
          - key: variable1
            path: key1
          - key: variable2
            path: key2


```

| Usage | What happens | Why use it? |
| --- | --- | --- |
| **Without `items`** | All keys become files with original names | When you want to mount all keys |
| **With `items`** | Only listed keys are mounted, can rename files | Selective mount + rename files |

* * *

### 🛑 Important Notes:

- ConfigMaps are **not designed to hold sensitive data** (use **Secrets** for that).
    
- Changes to a ConfigMap do **not automatically restart Pods**. You need to trigger a rollout or use a reloader mechanism.
    

&nbsp;

- **Updating a ConfigMap does NOT automatically restart Pods** because Kubernetes doesn’t assume your app should reload config changes on the fly.
    
- If the ConfigMap is used as **environment variables**, the new values take effect **only when the Pod restarts**.
    
- If the ConfigMap is **mounted as files**, the files update automatically, but your app must detect and reload them itself.
    
- To apply ConfigMap changes, you can:
    
    1.  **Manually restart Pods** (e.g., `kubectl rollout restart deployment`).
        
    2.  Use an **automated reloader tool** that watches ConfigMaps and triggers Pod restarts.
        
    3.  Build your app to **reload config dynamically** from mounted files without restarting.
        

| Goal | Best Approach |
| --- | --- |
| Simple updates with manual control | Use `kubectl rollout restart` |
| Automated updates in production | Use a reloader like **Stakater Reloader** |
| High performance, no restarts | Use apps that support dynamic config reloads |

&nbsp;

&nbsp;

&nbsp;