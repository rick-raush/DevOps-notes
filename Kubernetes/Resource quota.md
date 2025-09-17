## 🔹 What is a Kubernetes ResourceQuota?

A **`ResourceQuota`** is an object that enforces limits on the resources used by all objects (like pods, services, etc.) in a **namespace**.

It is commonly used to:

- Prevent a single team or application from using too much of the cluster's resources.
    
- Control the cost and stability of applications.
    
- Enforce policies in multi-tenant environments.
    

&nbsp;

In **Kubernetes**, there are several **types of quotas** that you can apply using the `ResourceQuota` object. These quotas fall into different **categories based on what they control**, such as compute resources, object counts, storage, and scope-based quotas.

Here’s a breakdown of the main **types of quotas in Kubernetes**:

* * *

## 🔹 1. **Compute Resource Quotas**

These quotas limit the **total amount of CPU and memory** that pods in a namespace can request or use.

| Resource Field | Description |
| --- | --- |
| `requests.cpu` | Total amount of CPU requested by all pods. |
| `limits.cpu` | Total CPU usage limit across all pods. |
| `requests.memory` | Total amount of memory requested. |
| `limits.memory` | Total memory usage limit. |

📌 **Use case:** Control how much compute power a team or app can use.

* * *

## 🔹 2. **Object Count Quotas**

These quotas limit the **number of Kubernetes objects** that can be created in a namespace.

| Resource Field | Description |
| --- | --- |
| `pods` | Maximum number of pods. |
| `services` | Max number of services. |
| `replicationcontrollers` | Max number of ReplicationControllers. |
| `secrets` | Max number of Secrets. |
| `configmaps` | Max number of ConfigMaps. |
| `persistentvolumeclaims` | Max number of PVCs. |
| `statefulsets.apps` | Max number of StatefulSets. |
| `deployments.apps` | Max number of Deployments. |

📌 **Use case:** Prevent users from spamming the cluster with too many objects.

* * *

## 🔹 3. **Storage Resource Quotas**

These quotas limit how much **persistent storage** can be requested by PVCs (PersistentVolumeClaims).

| Resource Field | Description |
| --- | --- |
| `requests.storage` | Total amount of storage requested across PVCs. |
| `persistentvolumeclaims` | Max number of PVCs. |
| `requests.<storage-class>.storage` | Limit storage per StorageClass. |

📌 **Use case:** Prevent storage overuse, especially in multi-tenant environments.

* * *

## 🔹 4. **Scope-Based Quotas**

These are **quotas that only apply to specific types of workloads**, using **scopes**. You define the scope in the `scopeSelector` or `scopes` field.

| Scope Type | Applies To |
| --- | --- |
| `BestEffort` | Pods that do not specify any resource requests/limits. |
| `NotBestEffort` | Pods that *do* specify resource requests/limits. |
| `Terminating` | Pods with `activeDeadlineSeconds` (i.e., temporary pods). |
| `NotTerminating` | Pods without `activeDeadlineSeconds`. |

📌 **Use case:** Enforce different quotas for production vs. temporary or low-priority pods.

* * *

## 🔹 5. **Extended Resource Quotas**

You can also define quotas for **custom or extended resources**, such as GPU usage.

| Example | Description |
| --- | --- |
| `requests.nvidia.com/gpu` | Total number of GPUs requested. |

📌 **Use case:** Limit access to specialized hardware like GPUs.

* * *

## ✅ Example of Mixed ResourceQuota

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mixed-quota
  namespace: dev
spec:
  hard:
    pods: "20"
    requests.cpu: "4"
    limits.cpu: "8"
    requests.memory: "8Gi"
    limits.memory: "16Gi"
    persistentvolumeclaims: "5"
    requests.storage: "100Gi"
```

* * *

## Summary Table

| Type | Limits On | Use Case |
| --- | --- | --- |
| Compute Resources | CPU and memory requests/limits | Prevent resource exhaustion |
| Object Count | Number of Kubernetes objects | Prevent object spamming |
| Storage Resources | PVCs and storage amounts | Control storage use |
| Scope-Based | Specific pod types (e.g., BestEffort) | Prioritize workloads |
| Extended | GPUs, custom resources | Limit specialized resources |

* * *

&nbsp;

(\*\*\*\*)If a user defines only resource limits and not resource requests for a container within a Kubernetes Pod, and a ResourceQuota is in place in that namespace, Kubernetes will automatically assign resource requests that match the defined limits.

This means that if a container specifies a CPU limit but no CPU request, Kubernetes will set the CPU request to be equal to the CPU limit. The same applies to memory: if a memory limit is specified without a memory request, the memory request will be automatically set to the memory limit.

This automatic assignment ensures that the Pod still has resource requests defined, which are necessary for the Kubernetes scheduler to make informed decisions about where to place the Pod and to enforce the ResourceQuota. The ResourceQuota will then count these automatically assigned requests against the quota limits defined for the namespace.

&nbsp;