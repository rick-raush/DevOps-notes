**Taints and Tolerations** 

They help enforce **workload isolation, scheduling rules**, and **dedicated node usage** (e.g., for GPUs, system workloads, etc.).

* * *

## 🚫 **What is a Taint?**

A **taint** is applied to a **Node**, and it says:

> "Don’t schedule any Pods on me **unless** they explicitly **tolerate** this taint."

It's a way for nodes to **repel** Pods unless the Pod is specially configured.

### 📌 Example: Add a taint to a node

```bash
kubectl taint nodes node1 key=value:NoSchedule
```

This adds a taint:

```
key=value:NoSchedule
#key operator value effect
```

This means:

- Pods **will not** be scheduled on `node1` **unless they have a matching toleration**.

* * *

## ✅ **What is a Toleration?**

A **toleration** is added to a **Pod**, and it says:

> "I am okay (I tolerate) being scheduled on a node with this specific taint."

So, tolerations **don't force** a Pod onto a tainted node — they just **allow** it to be scheduled there if other conditions match.

* * *

## 🔁 How They Work Together

| Node Has Taint | Pod Has Matching Toleration | Result |
| --- | --- | --- |
| Yes | Yes | ✅ Pod can run there |
| Yes | No  | ❌ Pod will not run there |
| No  | Doesn't matter | ✅ Pod can run there |

* * *

## 🔧 Example: Taint + Toleration

### ➤ Taint a Node:

```bash
kubectl taint nodes node1 env=prod:NoSchedule
```

Now, only Pods with this toleration can run on `node1`.

### ➤ Pod with Matching Toleration:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tolerant-pod
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "env"
    operator: "Equal"
    value: "prod"
    effect: "NoSchedule"
```

This Pod says: “I tolerate `env=prod:NoSchedule`” → so it **can** be scheduled on `node1`.

* * *

## 🧠 Taint Effects

| Effect | Description |
| --- | --- |
| `NoSchedule` | Don't schedule Pods unless they have a matching toleration |
| `PreferNoSchedule` | Try to avoid scheduling Pods here unless no better option |
| `NoExecute` | Evict existing Pods unless they tolerate the taint |

* * *

## 🔍 View Taints on a Node

```bash
kubectl describe node <node-name>
```

Look under the **Taints** section.

* * *

## 🚀 Common Use Cases

- **Dedicated nodes** for production workloads
    
- **GPU nodes** only for ML jobs
    
- **Critical system Pods** on control-plane nodes
    
- **Isolating tenant workloads** (multi-tenant clusters)
    

* * *

Let me know if you'd like a full example with a Deployment, or how to remove taints!