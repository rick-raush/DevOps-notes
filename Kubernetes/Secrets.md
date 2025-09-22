### 🔐 Definition: Secrets in Kubernetes (K8s)

A **Secret** in Kubernetes is an object used to **store and manage sensitive information**, such as passwords, tokens, SSH keys, or certificates, separately from application code.

* * *

### 📘 Official Definition:

> A **Secret** is a Kubernetes object that contains a small amount of sensitive data such as a password, a token, or a key. Such information might otherwise be put in a Pod specification or in a container image.

* * *

### ✅ Key Characteristics:

- Stored as **key-value pairs**
    
- Encoded in **Base64** (not encrypted by default)
    
- Designed to keep **sensitive data safe** from accidental exposure
    
- Can be:
    
    - **Mounted** as a file inside a Pod
        
    - Exposed as **environment variables**
        
    - Accessed programmatically via the **Kubernetes API**
        

* * *

### 📦 Basic Structure:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=     # base64 for "admin"
  password: czNjcjN0     # base64 for "s3cr3t"
```

* * *

### 🛡️ Security Notes:

- **Base64 is not encryption** — it's just encoding.
    
- Enable **encryption at rest** for real protection.
    
- Use **RBAC** to control who can access secrets.
    
- Consider external tools (e.g., **HashiCorp Vault**, **Sealed Secrets**) for higher security.
    

* * *

&nbsp;

&nbsp;

**Types of Secrets**, each intended for different use cases. Here's a breakdown of the **common types**:

* * *

## 🔑 Types of Kubernetes Secrets

| Secret Type | Description | Use Case |
| --- | --- | --- |
| `Opaque` | Default type. Arbitrary key-value pairs. | Storing passwords, tokens, API keys, etc. |
| `kubernetes.io/dockerconfigjson` | Stores Docker registry credentials in JSON format. | Pulling images from private Docker registries. |
| `kubernetes.io/tls` | Stores a TLS certificate and private key. | TLS for Ingress controllers, web servers, etc. |
| `kubernetes.io/service-account-token` | Automatically generated token for a service account. | Accessing the Kubernetes API from a Pod. |

* * *

## 📦 1. `Opaque` (Generic)

**Default type** when creating secrets without specifying a type.

```bash
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password='s3cr3t'
```

**YAML Example:**

```yaml
type: Opaque
data:
  username: YWRtaW4=
  password: czNjcjN0
```

* * *

## 🐳 2. `kubernetes.io/dockerconfigjson`

Used for Docker registry authentication.

```bash
kubectl create secret docker-registry regcred \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-server=<registry-url> \
  --docker-email=<email>
```

**YAML Example:**

```yaml
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-JSON>
```

Used in imagePullSecrets:

```yaml
imagePullSecrets:
  - name: regcred
```

* * *

## 🔐 3. `kubernetes.io/tls`

Used for storing **TLS certificates**.

```bash
kubectl create secret tls tls-secret \
  --cert=path/to/tls.crt \
  --key=path/to/tls.key
```

**YAML Example:**

```yaml
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded cert>
  tls.key: <base64-encoded key>
```

Used for TLS in Ingress or web services.

* * *

## 🧾 4. `kubernetes.io/service-account-token`

Automatically created by Kubernetes for **service accounts** to access the API securely from inside Pods.

You usually don’t create these manually — Kubernetes handles it.

**YAML Example:**

```yaml
type: kubernetes.io/service-account-token
metadata:
  name: my-token
  annotations:
    kubernetes.io/service-account.name: "my-service-account"
```

* * *

### 💡 Summary Table

| Type | Key Field(s) | Purpose |
| --- | --- | --- |
| `Opaque` | Custom key-value pairs | General-purpose secrets |
| `kubernetes.io/dockerconfigjson` | `.dockerconfigjson` | Docker registry credentials |
| `kubernetes.io/tls` | `tls.crt`, `tls.key` | TLS certs for HTTPS |
| `kubernetes.io/service-account-token` | `token`, `ca.crt`, `namespace` | API access from inside a Pod |

* * *

&nbsp;

&nbsp;

We can create `Secrets` using different methods: from **literal values**, **files**, **environment files**, or directly from a **YAML manifest**. Here's how to do each of them:

* * *

## 🔐 1. **Create Secret from Literal**

You can create a secret by passing key-value pairs directly in the command:

```bash
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123
```

✅ Result: A secret named `my-secret` with keys `username` and `password`.

* * *

## 📄 2. **Create Secret from File**

You can use files where the content becomes the secret value, and the filename becomes the key.

```bash
kubectl create secret generic my-secret \
  --from-file=./username.txt \
  --from-file=./password.txt
```

✅ Result: Each file becomes a key, and its content becomes the corresponding value.

> If you want to specify a custom key name:

```bash
kubectl create secret generic my-secret \
  --from-file=my-user=./username.txt
