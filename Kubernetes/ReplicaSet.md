&nbsp;

## What is a ReplicaSet?

- A **ReplicaSet (RS)** is a Kubernetes resource that **ensures a specified number of pod replicas are running** at any given time.
    
- It **creates or deletes pods** as necessary to maintain the desired replica count.
    
- ReplicaSet is the **next-generation version of ReplicationController (RC)** with more powerful label selector capabilities.
    
- Usually, ReplicaSets are **used by Deployments** under the hood.
    

* * *

## Key Differences Between ReplicaSet and ReplicationController

| Concept | RC  | RS  |
| --- | --- | --- |
| Selector support | Equality-based only | Equality + Set-based |
| Typical use case | Legacy, simple workloads | Modern workloads, used by Deployment |
| Flexibility in pod matching | Limited | Flexible |

* * *

* * *

## Equality-based selectors in ReplicaSet

- Use the **`matchLabels`** field to specify **simple equality-based selectors**.
    
- This means you list key-value pairs, and the selector matches pods whose labels exactly match those pairs.
    

### Example (Equality-based selector using `matchLabels`):

```yaml
selector:
  matchLabels:
    app: frontend
    tier: backend
```

- This selects pods with labels `app=frontend` **AND** `tier=backend`.
    
- It’s equivalent to a logical **AND** of all specified labels.
    

* * *

## Set-based selectors in ReplicaSet

- Use the **`matchExpressions`** field to specify **set-based selectors**.
    
- These support operators like `In`, `NotIn`, `Exists`, and `DoesNotExist`.
    
- Allows more complex label matching.
    

### Example (Set-based selector using `matchExpressions`):

```yaml
selector:
  matchExpressions:
  - key: environment
    operator: In
    values:
    - production
    - staging
  - key: tier
    operator: NotIn
    values:
    - cache
```

This selects pods where `environment` is either `production` or `staging` and `tier` is not `cache`.

* * *

### Summary:

| Selector Type | Field Name | Usage |
| --- | --- | --- |
| Equality-based | `matchLabels` | Simple key=value pairs |
| Set-based | `matchExpressions` | Complex expressions using operators like In, NotIn |

* * *

So yes:

- **`matchLabels` is used for equality-based selectors**.
    
- **`matchExpressions` is used for set-based selectors**.
    

* * *

&nbsp;

The **`Exists`** and **`DoesNotExist`** operators are part of **set-based selectors** in Kubernetes and they check for the **presence or absence of a label key**, regardless of the label's value.

* * *

## What do `Exists` and `DoesNotExist` mean?

### 1\. `Exists`

- Matches pods **that have a specific label key**, no matter what the label's value is.
    
- Useful when you just care whether the label is present.
    

**Example:**

```yaml
selector:
  matchExpressions:
  - key: environment
    operator: Exists
```

- This selects all pods **that have the `environment` label**, regardless of whether its value is `production`, `staging`, or anything else.

* * *

### 2\. `DoesNotExist`

- Matches pods **that do NOT have a specific label key**.
    
- Useful to select pods missing a certain label.
    

**Example:**

```yaml
selector:
  matchExpressions:
  - key: environment
    operator: DoesNotExist
```

- This selects all pods **that do NOT have the `environment` label** at all.

* * *

## Summary Table

| Operator | Meaning | Matches Pods Where... |
| --- | --- | --- |
| `Exists` | Label key is present | Pod **has** the specified label key |
| `DoesNotExist` | Label key is missing | Pod **does NOT have** the specified label key |

* * *

&nbsp;

<ins>**ReplicaSet YAML example -**</ins> that **includes both equality-based (`matchLabels`) and set-based (`matchExpressions`) selectors together** using the unified `selector` syntax:

* * *

### ReplicaSet with Both `matchLabels` and `matchExpressions`

```yaml
#before writing it we can do: (kubectl explain rs --recursive|less) to see the structure

apiVersion: apps/v1

kind: ReplicaSet

metadata:
  name: my-replicaset-mixed

spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      tier: backend
    matchExpressions:
    - key: environment
      operator: In
      values:
      - production
      - staging
    - key: debug
      operator: Exists
  template:
    metadata:
      labels:
        app: myapp
        tier: backend
        environment: production
        debug: "true"
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```

* * *

### Explanation:

- **`matchLabels`**: Selects pods with `app=myapp` AND `tier=backend`.
    
- **`matchExpressions`**:
    
    - `environment` label value must be in `[production, staging]`.
        
    - `debug` label must exist (value doesn’t matter).
        
- Pods **must match all conditions** in both `matchLabels` and `matchExpressions`.
    
- The pod template labels satisfy these conditions.
    

* * *

&nbsp;

&nbsp;

If you update a YAML and change the **name** of the resource (like a Pod, ReplicaSet, or ReplicationController) and then apply it, Kubernetes will **create a new resource with the new name** rather than rename the existing one.

* * *

### Why?

- In Kubernetes, the **name** is a unique identifier for a resource.
    
- Changing the name means it’s a **different resource**.
    
- `kubectl apply` sees it as a **new resource** to create.
    
- The old resource (with the old name) **remains unchanged** unless you delete it explicitly.
    

* * *

### Example:

- You have a pod named `mypod`.
    
- You update the YAML and change the name to `mypod-new`.
    
- Running `kubectl apply -f updated.yaml` will:
    
    - Create a new pod called `mypod-new`.
        
    - Leave the original pod `mypod` running.
        

* * *

### Summary:

| Action | Result |
| --- | --- |
| Change `metadata.name` and apply | New resource created, old remains |
| Keep same name and change spec | Existing resource updated |

* * *

If you want to rename a resource, the usual approach is:

1.  Create the new resource with the new name.
    
2.  Delete the old resource manually.
    

* * *

&nbsp;

Whether a resource gets **updated in place** or a **new resource is created** depends mostly on the **resource’s name and immutable fields**.

* * *

### What you **cannot** change without creating a new resource:

- **`metadata.name`** (the resource’s name): changing this creates a new resource.
    
- Certain **immutable fields** depending on resource type (e.g., for Pods: `spec.containers[*].name`, for PersistentVolumes: `spec.capacity.storage`).
    
- For some resources, the **`spec.selector`** is immutable (e.g., ReplicaSet, Deployment).
    
- Some fields are cluster-generated and can't be changed directly (like `metadata.uid`).
    

* * *

### What you **can** change and have Kubernetes update the existing resource:

- Most **spec fields** that are mutable:
    
    - Container images (`spec.containers[*].image`)
        
    - Resource requests and limits
        
    - Environment variables
        
    - Labels and annotations (except some selectors or labels used in selectors)
        
    - Number of replicas in controllers (ReplicaSet, Deployment)
        
    - Pod template metadata labels and spec (for controllers)
        
- `metadata.labels` and `metadata.annotations` (except labels used as selectors)
    

* * *

### Important notes:

- For **Pods**, many fields are immutable after creation (e.g., container specs), so often a Pod must be deleted and recreated for certain changes.
    
- For **controllers** like ReplicaSets or Deployments, you can update the Pod template spec and Kubernetes will handle rolling updates.
    

* * *

### Summary Table

| Field | Can Change & Update In-place? | Comments |
| --- | --- | --- |
| `metadata.name` | No  | Creates new resource |
| `spec.selector` (in controllers) | No  | Immutable |
| Container image | Yes | Triggers rollout in controllers |
| Replicas count | Yes | Scale up/down |
| Labels/Annotations | Yes (except selector labels) | Can update freely |
| Container names (in Pods) | No  | Immutable in Pods |

* * *

&nbsp;

&nbsp;