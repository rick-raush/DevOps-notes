Node Affinity dictates which nodes a pod can run on, based on node labels, while Pod Anti-Affinity prevents pods from being scheduled on the same node as other specific pods, based on pod labels. Node Affinity is used to target specific nodes for hardware or data residency needs, whereas Pod Anti-Affinity is used for high availability by spreading pods across nodes or preventing performance conflicts.

Node Affinity

- What it is: A rule to ensure a pod is scheduled on a node that meets certain label criteria.
- How it works: You label your nodes and then use nodeAffinity in the pod's spec to tell the Kubernetes scheduler to place pods on nodes with matching labels.
- Use Cases:
    - Hardware-specific workloads: Placing a pod on a node with a specific GPU or other hardware.
    - Geographic distribution: Ensuring pods run in a specific region for data residency or latency optimisation.
    - Data residency: Meeting compliance requirements by keeping pods within certain geographical boundaries.

Pod Anti-Affinity

- What it is: A rule that prevents a new pod from being scheduled on the same node as other pods that match a specific label.
- How it works: You define a rule in the podAntiAffinity section of the pod's spec that specifies labels of existing pods. The scheduler will then avoid placing the new pod on a node where pods with those matching labels are already running.
- Use Cases:
    - High availability: Preventing multiple instances of a critical application (like a database replica) from running on the same node to avoid a single point of failure.
    - Performance isolation: Avoiding placing pods that might interfere with each other's performance on the same node.
    - Workload balancing: Distributing pods evenly across a cluster to balance load.

&nbsp;

**Node Affinity vs. Pod Anti-Affinity** in Kubernetes

* * *

## 🧭 Summary: Node Affinity vs Pod Anti-Affinity

| Feature | Node Affinity | Pod Anti-Affinity |
| --- | --- | --- |
| **Purpose** | Schedule pods *on specific nodes* based on node labels | Prevent pods from *being placed with other pods* on same node |
| **Matches** | Node labels | Pod labels |
| **Key Use Case** | Targeting specific nodes | Spreading pods across nodes |
| **Configuration** | `spec.affinity.nodeAffinity` | `spec.affinity.podAntiAffinity` |

* * *

## 🧩 Node Affinity

### 🔷 What It Is:

A rule to schedule pods **only on nodes with specific labels**.

### 🔧 How It Works:

- You **label your nodes** (e.g., `region=us-west`, `gpu=true`).
    
- In your pod spec, use `nodeAffinity` to request placement on matching nodes.
    

### ✅ Example:

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: "disktype"
              operator: In
              values:
                - ssd
```

### 🚀 Use Cases:

| Use Case | Description |
| --- | --- |
| **Hardware Targeting** | Run pods only on GPU-enabled nodes (`gpu=true`) |
| **Region or Zone Targeting** | Run pods in specific locations for latency or compliance |
| **Custom Scheduling Rules** | Use labels for custom constraints (e.g., `team=db`) |

* * *

## 🧩 Pod Anti-Affinity

### 🔷 What It Is:

A rule that **prevents pods from being scheduled on nodes** where other **matching pods already run**.

### 🔧 How It Works:

- Define a label selector that **matches other pods**.
    
- The pod will avoid being placed on the **same node** as matching pods.
    

### ✅ Example:

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app
              operator: In
              values:
                - web
        topologyKey: "kubernetes.io/hostname"
```

### 🚀 Use Cases:

| Use Case | Description |
| --- | --- |
| **High Availability (HA)** | Spread replicas across nodes to avoid single point of failure |
| **Performance Isolation** | Prevent noisy neighbors affecting critical workloads |
| **Workload Balancing** | Distribute similar workloads across the cluster |

* * *

## ✅ Recap

- **Use Node Affinity** to **target nodes** with specific features.
    
- **Use Pod Anti-Affinity** to **spread pods out** across nodes for resilience or performance.
    

&nbsp;

**Pod Anti-Affinity soft and hard rules** with YAML examples:

* * *

## Pod Anti-Affinity: Soft Rule vs Hard Rule

### What is Pod Anti-Affinity?

It’s a way to prevent pods from being scheduled on the **same node** as other pods that match certain labels.

* * *

### 1\. **Hard Rule** (RequiredDuringSchedulingIgnoredDuringExecution)

- **Meaning:** The scheduler **must not** schedule the pod on a node if the anti-affinity rule is violated.
    
- This is a **strict, mandatory** constraint.
    
- If no suitable node is found, scheduling **fails**.
    

#### YAML Example — Hard Pod Anti-Affinity:

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app
              operator: In
              values:
                - frontend
        topologyKey: "kubernetes.io/hostname"
```

**Explanation:**

- This pod **cannot be scheduled on any node** that already has pods with label `app=frontend`.
    
- `topologyKey: "kubernetes.io/hostname"` means the rule applies per node (hostname).
    
- If no node satisfies this, pod will stay in **Pending** state.
    

* * *

### 2\. **Soft Rule** (PreferredDuringSchedulingIgnoredDuringExecution)

- **Meaning:** The scheduler **tries to avoid** placing the pod on nodes where the rule is violated, but **it can still schedule** the pod there if no better node exists.
    
- This is a **best-effort, preference** rule.
    
- Useful for improving pod distribution **without causing scheduling failures**.
    

#### YAML Example — Soft Pod Anti-Affinity:

```yaml
affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - frontend
          topologyKey: "kubernetes.io/hostname"
```

**Explanation:**

- The scheduler will **prefer** to avoid nodes with pods labeled `app=frontend`.
    
- `weight` is between 1 and 100 — higher weight means stronger preference.
    
- If no nodes satisfy the preference, the pod **can still be scheduled** there.
    

* * *

## Summary Table

| Rule Type | Field Name | Behavior | Scheduling Impact |
| --- | --- | --- | --- |
| Hard Rule | `requiredDuringSchedulingIgnoredDuringExecution` | Must satisfy rule to schedule | Scheduling fails if no match |
| Soft Rule | `preferredDuringSchedulingIgnoredDuringExecution` | Tries to satisfy rule | Scheduling allowed even if violated |

* * *

## Bonus: Combining Both in One Pod Spec

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app
              operator: In
              values:
                - backend
        topologyKey: "kubernetes.io/hostname"
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 50
        podAffinityTerm:
          labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - frontend
          topologyKey: "kubernetes.io/hostname"
```

- Pod **must not** be scheduled on nodes with `app=backend`.
    
- Pod **prefers not** to be scheduled on nodes with `app=frontend`, but can if needed.
    

* * *

&nbsp;

### 🔍 What is `topologyKey` in Kubernetes Affinity Rules?

`topologyKey` is a **required field** used in **pod affinity** and **pod anti-affinity** rules.  
It tells the Kubernetes scheduler the **topological domain** (scope) to evaluate when deciding **where to co-locate or avoid pods**.

## ✅ In Simple Terms:

- It defines **where** the rule applies:  
    Is it per **node**, per **zone**, per **region**, etc.?

* * *

## 💡 Example Values of `topologyKey`

| `topologyKey` | Meaning |
| --- | --- |
| `kubernetes.io/hostname` | Apply the rule per **node** |
| `topology.kubernetes.io/zone` | Apply the rule per **zone** (e.g., us-east-1a) |
| `topology.kubernetes.io/region` | Apply the rule per **region** |
| Any custom label | You can use custom node labels too |