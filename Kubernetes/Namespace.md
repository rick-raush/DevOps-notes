&nbsp;

## 🗂️ What is a **Namespace** in Kubernetes?

A **Namespace** in Kubernetes is a **logical partition** within a cluster. It allows you to **isolate resources** (like pods, services, deployments, etc.) within the same physical cluster.

> Think of it like a "folder" or "project space" inside your cluster.

* * *

### ✅ Why Use Namespaces?

- **Multi-tenancy**: Different teams/projects can work in the same cluster without interfering with each other.
    
- **Security & Access Control**: RBAC can restrict access to specific namespaces.
    
- **Resource Isolation**: You can set quotas to control how much CPU/memory each namespace can use.
    
- **Organization**: Makes large environments cleaner and easier to manage.
    

* * *

## 🔍 Resources that **support namespaces**

Not all Kubernetes resources live inside a namespace. Here's a clear breakdown:

### ✅ Namespaced Resources (most common):

| Resource Type | Namespaced? | Examples |
| --- | --- | --- |
| Pods | ✅ Yes | `kubectl get pods -n my-ns` |
| Deployments | ✅ Yes |     |
| ReplicaSets | ✅ Yes |     |
| Services | ✅ Yes |     |
| ConfigMaps | ✅ Yes |     |
| Secrets | ✅ Yes |     |
| PersistentVolumeClaims (PVCs) | ✅ Yes |     |
| Jobs & CronJobs | ✅ Yes |     |
| Ingress | ✅ Yes |     |
| ResourceQuota | ✅ Yes |     |
| LimitRange | ✅ Yes |     |

* * *

### ❌ Non-Namespaced (Cluster-wide) Resources:

| Resource Type | Namespaced? | Purpose |
| --- | --- | --- |
| Nodes | ❌ No | Represents a worker machine |
| PersistentVolumes (PVs) | ❌ No | Cluster-wide storage resources |
| Namespaces | ❌ No | Themselves are not namespaced |
| ClusterRoles | ❌ No | RBAC at the cluster level |
| ClusterRoleBindings | ❌ No | Bind users/groups to ClusterRoles |
| CustomResourceDefinitions (CRDs) | ❌ No | Define new resource types |
| StorageClass | ❌ No | Define how volumes are provisioned |

* * *

## 🛠️ Basic Namespace Commands

### 🔹 List all namespaces:

```bash
kubectl get namespaces
```

### 🔹 Create a namespace:

```bash
kubectl create namespace dev-team
```

### 🔹 Use a namespace (with `kubectl`):

```bash
kubectl get pods -n dev-team
```

### 🔹 Set a default namespace (for your `kubectl` context):

```bash
kubectl config set-context --current --namespace=dev-team
```

* * *

## 🧠 TL;DR

| Feature | Description |
| --- | --- |
| **Namespace** | Logical partition in a cluster |
| **Used for** | Isolation, organization, access control |
| **Most resources** | Are namespaced (pods, services, etc.) |
| **Some resources** | Are cluster-wide (nodes, CRDs, roles) |

* * *

&nbsp;

* * *

## 🖼️ Visual Diagram: Namespaced vs Non-Namespaced Resources

