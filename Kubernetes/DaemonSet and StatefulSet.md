&nbsp;

# DaemonSet

### What is it?

- A **DaemonSet** ensures that **a copy of a specific Pod runs on every node** (or a subset of nodes based on selectors/taints).
    
- Whenever a new node is added to the cluster, the DaemonSet automatically adds the pod to that node.
    
- When a node is removed, the pod is cleaned up.
    

### Use Cases

- Running cluster-level agents like:
    
    - Monitoring agents (e.g., Prometheus node exporters).
        
    - Log collectors (e.g., Fluentd, Logstash).
        
    - Networking components (e.g., Calico, Weave Net).
        
    - Security agents.
        
- Ensuring that infrastructure components run on all or some nodes.
    

### Key Characteristics

- One Pod per selected node.
    
- Pods are automatically scheduled on new nodes.
    
- Not suited for user applications that require storage or ordered deployment.
    

### Example YAML for DaemonSet

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        ports:
        - containerPort: 9100
```

&nbsp;

### Why is DaemonSet a Namespaced Resource?

1.  **Scope & Organization:**
    
    - Namespaces provide a **logical partition** within a Kubernetes cluster.
        
    - They help organize and isolate resources (like DaemonSets) by teams, projects, or environments.
        
    - DaemonSets often belong to a specific application or service running within a namespace, so it makes sense for them to be namespaced.
        

* * *

# StatefulSet

### What is it?

- **StatefulSet** manages **stateful applications** that require:
    
    - Stable, unique network identities (unique DNS).
        
    - Stable, persistent storage.
        
    - Ordered, graceful deployment, scaling, and deletion.
        
- Each Pod in a StatefulSet has:
    
    - A unique, stable identity (name like `pod-0`, `pod-1`).
        
    - Persistent storage that is retained even if the Pod is rescheduled.
        

### Use Cases

- Databases (MySQL, MongoDB, Cassandra).
    
- Distributed systems requiring stable identities (Zookeeper, Kafka).
    
- Applications needing stable, persistent storage volumes.
    

### Key Characteristics

- Pods are created and deleted in a defined order (`pod-0`, `pod-1`, ...).
    
- Each Pod gets a persistent volume claim (PVC) that sticks to it.
    
- Pods maintain stable hostnames and network IDs.
    
- Supports scaling up/down with ordered deployment.
    

### Example YAML for StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: "mysql-service"  # Headless service for stable network ID
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-persistent-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
      
      storageclass: "fast-storageabc"
```

* * *

## Summary Comparison

| Feature | DaemonSet | StatefulSet |
| --- | --- | --- |
| Purpose | Run one pod on **every** (or subset) node | Manage stateful app pods with stable IDs and storage |
| Pod Identity | No stable network identity | Stable network ID (`pod-0`, `pod-1` etc.) |
| Storage | Usually no persistent storage | Supports persistent volumes per pod |
| Scaling | Scales with cluster nodes | User-controlled replica scaling |
| Use Cases | Monitoring, logging, networking agents | Databases, clustered apps with state and identity |
| Pod creation order | No ordered creation | Pods created/deleted in order |

* * *

&nbsp;

* * *

## 1\. **StatefulSet having a stable network identity**

In Kubernetes, a **StatefulSet** manages pods with unique, persistent identities and stable network IDs. Here’s what that means:

- **Stable Network ID** means:
    
    - Each pod in a StatefulSet gets a **unique, stable hostname** based on its ordinal index.
        
    - For example, if the StatefulSet is named `web`, its pods will be named `web-0`, `web-1`, `web-2`, etc.
        
    - These pods have **stable DNS names** like `web-0.my-service.default.svc.cluster.local`.
        
    - Even if a pod is deleted and recreated, its hostname **does not change**.
        
    - This stable identity is important for applications that require stable network endpoints — like databases or clustered applications.
        
- **Why is this important?**
    
    - Some applications (e.g., databases like MySQL, Cassandra) rely on knowing the exact identity or address of each member in the cluster.
        
    - StatefulSets ensure pods have persistent identities and network IDs to support such stateful workloads.
        

* * *

## 2\. **`volumeClaimTemplates` in StatefulSets**

The `volumeClaimTemplates` field is a powerful feature that helps StatefulSets manage **persistent storage** for each pod automatically.