```

* * *

## 🌍 3. **Create Secret from Env File**

You can use an env-style `.env` file, like:

```env
DB_USER=admin
DB_PASS=secret
```

Then run:

```bash
kubectl create secret generic my-secret --from-env-file=./app.env
```

✅ Result: Each line in the env file becomes a key-value pair in the secret.

* * *

## 📜 4. **Create Secret from YAML Manifest**

You can define the secret explicitly in a YAML file. Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=         # base64 encoded 'admin'
  password: c2VjcmV0MTIz     # base64 encoded 'secret123'
```

To apply it:

```bash
kubectl apply -f my-secret.yaml
```

> 🔐 You must **base64 encode** the values manually:

```bash
echo -n 'admin' | base64      # YWRtaW4=
echo -n 'secret123' | base64  # c2VjcmV0MTIz
```

* * *

## 🧠 Quick Tips

- Use `kubectl describe secret my-secret` to inspect metadata (but not secret values).
    
- Use `kubectl get secret my-secret -o yaml` to see encoded values.
    
- Decode with `echo <base64_string> | base64 --decode`.
    

* * *

&nbsp;

## Follow Ups-

* * *

## 📌 **1\. Why do we need `-n` with `echo` when base64 encoding?**

### 🔍 The issue:

If you use `echo` **without** `-n`, it adds a **newline (`\n`)** at the end of the string by default.

### 🔧 Example:

```bash
echo "admin" | base64
# Output: YWRtaW4K
```

> `YWRtaW4K` is **admin + newline**

Now with `-n`:

```bash
echo -n "admin" | base64
# Output: YWRtaW4=
```

✅ `YWRtaW4=` is the correct base64 for just `"admin"` (no newline).

### 🧠 Why does this matter?

When creating secrets in Kubernetes, the values should be exactly what you intend. A newline (`\n`) is a valid character, so if you leave it in, you might get unexpected behavior (e.g., wrong passwords or connection errors).

* * *

## 🔐 **2\. Base64 Encoding vs. Encryption**

| Feature | **Base64 Encoding** | **Encryption** |
| --- | --- | --- |
| **Purpose** | Data transformation for storage or transfer | Data protection and confidentiality |
| **Readable?** | ✅ Easily reversible (not secure) | ❌ Not easily readable without a key |
| **Reversible?** | ✅ Yes (using `base64 --decode`) | ✅ Only with the correct decryption key |
| **Security?** | ❌ None | ✅ Yes, if a strong algorithm and key are used |
| **Use in Secrets?** | ✅ Required for Kubernetes YAML | ✅ Used **behind the scenes** for secret storage |

### 🔍 So why does Kubernetes use **base64**?

- Kubernetes expects **base64-encoded values** in YAML so that they can be safely serialized and transported (e.g., no weird characters or line breaks).
    
- But Kubernetes **doesn't treat base64 as secure**. That's why it's still up to your cluster setup (like encrypted etcd, RBAC, etc.) to **secure** the actual secret data.
    

* * *

## 🔒 Summary

- Use `echo -n` to avoid unintended newlines in base64 encoding.
    
- **Base64 is not encryption.** It's just encoding for safe representation.
    
- Kubernetes uses base64 for technical reasons, not for security. Protect secrets via:
    
    - Encrypted etcd
        
    - RBAC policies
        
    - Network policies
        
    - Secret management tools like Vault or Sealed Secrets
        

* * *

&nbsp;

&nbsp;

Using `kubectl get ... -o yaml` and `kubectl create ... --dry-run=client -o yaml` are **two powerful methods** to generate YAML for Kubernetes resources, including **Secrets** and many others.

Let’s break them down with clarity:

* * *

## ✅ 1. `kubectl get <resource> -o yaml`

> **Purpose:** Export the existing resource’s full YAML definition (as applied in the cluster).

### 📌 Use Case:

You already created a secret (or any resource) and want to **see or reuse its YAML**.

### Example:

```bash
kubectl get secret sfromenvfile -o yaml
```

🔍 This shows the actual YAML **as stored in Kubernetes**, including:

- Metadata
    
- Type
    
- Base64-encoded data
    

> You can redirect it to a file:

```bash
kubectl get secret sfromenvfile -o yaml > my-secret.yaml
```

* * *

## ✅ 2. `kubectl create <resource> ... --dry-run=client -o yaml`

> **Purpose:** Generate a **YAML definition without applying it** to the cluster — a perfect way to get sample YAML templates.

### 📌 Use Case:

You want to write YAML files for source control, but don’t want to apply them just yet.

### Example:

```bash
kubectl create secret generic ssample \
  --from-literal=key=value \
  --dry-run=client -o yaml
```

🔍 This outputs:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ssample
type: Opaque
data:
  key: dmFsdWU=   # base64 for 'value'
```

> Again, you can redirect this to a file:

```bash
kubectl create secret generic ssample \
  --from-literal=key=value \
  --dry-run=client -o yaml > ssample-secret.yaml