```
#can check using:  kubectl api-resources

┌──────────────────────────── Kubernetes Cluster ─────────────────────────────┐
│                                                                             │
│   ┌────────────── Namespace: dev ──────────────┐      Cluster-Wide Resources│
│   │                                            │                             │
│   │  [Pod: app-dev]                            │                             │
│   │  [Service: dev-service]                    │                             │
│   │  [Deployment: dev-deploy]                  │                             │
│   │  [Secret: db-password]                     │                             │
│   │  [PVC: dev-volume-claim]                   │                             │
│   └────────────────────────────────────────────┘                             │
│                                                                             │
│   ┌────────────── Namespace: prod ─────────────┐                             │
│   │                                            │                             │
│   │  [Pod: app-prod]                           │                             │
│   │  [Service: prod-service]                   │                             │
│   │  [Deployment: prod-deploy]                 │                             │
│   │  [ConfigMap: prod-config]                  │                             │
│   │  [PVC: prod-volume-claim]                  │                             │
│   └────────────────────────────────────────────┘                             │
│                                                                             │
│   Cluster-Wide Resources:                                                   │
│     [Node: ip-10-0-0-1]                                                     │
│     [PersistentVolume: pv-001]                                              │
│     [ClusterRole: admin]                                                    │
│     [ClusterRoleBinding: team-admins]                                       │
│     [CRD: custom.mycompany.com]                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

* * *

&nbsp;

### **Get All Kubernetes Resources**

* * *

#### 📌 **All resources in a specific namespace:**

```bash
kubectl get all -n <namespace>
```

- Shows: Pods, Services, Deployments, etc.

To include more (like ConfigMaps, Secrets, PVCs):

```bash
kubectl get all,cm,secret,pvc,ingress -n <namespace>
```

* * *

#### 🌍 **All resources in the entire cluster (all namespaces):**

```bash
kubectl get all --all-namespaces
```

To include **everything** (cluster-wide and namespaced):

```bash
kubectl api-resources -o name | xargs -n1 kubectl get --all-namespaces
```

* * *

#### 🧠 TL;DR

| Task | Command |
| --- | --- |
| All in a namespace | `kubectl get all -n <namespace>` |
| All + extras in namespace | `kubectl get all,cm,secret,pvc -n <ns>` |
| All across all namespaces | `kubectl get all --all-namespaces` |
| Absolutely everything | `kubectl api-resources -o name \| xargs ...` |

&nbsp;

&nbsp;

&nbsp;

To **declaratively add or specify a namespace** in Kubernetes, you have a few options depending on what you're defining.

Here's a **clear breakdown** for different use cases:

* * *

## ✅ 1. **Create a Namespace Declaratively**

```bash
#Better to use: 
kubectl create namespace new_namespace_name

```

Create a YAML file called `namespace.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace
```

Apply it:

```bash
kubectl apply -f namespace.yaml
```

* * *

## ✅ 2. **Assign a Resource to a Namespace Declaratively**

Inside your Kubernetes manifest (e.g., `manifest.yml`), add the `namespace` field under `metadata`.

### 🔹 Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: my-namespace  # 👈 Declarative namespace assignment
spec:
  containers:
    - name: nginx
      image: nginx
```

This ensures the pod is created **in `my-namespace`**, not the default.

* * *

## ✅ 3. **Declarative Namespace in `Jenkinsfile` (for Deployment)**

In your **Jenkins pipeline**, pass the namespace as an environment variable and use it in your `kubectl` commands:

```groovy
environment {
  NAMESPACE = 'my-namespace'
}

...

stage('Deploy') {
  steps {
    sh "kubectl apply -f Service-v1/${SERVICE}/manifest.yml -n ${NAMESPACE}"
  }
}
```

* * *

## ✅ Summary

| Purpose | Declarative Example |
| --- | --- |
| Create a namespace | `kind: Namespace` YAML |
| Assign resource to namespace | `metadata.namespace: my-namespace` in YAML |
| Use in Jenkins CI/CD | Use `-n ${NAMESPACE}` in `kubectl apply` |

&nbsp;

&nbsp;

&nbsp;

Kubernetes creates **four default namespaces** automatically when a cluster is initialized.

Here's a breakdown of each:

* * *

## 📂 **1\. `default`**

- The **default namespace** for user-created resources **when none is specified**.
    
- If you create a pod, service, etc., without setting a namespace, it goes here.
    

```bash
kubectl create deployment my-app --image=nginx
# This goes into 'default' unless specified otherwise
```

* * *

## ⚙️ **2\. `kube-system`**

- Contains **Kubernetes core components** managed by the system.
    
- Includes things like:
    
    - kube-dns / CoreDNS
        
    - kube-proxy
        
    - kube-controller-manager
        
    - cloud-controller-manager (if used)
        
    - etcd (on control plane node)
        

**⚠️ Do not modify resources here unless you know what you're doing.**

* * *

## 🔐 **3\. `kube-public`**

- Readable by **all users**, including unauthenticated users.
    
- Mostly used for cluster-wide public information, like the cluster's public config.
    
- Rarely used by users directly.
    

```bash
kubectl get configmap -n kube-public
```

* * *

## 👮 **4\. `kube-node-lease`**

- Used for **node heartbeats** and tracking node availability.
    
- Each node has a corresponding Lease object updated frequently to signal it’s healthy.
    
- Helps Kubernetes scale efficiently by reducing load on etcd.
    

* * *

## 🧠 Summary Table

| Namespace | Purpose |
| --- | --- |
| `default` | Where user resources go when no namespace is set |
| `kube-system` | Kubernetes system components |
| `kube-public` | Public data accessible by all (including unauth) |
| `kube-node-lease` | Tracks node heartbeats for availability checks |

* * *

You can list them using:

```bash
kubectl get namespaces
```

&nbsp;