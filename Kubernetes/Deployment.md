&nbsp;

## ✅ What is a Deployment in Kubernetes?

A **Deployment** is a Kubernetes resource that helps you **manage and update a group of Pods** in a controlled and automated way.

A **Deployment** is a **higher-level controller** in Kubernetes that:

- **Manages ReplicaSets**, which in turn manage **Pods**.
    
- Provides **automated rollout, rollback, and update management** for your application.
    
- Helps ensure zero downtime during updates using **rolling updates**.
    
- Tracks the **history** of changes (versions) for potential rollback.
    

* * *

## 🤔 Why Did We Need Deployment if We Already Had ReplicaSet?

A **ReplicaSet** ensures a desired number of Pods are running — but it’s **not very smart** beyond that. It doesn’t manage **updates**, **rollbacks**, or **version control**.

* * *

### 🔥 Limitations of ReplicaSet (RS)

| Feature | ReplicaSet Supports? |
| --- | --- |
| Keep N pods running | ✅ Yes |
| Rolling updates | ❌ No |
| Rollbacks | ❌ No |
| Declarative update history | ❌ No |
| Strategy control (e.g., maxSurge) | ❌ No |

If you wanted to **update your app** with a new image or configuration using ReplicaSets, you'd have to:

- Manually create a new RS,
    
- Scale down the old one,
    
- Scale up the new one,
    
- Monitor the rollout manually...
    

→ **This is error-prone and not scalable**.

* * *

### ✅ What Deployments Add on Top of ReplicaSets

| Feature | Description |
| --- | --- |
| **Rolling updates** | Seamlessly updates Pods one-by-one or in batches (configurable). |
| **Rollback support** | Easily revert to a previous RS version using one command. |
| **Declarative updates** | You declare what you want, Deployment manages the transition. |
| **History tracking** | Maintains revision history for RS versions it has created. |
| **Strategy config** | Supports `RollingUpdate` and `Recreate` strategies with options like `maxSurge`, `maxUnavailable`. |

* * *

&nbsp;

![5a1ecbdc80dd82777af4ff82a952fa8e.png](../_resources/5a1ecbdc80dd82777af4ff82a952fa8e.png)

&nbsp;

Here's a **fully structured and well-formatted Deployment YAML** named `deployment1.yml`, covering all the main fields you’ll commonly use in a real-world deployment.

* * *

## 📄 `deployment1.yml` – Full Example with Structure & Syntax

```yaml
apiVersion: apps/v1           # API group for Deployments
kind: Deployment              # Resource type

metadata:
  name: my-deployment         # Name of the deployment
  labels:                     # Labels for the deployment resource itself
    app: myapp
    environment: production

spec:
  replicas: 3                 # Desired number of pod replicas

  selector:                   # How to identify the pods this Deployment manages
    matchLabels:
      app: myapp              # Must match pod template labels exactly

  strategy:                   # (Optional) Rollout strategy
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1             # Allow 1 extra pod (above desired count) during update
      maxUnavailable: 1       # Allow 1 unavailable pod during update

  template:                   # Pod template — defines what the pods will look like
    metadata:
      labels:                 # These labels must match the selector above
        app: myapp
        tier: backend

    spec:
      containers:
      - name: myapp-container             # Container name
        image: nginx:1.25                 # Docker image to use
        ports:
        - containerPort: 80               # Port exposed inside the pod

        env:                              # (Optional) Environment variables
        - name: ENV
          value: production

        resources:                        # (Optional) Resource requests/limits
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "250m"
            memory: "256Mi"

        livenessProbe:                    # (Optional) Pod health check
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 5

        readinessProbe:                   # (Optional) Determines if pod is ready for traffic
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5

      restartPolicy: Always               # Default for Deployments (optional)
```

* * *

## ✅ Key Sections Explained

| Section | Purpose |
| --- | --- |
| `apiVersion`, `kind` | Define the resource type |
| `metadata` | Names and labels the deployment |
| `spec.replicas` | Number of desired pods |
| `spec.selector` | Identifies pods this deployment manages |
| `spec.strategy` | Controls how updates are rolled out |
| `spec.template` | Template for the pods (must match selector) |
| `template.spec.containers` | Defines the actual app container(s) |

* * *

## 🧪 Deploy it:

```bash
kubectl apply -f deployment1.yml
```

* * *

&nbsp;

&nbsp;

* * *

## ✅ When is a Deployment **rollout triggered**?

A **rollout** happens **only when Kubernetes detects a change in the Pod template** inside the Deployment.  
That means:

### ✅ **Rollout is triggered when you change:**

- `spec.template.spec.containers[*].image` (e.g., changing the image version)
    
- Environment variables
    
- Resource limits
    
- Volume mounts
    
- Labels/annotations inside the **`spec.template.metadata`** (not the outer metadata)
    

> Basically, anything inside `spec.template` — since that's what defines the Pod.

* * *

### ❌ Rollout is **not triggered** when you change:

- `metadata.name` or labels of the Deployment itself
    
- `spec.replicas` (just scales, doesn’t roll out new Pods)
    
- Labels/annotations outside `spec.template`
    
- Strategy fields like `maxSurge` or `maxUnavailable`
    

* * *

## 🔍 Commands to **monitor rollout**

### ✅ 1. Check rollout status:

```bash
kubectl rollout status deployment <deployment-name>
```

> Shows if the Deployment is still updating or has completed the rollout.

* * *

### ✅ 2. View rollout history:

```bash
kubectl rollout history deployment <deployment-name>
```

> Shows previous versions (revisions) of the Deployment, including image changes.

* * *

### ✅ 3. Watch changes in real time:

```bash
kubectl get pods -l app=<your-app-label> -w
```

> The `-w` flag watches pod changes as the rollout progresses.

* * *

## 📌 Bonus: Manually trigger a rollout (without changing spec)

If you want to **force a rollout** even when there’s no real change, use:

```bash
kubectl rollout restart deployment <deployment-name>
```

> This forces the Deployment to restart all Pods, triggering a rollout.

* * *

## Summary

| Action | Triggers Rollout? |
| --- | --- |
| Changing container image | ✅ Yes |
| Changing env vars / volume mounts | ✅ Yes |
| Changing replica count | ❌ No |
| Changing labels outside pod template | ❌ No |
| `kubectl rollout restart` | ✅ Yes (manual) |

* * *

&nbsp;

```bash
kubectl apply -f dep.yml; kubectl get rs -o wide -w
```

is **similar**, but not **exactly** the same as:

```bash
kubectl apply -f dep.yml; watch kubectl get rs -o wide
```

Let’s break down the **differences** 👇

* * *

## 🔁 Difference Between `watch` vs `kubectl -w`

| Command | Behavior |
| --- | --- |
| `watch kubectl get rs -o wide` | Re-runs `kubectl get` every 2 seconds (snapshot view on interval) |
| `kubectl get rs -o wide -w` | Opens a **stream** to watch real-time updates as they happen (event-based) |

* * *

&nbsp;

&nbsp;

**`maxUnavailable`**, **`maxSurge`** in a Kubernetes **Deployment**'s **rolling update strategy**.

* * *

## 🚀 Where are these used?

They are part of the Deployment’s **rolling update strategy**, which controls how Kubernetes **updates pods** with zero or minimal downtime.

You define them under:

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: <value>
      maxUnavailable: <value>
  minReadySeconds: <value>
```

* * *

## ✅ 1. `maxSurge`

> **How many extra pods can be created temporarily during an update.**

- It allows Kubernetes to **create more pods than desired replicas** (temporarily) to maintain availability.
    
- You can specify:
    
    - An absolute number (e.g. `1`)
        
    - A percentage (e.g. `25%`)
        

### Example:

```yaml
replicas: 3
rollingUpdate:
  maxSurge: 1
```

- During an update, up to **4 pods** (3 desired + 1 surge) can exist temporarily.
    
- This helps keep service available while old pods are being terminated.
    

* * *

## ✅ 2. `maxUnavailable`

> **How many pods can be unavailable during the update.**

- Limits how many pods **can be taken down at the same time** while updating.
    
- You can specify:
    
    - A number (e.g. `1`)
        
    - A percentage (e.g. `25%`)
        

### Example:

```yaml
replicas: 4
rollingUpdate:
  maxUnavailable: 1
```

- At most **1 pod can be unavailable** at any moment during the rollout.
    
- Ensures minimum service availability.
    

* * *

### 🔄 How `maxSurge` and `maxUnavailable` work together

| Setting | Effect |
| --- | --- |
| `maxSurge: 1` | 1 extra pod allowed during update (temp total = 4 pods) |
| `maxUnavailable: 1` | At most 1 pod can be taken down during the update |

These values **balance speed and availability** during rollouts.

* * *

## `minReadySeconds`

> **How long a new pod must be “ready” before it's considered available.**

- Prevents rolling forward too quickly by ensuring that new pods are **stable**.
    
- Helps catch **flaky pods** or startup issues.
    

### Example:

```yaml
minReadySeconds: 10
```

- A newly started pod must be in the `Ready` state for **at least 10 seconds** before it's considered "available".
    
- The Deployment won't proceed with replacing more pods until this time passes.
    

* * *

## 🧠 Summary Table

| Field | Purpose | Example |
| --- | --- | --- |
| `maxSurge` | How many **extra pods** can be added during update | `1` or `25%` |
| `maxUnavailable` | How many **pods can be down** during update | `0`, `1`, or `30%` |
| `minReadySeconds` | How long a new pod must be ready before considered valid | `10` seconds |

* * *

## 🧪 Bonus Example:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
minReadySeconds: 5
```

- Allows 1 extra pod
    
- Ensures no downtime (`maxUnavailable: 0`)
    
- Each new pod must be stable for 5 seconds before moving on
    

* * *

&nbsp;

Excellent question! 🙌

In a Kubernetes **Deployment YAML**, the fields you mentioned:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
minReadySeconds: 5
```

They are part of the **`spec`** section.

* * *

### ✅ Full YAML Structure (Simplified for Clarity)

```yaml
apiVersion: apps/v1          # <-- Top-level: API group
kind: Deployment             # <-- Resource type
metadata:                    # <-- Metadata (name, labels)
  name: my-deployment

spec:                        # <-- The main Deployment spec
  replicas: 3                # Number of pods
  selector:                  # How to identify managed pods
    matchLabels:
      app: myapp

  strategy:                  # <-- Update strategy lives here
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0

  minReadySeconds: 5         # <-- Also part of spec (same level as strategy)

  template:                  # <-- Pod template
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx:latest
```

* * *

### ✅ Final Answer:

- **`strategy`** and **`minReadySeconds`** both go under:  
    🔽 `**spec**` of the **Deployment**

* * *

&nbsp;

&nbsp;