```

* * *

## 🧩 Works for Other Resources Too!

You can use the same pattern for:

- ✅ ConfigMaps
    
- ✅ Deployments
    
- ✅ Services
    
- ✅ Ingress
    
- ✅ CronJobs
    
- ✅ PersistentVolumeClaims
    
- ... and many more
    

### Example: Generate a Deployment YAML

```bash
kubectl create deployment myapp --image=nginx --dry-run=client -o yaml
```

* * *

## ✅ Summary

| Method | Purpose |
| --- | --- |
| `kubectl get <resource> -o yaml` | Export current live resource |
| `kubectl create ... --dry-run -o yaml` | Generate YAML before applying |

* * *

&nbsp;

&nbsp;

# Inject Secret in a Pod

* * *

## ✅ 1.Inject Secret as Env Var in a Pod

Let’s say you have a Secret like this:

### 🔐 Example Secret: `my-secret`

```bash
kubectl create secret generic my-secret \
  --from-literal=DB_USER=admin \
  --from-literal=DB_PASS=secret123
```

This creates a Secret with two keys: `DB_USER` and `DB_PASS`.

* * *

### 📦 Now: Define a Pod that uses those secrets as env vars

Here’s the YAML manifest to inject them into a Pod:

```yaml
#can use "kubectl explain pods/secrets --recursive" to get the format
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-demo
spec:
  containers:
  - name: my-container
    image: busybox
    command: [ "sh", "-c", "env; sleep 3600" ]
    env:
      - name: DB_USERNAME
        valueFrom:
          secretKeyRef:
            name: my-secret
            key: DB_USER
      - name: DB_PASSWORD
        valueFrom:
          secretKeyRef:
            name: my-secret
            key: DB_PASS
```

* * *

## 🔍 What’s Happening Here?

- The pod will set two environment variables:
    
    - `DB_USERNAME` ← gets value from `my-secret` → `DB_USER`
        
    - `DB_PASSWORD` ← gets value from `my-secret` → `DB_PASS`
        

* * *

### 🚀 Apply the Pod:

```bash
kubectl apply -f pod-secret-env.yaml
```

Then check the environment inside the Pod:

```bash
kubectl exec -it secret-env-demo -- sh
# Inside the pod:
env | grep DB_
```

Expected output:

```
DB_USERNAME=admin
DB_PASSWORD=secret123
#Here the decoded values are seen in the pod
```

* * *

## 🧠 Alternative: Inject All Secret Keys as Env Vars (Bulk)

You can inject **all keys from a Secret** like this:

```yaml
    envFrom:
      - secretRef:
          name: my-secret
```

> This will automatically expose `DB_USER` and `DB_PASS` as environment variables **with those exact names**.

* * *

## ✅ Summary

| Method | Use Case |
| --- | --- |
| `env` + `secretKeyRef` | Map specific secret keys to custom env vars; can rename the keys |
| `envFrom` + `secretRef` | Bulk expose all keys as env vars |

* * *

&nbsp;

&nbsp;

* * *

## ✅ 2.**Inject a Secret into a Pod as Files using a Volume**

This method mounts the Secret as files inside the container, where:

- Each **key** becomes a **filename**
    
- Each **value** becomes the **file content**
    

* * *

### 🔐 Example: Create the Secret

```bash
kubectl create secret generic my-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123
```

* * *

### 📦 Example Pod YAML: Mount Secret as Files

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-demo
spec:
  containers:
  - name: demo-container
    image: busybox
    command: ["sh", "-c", "cat /etc/creds/username && cat /etc/creds/password && sleep 3600"]
    volumeMounts:
    - name: secret-volume
      mountPath: "/etc/creds"
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: my-secret
```

* * *

## 🔍 Breakdown

- **Volume Name**: `secret-volume`
    
- **Mount Path**: `/etc/creds` inside the container
    
- The files created:
    
    - `/etc/creds/username` → contains `admin`
        
    - `/etc/creds/password` → contains `secret123`
        

* * *

### 🚀 Deploy the Pod

```bash
kubectl apply -f secret-volume-pod.yaml
```

Then check the files:

```bash
kubectl exec -it secret-volume-demo -- cat /etc/creds/username
# Output: admin

kubectl exec -it secret-volume-demo -- cat /etc/creds/password
# Output: secret123
```

* * *

## 📁 Optional: Change File Names or Paths

You can customize file names using `items`:

```yaml
  volumes:
  - name: secret-volume
    secret:
      secretName: my-secret
      items:
        - key: username
          path: user.txt
        - key: password
          path: pass.txt
```

Now the files would be:

- `/etc/creds/user.txt`
    
- `/etc/creds/pass.txt`
    

* * *

## ✅ Summary: Secret as Files in a Pod

| Key | Benefit |
| --- | --- |
| Secure way to provide credentials | ✅   |
| Avoids hardcoding in YAML | ✅   |
| Automatic file creation per key | ✅   |
| Supports read-only volumes | ✅   |

* * *

Let me know if you want an example using a **Deployment**, or want to inject both **env vars and files** together.

&nbsp;