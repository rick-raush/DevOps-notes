&nbsp;

## 🧭 What is `nodeSelector`?

`nodeSelector` is the **simplest way** to tell Kubernetes to **schedule a Pod on specific nodes** by **matching labels**.

> It’s a **Pod-level setting** that selects nodes **based on key-value labels**.

* * *

## 🛠️ How It Works

1.  You add **labels** to your nodes:
    
    ```bash
    kubectl label nodes <node-name> disktype=ssd
    ```
    
2.  Then, in your Pod spec, you specify a `nodeSelector` that matches the node label:
    
    ```yaml
    spec:
      nodeSelector:
        disktype: ssd
    ```
    
3.  Kubernetes will only **schedule the Pod on nodes** that have that label.
    

* * *

## ✅ Example

### 🧷 Label a node:

```bash
kubectl label nodes node-1 disktype=ssd
```

### 📦 Pod YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: nginx
      image: nginx
  nodeSelector:
    disktype: ssd
```

> The Pod will only be scheduled on nodes that have `disktype=ssd`.

* * *

## ⚠️ Important Notes

- `nodeSelector` is **match all** (AND logic) — all key-value pairs must match.
    
- If no matching node is found, the Pod stays **Pending**.
    
- It's **static** — no complex expressions (unlike node affinity).
    

* * *

## 🆚 `nodeSelector` vs `nodeAffinity`

| Feature | `nodeSelector` | `nodeAffinity` |
| --- | --- | --- |
| Syntax | Simple key-value | More expressive (operators, weights) |
| Logic | Exact match (AND) | Supports OR, IN, NOT_IN, etc. |
| Use case | Simple targeting | Complex scheduling rules |
| Flexibility | Low | High |

* * *

## 🔍 When to Use `nodeSelector`?

Use it when:

- You have **simple node targeting needs**
    
- You want to **quickly pin pods to certain nodes**
    
- You don’t need conditional or weighted rules
    

* * *

&nbsp;

&nbsp;