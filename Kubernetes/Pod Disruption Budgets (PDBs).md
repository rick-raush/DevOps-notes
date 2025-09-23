**Pod Disruption Budgets (PDBs)** in Kubernetes — what they are, their sub-fields, and YAML examples to help you understand exactly how to use them.

* * *

## ✅ What is a Pod Disruption Budget (PDB)?

A **Pod Disruption Budget** is a Kubernetes resource that ensures **a minimum number of pods remain available** during voluntary disruptions.

> It **does not prevent** involuntary disruptions (like node crashes), but **does protect** against **voluntary disruptions** like:

- Node draining (`kubectl drain`)
    
- Cluster autoscaler scale-down
    
- Rolling updates
    

* * *

## 🧩 Core Components (Subfields) of a PDB

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: <pdb-name>
spec:
  minAvailable: <int or string>       # OR
  maxUnavailable: <int or string>     # one of these two is required
  selector:                           # required
    matchLabels:
      <label-key>: <label-value>
```

### 🔹 `minAvailable` (Optional, but one of `minAvailable` or `maxUnavailable` is required)

- Specifies the **minimum number** or **percentage** of pods that **must remain available** during a disruption.
    
- Can be an absolute number (`2`) or a percentage (`50%`).
    

### 🔹 `maxUnavailable` (Optional)

- Specifies the **maximum number** or **percentage** of pods that **can be disrupted** at a time.
    
- Can also be a number or percentage.
    
- You can use either `minAvailable` **or** `maxUnavailable`, but **not both**.
    

### 🔹 `selector`

- Selects **which pods** this PDB applies to using labels.
    
- Usually matches the label of your **Deployment**, **ReplicaSet**, **StatefulSet**, etc.
    

* * *

## 🔧 YAML Examples

* * *

### ✅ Example 1: PDB with `minAvailable`

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: web
```

➡️ At least **2 pods must always be available**. If fewer than 2 pods are running, voluntary disruptions will be **blocked**.

* * *

### ✅ Example 2: PDB with `maxUnavailable`

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: api-pdb
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
      app: api
```

➡️ At most **1 pod can be disrupted** at a time. The rest must remain running.

* * *

### ✅ Example 3: PDB using Percentage

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: backend-pdb
spec:
  minAvailable: 60%
  selector:
    matchLabels:
      app: backend
```

➡️ At least **60% of the pods** with label `app=backend` must remain available during a disruption.

* * *

## 🧠 Important Notes

| Rule | Explanation |
| --- | --- |
| You must use **either `minAvailable` or `maxUnavailable`**, not both. |     |
| PDBs work only with **controllers** (Deployments, StatefulSets, etc.) — **not standalone pods**. |     |
| `minAvailable` > number of replicas? → No disruptions allowed. |     |
| PDBs only protect against **voluntary** disruptions (e.g., `kubectl drain`). |     |

* * *

## 👀 Check PDB Status

To see how your PDB is behaving:

```bash
kubectl get pdb
kubectl describe pdb <pdb-name>
```

You'll see:

- Allowed disruptions
    
- Current healthy pods
    
- Number of disruptions that are currently allowed
    

* * *

## ✅ Example Use Case in a Deployment

If you have a Deployment with 3 replicas:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx
```

Then your PDB might be:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: nginx-pdb
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
      app: nginx
```

➡️ This ensures that **at least 2 out of 3** pods remain available during a rolling update or node drain or scale down.

* * *

&nbsp;

| Resource | Field(s) | Purpose |
| --- | --- | --- |
| PodDisruptionBudget | `minAvailable` or `maxUnavailable` | Control pod availability during voluntary disruptions |
| Deployment (RollingUpdate) | `maxUnavailable`, `maxSurge` | Control how many pods can be updated/unavailable simultaneously |
| StatefulSet (RollingUpdate) | `partition` (for update control) | Control update ordering (not exactly `maxUnavailable`) |
| DaemonSet (RollingUpdate) | `maxUnavailable` | Control rolling update disruption |

&nbsp;

## **PDBs** vs **Deployment update strategies**

| Feature | When it applies | What it controls |
| --- | --- | --- |
| Deployment maxUnavailable/ maxSurge | Only during rolling updates | How many pods are unavailable or added during update |
| Pod Disruption Budget | Any voluntary disruption | Minimum number of pods that must be available overall |