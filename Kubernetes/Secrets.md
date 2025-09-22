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