- **What it does:**
    
    - Defines a **template for PersistentVolumeClaims (PVCs)** that Kubernetes will create **dynamically** for each pod in the StatefulSet.
        
    - Each pod gets its **own PersistentVolumeClaim** based on this template, so storage is **dedicated and stable per pod**.
        
    - When a pod is recreated, it gets **the same PVC** and thus the same storage, preserving data.
        

* * *

### Explanation of your example YAML snippet:

```yaml
volumeClaimTemplates:
  - metadata:
      name: mysql-persistent-storage      # The name used to mount the volume in the pod spec
    spec:
      accessModes: ["ReadWriteOnce"]       # The access mode: only one node can mount this volume for writing
      resources:
        requests:
          storage: 10Gi                    # Request 10 GiB of storage for each PVC
```

- For each pod in the StatefulSet, Kubernetes creates a PVC named `mysql-persistent-storage-<pod-name>`.
    
- These PVCs will request **10Gi** of storage each.
    
- The `accessModes: ReadWriteOnce` means each volume can only be mounted as read-write by a single node at a time (common for block storage).
    
- This storage is **persistent**, so pod restarts or rescheduling won’t lose data.
    

* * *

### Summary

| Concept | Explanation |
| --- | --- |
| **Stable Network ID** | Each StatefulSet pod gets a unique, persistent hostname and DNS name that doesn’t change. |
| **volumeClaimTemplates** | Template to auto-create PVCs for each pod to ensure each pod has its own persistent storage. |
| **AccessModes** | Controls how the volume can be mounted (e.g., ReadWriteOnce = single node write access). |
| **Storage Request** | Specifies how much storage each pod's PVC should request (e.g., 10Gi). |

* * *

&nbsp;

## How PVCs in StatefulSets relate to dynamic provisioning and reclaim policies:

* * *

### 1\. **Dynamic Provisioning of PVCs in StatefulSets**

- When you define a **`volumeClaimTemplates`** inside a StatefulSet, Kubernetes creates one PVC per pod automatically.
    
- These PVCs typically use **dynamic provisioning**:
    
    - The PVC references a **StorageClass** (either explicitly or via default StorageClass).
        
    - The StorageClass defines **how and where** the PersistentVolumes (PVs) are created.
        
    - The cloud provider or the cluster’s storage plugin dynamically provisions the actual PV behind the scenes based on the StorageClass parameters.
        
- **So, for each pod:**
    
    - A PVC is created from the `volumeClaimTemplates`.
        
    - Kubernetes dynamically provisions a matching PV as per the StorageClass.
        
    - The PVC is **bound** to that PV.
        
    - The pod mounts the volume via the PVC, so each pod gets its own persistent storage.
        

* * *

&nbsp;

### 3. **Summary Flow**

| Step | Description |
| --- | --- |
| StatefulSet with `volumeClaimTemplates` | Creates PVC per pod automatically |
| PVC references a StorageClass | StorageClass defines dynamic provisioning details |
| StorageClass provisions a PV | A matching PersistentVolume is dynamically created |
| Pod mounts PVC | Pod uses persistent storage with stable data |
| PVC deletion triggers reclaim policy | StorageClass’s reclaim policy determines PV fate |

* * *

### Example

If your StorageClass has:

```yaml
reclaimPolicy: Delete
```

- When you delete the PVC (or StatefulSet), the PV and its storage are deleted automatically.

If your StorageClass has:

```yaml
reclaimPolicy: Retain
```

- The PV and data persist even after PVC deletion, and manual intervention is needed to reuse or delete the PV.

* * *

### Key points:

- **You do not specify reclaim policy in `volumeClaimTemplates`.**
    
- **Reclaim policy is set in the StorageClass**, which handles PV lifecycle.
    
- Always check your StorageClass reclaim policy to understand how storage cleanup behaves.
    

* * *

&nbsp;

## **How you add a StorageClass to `volumeClaimTemplates` in a StatefulSet YAML**:

- **We specify the StorageClass** inside the `volumeClaimTemplates` by adding the `storageClassName` field under the PVC spec.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-persistent-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "fast-storage"        # <----- Specify StorageClass here
      resources:
        requests:
          storage: 10Gi
```

* * *

### Explanation:

- `storageClassName: "fast-storage"` tells Kubernetes to use the **StorageClass named `fast-storage`** for dynamically provisioning the PV.
    
- If you omit `storageClassName`, Kubernetes will use the **default StorageClass** (if one exists in your cluster).
    
- You can create different StorageClasses for different types of storage (e.g., SSD-backed, network storage, etc.) and choose the appropriate one here.
    

* * *

&nbsp;