Absolutely! Let’s go **from the ground up** and explain the Kubernetes **Control Plane** and how it works when you apply a **Deployment**, step by step. I’ll keep it clear, practical, and structured.

* * *

# **1️⃣ What is the Control Plane**

The **control plane** is the “brain” of a Kubernetes cluster. It **manages the cluster**, makes scheduling decisions, ensures desired state, and exposes the API to users and agents.

### **Main Components**

1.  **kube-apiserver**
    
2.  **etcd**
    
3.  **kube-scheduler**
    
4.  **kube-controller-manager**
    
5.  **cloud-controller-manager** (optional if using cloud)
    

* * *

# **2️⃣ Component Details**

### **A) kube-apiserver**

- **Role:** Frontend for the Kubernetes API. All operations go through it.
    
- **Responsibilities:**
    
    - Validate API requests (`kubectl apply`, `kubectl get pods`).
        
    - Authenticate & authorize requests.
        
    - Serve data from **etcd** (cluster state).
        
    - Act as the **hub** for all components (scheduler, controllers, kubelets).
        
- **Command Example:**
    
    ```bash
    kubectl get componentstatuses
    ```
    
- **Notes:**
    
    - Stateless; talks to **etcd** for state storage.
        
    - Every `kubectl apply` → API server validates & stores in etcd.
        

* * *

### **B) etcd**

- **Role:** Distributed key-value store; the **source of truth** for the cluster.
    
- **Responsibilities:**
    
    - Stores cluster state: Pods, Services, ConfigMaps, Deployments, etc.
        
    - Provides consistent, reliable storage for control plane components.
        
- **Example:**
    
    - Deployment spec applied → stored in etcd.
- **Notes:**
    
    - Critical for cluster operation; if etcd is down → cluster state can’t be read or updated.

* * *

### **C) kube-scheduler**

- **Role:** Assigns newly created pods to nodes.
    
- **Responsibilities:**
    
    - Watches for unscheduled pods via API server.
        
    - Chooses the best node based on:
        
        - Resource requests/limits
            
        - Node taints/tolerations
            
        - Affinity/anti-affinity
            
        - Policies
            
- **Example:**
    
    ```bash
    kubectl describe pod <pod-name>
    ```
    
    - You’ll see `Node:` assigned by scheduler.
- **Notes:** Only runs for **unscheduled pods**.
    

* * *

### **D) kube-controller-manager**

- **Role:** Runs **controllers** that maintain the cluster state.
    
- **Key Controllers:**
    
    1.  **ReplicationController / ReplicaSet** → Ensures the desired number of pod replicas.
        
    2.  **Deployment Controller** → Manages rolling updates, scaling, rollback.
        
    3.  **Node Controller** → Detects node failure.
        
    4.  **Endpoints Controller** → Updates Service → Pod mappings.
        
    5.  **Service Account & Token Controllers**
        
- **Responsibilities:**
    
    - Compare **desired state** (from etcd) vs **current state** (from nodes via kubelet) and take corrective actions.

* * *

### **E) cloud-controller-manager** (if using cloud)

- Interacts with cloud APIs for:
    
    - Node management
        
    - Load balancers
        
    - Storage provisioning
        
- Only present in cloud environments.
    

* * *

# **3️⃣ What happens when you apply a Deployment**

The **Deployment Controller** learns about a new or updated **Deployment** object through the **Kubernetes API Server** using a process called **watching**. Similarly, the Deployment Controller interacts with the API Server, not etcd directly, to create and store the **ReplicaSet**.

* * *

## 1\. How the Deployment Controller Knows a Deployment Exists

When you run `kubectl apply -f deployment.yaml`, the following sequence of events occurs:

1.  **`kubectl` sends the request:** The `kubectl` CLI acts as a client and sends an HTTP request (a POST request for a new resource or a PUT/PATCH for an update) containing the Deployment YAML/JSON to the **Kubernetes API Server**.<sup>1</sup>
    
2.  **API Server stores the object in etcd:** The API Server is the only component that directly interacts with **etcd** (the cluster's database and single source of truth).<sup>2</sup> After validation and admission control, the API Server persists the Deployment object into etcd.<sup>3</sup>
    
3.  **API Server Notifies Controllers:** The Kubernetes **Controller Manager** (which hosts the Deployment Controller and many other controllers) maintains persistent, long-lived connections (**watches**) with the API Server. These watches allow the API Server to stream real-time notifications about changes (creation, updates, deletions) to resources like Deployments.
    
4.  **Deployment Controller Reacts:** The **Deployment Controller** receives the notification about the new Deployment object from the API Server.<sup>4</sup> This triggers its control loop to start the reconciliation process, whose goal is to make the cluster's current state match the desired state defined in the Deployment object.
    

The key takeaway is that the controller never looks at etcd itself; it relies entirely on the **API Server's Watch mechanism** to be informed of state changes.

* * *

## 2\. How the Deployment Controller Creates the ReplicaSet

The Deployment Controller also does **not** write the **ReplicaSet** object directly to etcd. It interacts with the **API Server** for all cluster state modifications:

1.  **Controller Creates the Object:** When the Deployment Controller needs a ReplicaSet, it constructs the full ReplicaSet object definition (including the appropriate labels, selectors, and the Pod template copied from the Deployment).
    
2.  **Controller Sends Request to API Server:** The Deployment Controller sends this complete ReplicaSet object definition as an HTTP POST request to the **Kubernetes API Server**'s `/apis/apps/v1/namespaces/{namespace}/replicasets` endpoint.
    
3.  **API Server Stores and Notifies:** The API Server receives the request, validates the ReplicaSet, writes the object to **etcd**, and then sends a notification (via its watch mechanism) that a new ReplicaSet exists. This notification is received by the **ReplicaSet Controller**, which then takes over the task of creating the individual Pods.
    

The entire principle of Kubernetes is that no component (like a controller or Kubelet) is allowed to access **etcd** directly. **The API Server is the sole gateway to the cluster's persistent state.**

That's an excellent question that gets to the heart of how Kubernetes controllers operate! The entire flow, from `kubectl apply` to running Pods, involves a chain of control loops in the **Kubernetes Control Plane**.

Here is the step-by-step flow, focusing on the roles of the three key components: the **Deployment Controller**, the **ReplicaSet Controller**, and the **Kubelet**.

* * *

## 1\. The Deployment Controller's Role (The Orchestrator)

The **Deployment Controller**'s job is to ensure that a cluster's state matches the desired state defined in the **Deployment** object.

| **Step** | **Component** | **Action** | **Description** |
| --- | --- | --- | --- |
| 1   | **`kubectl`** | **CREATE/UPDATE Deployment** | The user runs `kubectl apply -f deployment.yaml`. The request goes to the **API Server**. |
| 2   | **API Server & etcd** | **STORE Deployment** | The API Server validates the object and persists it to **etcd**. |
| 3   | **Deployment Controller** | **OBSERVE Change** | The Controller Manager detects the new Deployment via its **watch** on the API Server. |
| 4   | **Deployment Controller** | **CREATE ReplicaSet** | It analyzes the Deployment's `spec.replicas` and `spec.template`. The controller then creates a new **ReplicaSet** object definition, including the Deployment's Pod template and a unique hash label (e.g., `pod-template-hash`) to distinguish it from other ReplicaSets. It sends this ReplicaSet object (via an API call, not to etcd directly) to the **API Server**. |

## 2\. The ReplicaSet Controller's Role (The Manager)

The **ReplicaSet Controller**'s job is to maintain the specified number of identical Pods for its associated ReplicaSet.

| **Step** | **Component** | **Action** | **Description** |
| --- | --- | --- | --- |
| 5   | **API Server & etcd** | **STORE ReplicaSet** | The API Server validates the ReplicaSet and persists it to **etcd**. |
| 6   | **ReplicaSet Controller** | **OBSERVE Change** | This controller, also watching the API Server, sees the new ReplicaSet object. |
| 7   | **ReplicaSet Controller** | **CREATE Pods** | It compares the desired replica count (e.g., 3) with the current count (0). Since they don't match, it generates the required number of **Pod** object definitions from the ReplicaSet's template and sends them to the **API Server**. |
| 8   | **API Server & etcd** | **STORE Pods** | The API Server stores the new Pod objects in **etcd**. |
| 9   | **Scheduler** | **BIND Pod to Node** | The **Scheduler** watches for new Pods that don't have a Node assigned. It selects an appropriate Node based on resource requests, constraints, etc., and sends a binding event (via API call) to the API Server, updating the Pod's specification with the assigned Node. |

The **Scheduler** does not interact with the Kubelet or the container runtime directly. Its sole function is to update the **Pod** object in the API Server with the correct **Node name**. This update is what tells the correct Kubelet to take action.

## 3\. The Kubelet's Role (The Worker)

The **Kubelet** runs on every Worker Node and is responsible for managing the Pods assigned to that Node.

| **Step** | **Component** | **Action** | **Description** |
| --- | --- | --- | --- |
| 10  | **Kubelet** | **OBSERVE Pod** | The Kubelet on the assigned Node (via a watch on the API Server) sees that a new Pod has been scheduled to its node. |
| 11  | **Kubelet** | **LAUNCH Container** | The Kubelet interacts with the **Container Runtime Interface (CRI)** (like containerd or CRI-O). It pulls the required container image (e.g., `nginx:latest`) and starts the containers within the Pod on the Node's operating system. |
| 12  | **Kubelet** | **UPDATE Pod Status** | As the containers start and the Pod goes into the `Running` state, the Kubelet constantly reports the Pod's current status back to the **API Server** (which then updates etcd). |
| 13  | **Controller Loops** | **RECONCILE Status** | The **ReplicaSet Controller** and **Deployment Controller** observe the Pods' status changes, see that the desired number of Pods are now running and ready, and consider their tasks complete (for now). |

* * *

### 🔑 The Golden Rule: API Server as the Single Gateway

The reason all this works without the Deployment or ReplicaSet Controllers accessing etcd directly is because the **API Server** acts as the *only* read/write interface for the cluster state. All controllers (and `kubectl`) use the API Server's REST interface to:

- **Read State:** Use the **Watch** mechanism to get real-time updates on objects they care about.
    
- **Write State:** Use standard HTTP methods (POST, PUT, DELETE) to create or modify objects.
    

&nbsp;

* * *

# **4️⃣ Visual Flow (simplified)**

```
kubectl apply → kube-apiserver → etcd
      ↓
 Deployment controller → ReplicaSet
      ↓
 Scheduler assigns pods → nodes
      ↓
 kubelet pulls image & starts pods
      ↓
 kube-proxy updates Service networking
      ↓
 Pod ready → Deployment status updated
```

* * *

# **5️⃣ TL;DR**

- **Control Plane = brain of the cluster**
    
    - kube-apiserver: talks to users/components
        
    - etcd: stores desired state
        
    - kube-scheduler: assigns pods to nodes
        
    - kube-controller-manager: keeps desired vs actual state in sync
        
    - cloud-controller-manager: manages cloud-specific resources
        
- **Applying Deployment flow**: API server → etcd → controller → scheduler → kubelet → kube-proxy → Pod ready.
    

* * *

&nbsp;

&nbsp;

&nbsp;

That's where the real power of the **Deployment Controller** comes in! When you make a change (like updating the container image version), Kubernetes automatically performs a **Rolling Update** to swap out the old version for the new one with **zero downtime** for your application.

Here is the step-by-step process of what happens when you modify a Deployment:

* * *

## 🔁 The Rolling Update Flow

When you run `kubectl apply` with a modified Deployment manifest (e.g., changing the `image: myapp:v1` to `image: myapp:v2`), the following reconciliation loop is triggered:

### 1\. The Trigger and New ReplicaSet Creation

1.  **API Server Update:** The change is sent to the **API Server** and persisted to **etcd**.
    
2.  **Deployment Controller Reacts:** The **Deployment Controller** detects the change in the Pod template (the difference between version 1 and version 2).
    
3.  **New ReplicaSet (RS) Created:** The Deployment Controller does **not** update the old **ReplicaSet**. Instead, it creates a **brand new ReplicaSet (RS-v2)** that contains the new Pod template (`myapp:v2`). The new ReplicaSet initially has **zero replicas**.
    
    - *The old ReplicaSet (RS-v1) still manages all the running Pods.*

### 2\. The Rollout Strategy (Scale Up/Scale Down)

The Deployment Controller now manages the scaling of both the new and old ReplicaSets simultaneously, based on the **RollingUpdate** strategy parameters, especially `maxSurge` and `maxUnavailable` (which default to 25%):

| **Parameter** | **Meaning** | **Default Value** |
| --- | --- | --- |
| **`maxSurge`** | Max number of Pods that can be **above** the desired replica count during the update. | 25% |
| **`maxUnavailable`** | Max number of Pods that can be **unavailable** (old or terminating) during the update. | 25% |

The update proceeds in increments, ensuring the application remains available:

- **Scale Up (New RS-v2):** The Deployment Controller tells the **new ReplicaSet (RS-v2)** to create a new Pod (`myapp:v2`).
    
- **Health Check:** The cluster waits for the new Pod to become **Ready** (signaled by its **Readiness Probe**).
    
- **Scale Down (Old RS-v1):** Once the new Pod is Ready, the Deployment Controller tells the **old ReplicaSet (RS-v1)** to terminate one of its old Pods (`myapp:v1`).
    
- **Repeat:** This cycle repeats until all old Pods are terminated and all new Pods are running and Ready.
    

### 3\. Final State and History

1.  **Old Pods Gone:** When the process is complete, the **old ReplicaSet (RS-v1)** is scaled down to **zero replicas**.
    
2.  **New Pods Running:** The **new ReplicaSet (RS-v2)** is managing the full desired replica count.
    
3.  **Revision History:** The Deployment maintains a history of its changes. This history is stored by keeping the old ReplicaSet (RS-v1) around at zero replicas.
    

* * *

## ↩️ The Power of Rollback

Because Kubernetes keeps the old ReplicaSet (RS-v1) around at zero replicas, you have an immediate "undo" button.

If the new version (`myapp:v2`) has a bug, you can instantly tell Kubernetes to revert:

Bash

```
kubectl rollout undo deployment/<deployment-name>
```

When you execute this:

1.  The Deployment Controller detects the rollback command.
    
2.  It scales the **broken RS-v2** down to zero.
    
3.  It scales the **stable RS-v1** back up to the desired replica count.
    

This process ensures that your application can recover quickly from bad updates without needing to rebuild or redeploy any code.

&nbsp;

&nbsp;

The **Deployment Controller** detects changes to a Deployment object using a sophisticated, efficient, and event-driven mechanism called **Informers**, which are built on top of the Kubernetes **Watch API**.

This entire process ensures that the controller doesn't have to constantly ask the API Server for the current state (polling), which would overload the system. Instead, the API Server pushes changes to the controller.

Here is the in-depth, step-by-step breakdown of how the controller knows about changes.

* * *

## 🧠 1. The Core Principle: The Control Loop

All Kubernetes controllers, including the Deployment Controller, operate on the **Control Loop** pattern:

1.  **Observe:** Read the **current state** of the cluster (via the API Server).
    
2.  **Compare:** Compare the current state against the **desired state** (defined in the Deployment's `.spec`).
    
3.  **Act:** If they don't match, take action (e.g., create a new ReplicaSet, scale an old one down) to move the current state closer to the desired state.
    

The key question is how the controller efficiently performs step 1, **Observe**, without constantly querying the API Server. This is where **Informers** come in.

* * *

## 📡 2. The Informer Architecture (The "Eyes" of the Controller)

The Deployment Controller is actually part of the **kube-controller-manager** process and uses a client-side component called a **Shared Informer** to monitor the Deployment resources. This mechanism involves three key components:

### A. The Reflector (The Watcher)

- **Initial List:** When the Deployment Controller starts, the Reflector first sends a **LIST** request to the **API Server** (e.g., `/apis/apps/v1/deployments`) to fetch *all* existing Deployment objects and their current state.
    
- **Persistent Watch:** The Reflector then immediately sends a **WATCH** request, subscribing to all future changes for Deployment objects. The API Server holds this connection open and streams real-time notifications (`ADDED`, `MODIFIED`, `DELETED`) whenever a change occurs in **etcd**.
    

### B. The Local Cache (The Memory)

- The Reflector continuously receives these events and stores a local, in-memory copy of the entire set of Deployment objects. This cache is kept up-to-date with every event received from the API Server.
    
- **Crucial Benefit:** When the Deployment Controller needs to read the current state of a Deployment, it reads directly from this **local cache**, dramatically reducing the load on the API Server and **etcd**.
    

### C. The WorkQueue (The Task List)

- The Shared Informer uses a component called a **WorkQueue** (or DeltaFIFO queue) to manage the detected changes.
    
- Whenever the local cache is updated due to a `MODIFIED` event (e.g., a change to the Deployment's image tag):
    
    1.  The Informer extracts the unique **key** of the changed object (typically `<namespace>/<name>`).
        
    2.  It pushes this key onto the **WorkQueue**.
        
    3.  The WorkQueue is smart: it **deduplicates** keys, so if the same Deployment is modified twice in rapid succession, the controller only needs to process it once.
        

* * *

## ⚙️ 3. The Controller's Reaction (The Action)

The final step is the reconciliation:

1.  **Worker Picks Up Key:** The Deployment Controller has multiple workers that continuously pull keys from the **WorkQueue**.
    
2.  **Fetch Latest Object:** When a worker picks up the key for the Deployment, it reads the latest version of that Deployment object from the **local cache**.
    
3.  **Reconciliation Logic:** The controller runs its core logic:
    
    - It checks the object's Pod Template Specification (`.spec.template`).
        
    - If the template has changed (e.g., the container image is different), it knows the desired state has diverged from the current state.
        
4.  **Take Action:** The controller performs the necessary API calls (through the API Server) to create a **new ReplicaSet** and begin the rolling update process to achieve the new desired state.
    

The core answer is: the **Deployment Controller** is immediately informed of changes by the **API Server** pushing events via the **Watch API**, which are efficiently processed and cached by the **Informer** mechanism.

&nbsp;

&nbsp;

&nbsp;

The **Watch API** and the **Informer Mechanism** are the fundamental pillars that allow the Kubernetes Control Plane to be event-driven and highly scalable. They work together to ensure that controllers and other components are immediately notified of state changes without overwhelming the API Server.<sup>1</sup>

* * *

## 👂 The Kubernetes Watch API (The "Ears")

The Watch API is the basic primitive provided by the **`kube-apiserver`** that enables real-time, one-way communication of resource changes.

### 1\. Definition

The Watch API allows a client (like the Controller Manager or Kubelet) to establish a **long-lived HTTP connection** with the API Server. The client specifies a resource type (e.g., Pods, Deployments) and a starting resource version.<sup>2</sup>

### 2\. How it Works

- **Initial Sync:** The client first sends a standard **GET** or **LIST** request to fetch the current state of all objects of a certain type.<sup>3</sup>
    
- **Persistent Connection:** The client then sends a **WATCH** request, which the API Server holds open.<sup>4</sup>
    
- **Event Streaming:** Whenever a change occurs to a watched object in **etcd** (the cluster's database), the API Server immediately streams an event over that persistent connection to the client. These events include:
    
    - `ADDED`: A new object was created.
        
    - `MODIFIED`: An existing object was updated.
        
    - `DELETED`: An object was removed.
        
- **Efficiency:** This is far more efficient than **polling** (repeatedly asking "Has anything changed?"), as the API Server only transmits data when an actual change happens.
    

* * *

## 🧠 The Informer Mechanism (The "Brain")

The Informer Mechanism is a client-side library (built into Kubernetes client-go) that consumes the events from the Watch API and provides a safe, efficient, and robust way for controllers to receive and process them.<sup>5</sup>

It solves the problems associated with raw watching, such as handling disconnections and minimizing the load on the API Server.<sup>6</sup>

### 1\. Key Components

An Informer consists of three main parts that work together:

| **Component** | **Role** | **Function** |
| --- | --- | --- |
| **Reflector** | **The Listener** | Establishes the watch connection, manages the initial LIST request, and automatically handles disconnections by starting a new watch from the last known resource version. |
| **Local Cache** (or **Index/Store**) | **The Memory** | A thread-safe, in-memory copy of all resource objects watched by the Informer. The Reflector updates this cache with every event received. |
| **WorkQueue** (or **DeltaFIFO**) | **The Task List** | A queue that receives the keys of objects that have changed. Controllers read from this queue to know which object needs reconciliation. It automatically **deduplicates** changes to prevent the controller from processing the same object multiple times in rapid succession. |

### 2\. How the Informer Mechanism Works

1.  The **Reflector** receives an event from the **API Server** via the Watch API.
    
2.  The Reflector immediately updates the **Local Cache** with the object's new state.
    
3.  The Reflector puts the object's unique key (`<namespace>/<name>`) into the **WorkQueue**.
    
4.  The **Controller** (e.g., the Deployment Controller) is notified that a new item is in the WorkQueue.
    
5.  The Controller's worker thread pulls the key from the WorkQueue.
    
6.  The Controller reads the object's latest state **directly from the Local Cache**, avoiding a costly API call to the API Server.
    
7.  The Controller performs its reconciliation logic (Compare desired state vs. current state) and acts if necessary.
    

## 🤝 Summary: Why They are Essential

| **Feature** | **Watch API** | **Informer Mechanism** |
| --- | --- | --- |
| **Function** | Pushes raw change events from the API Server. | Consumes events, maintains local state, and queues work for the controller. |
| **Location** | Part of the **API Server** (Server-side primitive). | Part of the **Controller** (Client-side library). |
| **Key Benefit** | Eliminates polling, making communication real-time. | Minimizes API Server load by using a local cache and handles event reliability.<sup>7</sup> |

In short, the **Watch API** gives the controllers a way to listen for changes, and the **Informer Mechanism** gives them the robust tooling to handle those changes efficiently and reliably.

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

That independence is achieved through a specific design pattern within the **Shared Informer** architecture known as **registration** and **event handling**.

Here's a breakdown of how the single event streams from the **Local Cache** are distributed into multiple, independent **WorkQueues**:

* * *

## 📦 How Independent WorkQueues are Maintained

The mechanism ensures that while all controllers read from the same source of truth (the single **Local Cache**), their reconciliation work is completely isolated.

### 1\. Controller Registration (The Subscription)

When a controller (like the Deployment Controller or the HPA Controller) starts up and decides it needs to watch a specific resource (e.g., **ReplicaSets**), it doesn't just ask for the Shared Informer; it **registers an event handler** with the Shared Informer.

- **Registration API:** The controller calls a method like `AddEventHandler` on the Shared Informer object.
    
- **The Handler:** This handler is a set of callback functions specific to the controller's needs (e.g., `OnAdd`, `OnUpdate`, `OnDelete`).
    
- **The Filter:** The controller defines its own logic within these handlers. For instance, the Deployment Controller only cares about ReplicaSets that are owned by a Deployment.
    

### 2\. The Fan-Out Mechanism

The **Shared Informer** manages a master list of all these registered handlers. When the **Reflector** updates the **Local Cache** (because it received a new event from the API Server), the Shared Informer performs a **fan-out** operation:

- **Event Receipt:** A single event (e.g., a `MODIFIED` event for `ReplicaSet-X`) arrives and updates the single **Local Cache**.
    
- **Sequential Dispatch:** The Shared Informer iterates through **every single registered handler** (one for the Deployment Controller, one for the HPA Controller, etc.).
    
- **Handler Execution:** For each handler, the Shared Informer calls the appropriate function (e.g., `OnUpdate`) and passes the changed object (ReplicaSet-X).
    

### 3\. Isolation by WorkQueue

This is the most critical step. The responsibility of putting the object into the **WorkQueue** belongs to the **controller's registered handler function**, not the Shared Informer itself.

| **Controller** | **Handler Logic (Simplified)** | **WorkQueue Entry** |
| --- | --- | --- |
| **Deployment Controller** | `OnUpdate(new_rs)`: Check if `new_rs` needs rolling out. If so, add `new_rs`'s key to the **Deployment Controller's WorkQueue**. | **Independent WorkQueue A** |
| **HPA Controller** | `OnUpdate(new_rs)`: Check if the pods managed by `new_rs` need scaling. If so, add `new_rs`'s key to the **HPA Controller's WorkQueue**. | **Independent WorkQueue B** |

Because each controller maintains its own independent WorkQueue, the failure or backlog of one controller's work (e.g., the Deployment Controller is busy with a massive rollout) **does not** prevent other controllers (like the HPA Controller) from receiving and acting on the same event.

This separation of the **data source (Shared Informer)** from the **processing queue (WorkQueue)** is what allows all controllers to act independently using the same cluster state.

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

That's an excellent question that clarifies a crucial point about the **Informer Mechanism**!

The **Deployment Controller does not call the API Server for the current state** when the local cache is updated. This is the whole purpose and efficiency gain of the Informer architecture.

Here is the in-depth explanation of how the comparison and reconciliation happen:

* * *

## 🧐 How the Controller Compares States

The comparison happens entirely using the **local, in-memory cache** and the object key pulled from the **WorkQueue**.

### 1\. The Controller Reads the Desired State

When an item's key (`<namespace>/<deployment-name>`) is popped off the **WorkQueue**, the controller's worker thread does the following:

- **Read Deployment Spec:** It accesses the **Local Cache** and retrieves the full **Deployment** object associated with that key. The specification (`.spec`) of this object represents the **Desired State** (e.g., "I want 3 replicas and image `myapp:v2`"). Since the local cache was just updated by the **Reflector** receiving the event from the API Server, this is the freshest desired state.

### 2\. The Controller Determines the Actual State

The **actual state** is not just the Deployment object itself, but the status of the related child resources (the ReplicaSets and Pods) that the Deployment manages.

The Deployment Controller doesn't need to call the API Server to get these either, because it uses **separate, shared Informers** for the child resources:

- **ReplicaSet Informer:** The Deployment Controller maintains a separate **Informer and Local Cache** for all **ReplicaSet** objects in the cluster.
    
- **Pod Informer:** Similarly, it has a separate **Informer and Local Cache** for all **Pod** objects.
    

To determine the **Actual State**, the Deployment Controller performs an **internal lookup** on its *local* ReplicaSet and Pod caches, using **labels and owner references**:

- **Find Child ReplicaSets:** It queries its local ReplicaSet cache for all ReplicaSets that have an **Owner Reference** pointing back to the current Deployment object.
    
- **Find Child Pods:** It then queries its local Pod cache for all Pods owned by those identified ReplicaSets.
    
- **Count Status:** It sums up the running, ready, and failed counts for those Pods to know the cluster's **current state** (e.g., "I currently have 3 Pods running, but they belong to the old ReplicaSet `RS-v1`").
    

### 3\. The Reconciliation (Compare and Act)

Now the comparison is straightforward and efficient, as it requires **zero external API calls**:

| **State** | **Source** | **Example** |
| --- | --- | --- |
| **Desired State** | Deployment object from **Local Cache (Deployment)** | Pods should use `myapp:v2`. |
| **Actual State** | ReplicaSet/Pod objects from **Local Cache (RS/Pod)** | All running Pods currently use `myapp:v1`. |

Since the **Desired State** (`v2`) does not match the **Actual State** (`v1`), the Deployment Controller takes action: it constructs and sends an API request to the API Server (e.g., POST a new ReplicaSet object), kicking off the next stage of the rollout.

* * *

## 💡 The Efficiency Gain

The **Informer Mechanism** is built precisely to avoid Step 1 in your initial question ("then it call apiserver for current state?").

By reading from the **Local Cache** instead of making a direct API call, the controller achieves the following:

1.  **Low Latency:** Reading from local memory is nearly instantaneous.
    
2.  **High Throughput:** The controller can process hundreds of changes per second without waiting for network round trips.
    
3.  **Low Load on API Server:** The API Server is protected from being overwhelmed by controllers constantly querying for state. The Watch API handles the heavy lifting of state distribution efficiently.
    

&nbsp;

&nbsp;

&nbsp;

&nbsp;

The Kubernetes Control Plane uses many specialized controllers beyond the Deployment, ReplicaSet, and Cloud Controller Manager. Each one is responsible for a specific resource type and maintains the desired state for that resource.

Here are some of the other crucial controllers running within the **`kube-controller-manager`** and elsewhere:

* * *

## ⚙️ Controllers in the `kube-controller-manager`

These controllers run as a single process and manage the core logic for common built-in resources:

### 1\. Node Controller 🌐

- **Role:** Responsible for noticing and responding when nodes become unavailable or get deleted.
    
- **Action:**
    
    - Monitors the health and reachability of all **Nodes**.
        
    - Updates a Node's status (e.g., from `Ready` to `NotReady`) when it detects unresponsiveness.
        
    - If a Node remains unreachable for a configured time, it safely deletes the Pods bound to that Node so they can be rescheduled by the **Scheduler** onto a healthy Node.
        

### 2\. Service Controller 🔗

- **Role:** Manages the lifecycle of **Services**, specifically **LoadBalancer** and **NodePort** types. (Note: This is distinct from the **Cloud Controller Manager's Service Controller**.)
    
- **Action:**
    
    - For **Type=NodePort** services, it ensures the firewall rules on the Nodes are configured correctly to expose the service port.
        
    - It ensures the necessary virtual IPs (Cluster IPs) are allocated and managed.
        

### 3\. Endpoints Controller 🎯

- **Role:** Bridges the gap between a **Service** and the set of healthy **Pods** backing it.
    
- **Action:**
    
    - Constantly watches for new, updated, or deleted **Pods** that match a Service's selector.
        
    - Automatically creates and updates the **Endpoints** object, which is essentially a list of IP addresses and ports for the healthy, ready Pods that belong to a Service. The `kube-proxy` uses this list to route traffic.
        

### 4\. Job Controller ⏳

- **Role:** Manages **Job** objects, which represent batch workloads that should run once to completion.
    
- **Action:**
    
    - Creates one or more **Pods** corresponding to the Job spec.
        
    - Monitors the Pods and tracks the completion status.
        
    - If the Job is successful, the controller marks the Job as complete. If a Pod fails, it may restart it depending on the Job's restart policy.
        

### 5\. PersistentVolume (PV) Controller 💾

- **Role:** Manages the lifecycle of **Persistent Volumes (PVs)** and **Persistent Volume Claims (PVCs)**, which handle storage in Kubernetes.
    
- **Action:**
    
    - The **Volume Controller** handles the binding of a PVC to an appropriate PV.
        
    - The **Volume Attachment Controller** handles attaching the underlying storage volume to the correct Node when a Pod needs it.
        

* * *

## ☁️ Other Specialized Controllers

These controllers often run outside the main `kube-controller-manager` process, sometimes as separate components or add-ons:

### 1\. Kube-Proxy (Not a traditional controller, but vital) 🔄

While technically a network agent that watches for Services and Endpoints changes, it's often thought of alongside controllers.

- **Role:** Manages network rules on each Node to enable communication to Pods via Services.
    
- **Action:**
    
    - Watches the **Endpoints Controller**'s updates.
        
    - Programs network rules (usually using **iptables** or **IPVS**) on the Node to ensure that traffic directed to a Service's Cluster IP is correctly forwarded to the back-end Pod IPs.
        

### 2\. Horizontal Pod Autoscaler (HPA) Controller 📈

- **Role:** Automatically scales the number of Pod replicas (managed by Deployments or ReplicaSets) up or down based on observed CPU utilization or other custom metrics.
    
- **Action:**
    
    - Watches the utilization of Pods.
        
    - Compares the current metric against the target metric defined in the **HPA** object.
        
    - If utilization is too high, it sends an API request to the Deployment/ReplicaSet to **increase** the replica count. If too low, it **decreases** the count.
        

### 3\. Custom Resource Definition (CRD) Controllers 🧱

- **Role:** These are user-defined controllers that extend Kubernetes functionality to manage **Custom Resources**. They form the core of the Operator pattern.
    
- **Example:** A controller for a resource called `Database` would watch for new `Database` objects and then call external cloud APIs (or Helm charts) to provision an actual database instance.
    

Would you like a deeper dive into how the **HPA Controller** monitors metrics and decides when to scale?

&nbsp;

&nbsp;

&nbsp;

That's a great idea! Seeing the components in action on a local cluster like **Minikube** is the best way to solidify your understanding.

Here are the commands you can use in your terminal to inspect the Control Plane components and see their working flow.

* * *

## 1\. 🔍 Inspecting Control Plane Components (Static Pods)

In Minikube, the Control Plane components (`kube-apiserver`, `etcd`, `kube-scheduler`, and `kube-controller-manager`) run as **static Pods** on the single master node. You can view them by looking at the Pods in the `kube-system` namespace.

### Command

Bash

```
kubectl get pods --namespace kube-system
```

### What You'll See

You'll see Pods for the core components, typically labeled with their function:

- `kube-apiserver-minikube` (or similar name)
    
- `etcd-minikube`
    
- `kube-scheduler-minikube`
    
- `kube-controller-manager-minikube`
    

* * *

## 2\. 🩺 Checking Component Health

You can ask the API Server for the reported health status of the key components that are part of the Control Plane's self-check.

### Command

Bash

```
kubectl get componentstatuses
```

### What You'll See

This will show whether the **scheduler**, **controller-manager**, and **etcd** are reporting as `Healthy`.

* * *

## 3\. 👂 Seeing the Controllers "Watch" (Live Logs)

The most insightful command is watching the **Controller Manager's** logs, as this is where the Deployment Controller and ReplicaSet Controller live.

1.  **Get the Controller Manager Pod Name:**
    
    Bash
    
    ```
    CONTROLLER_POD=$(kubectl get pods -n kube-system -l component=kube-controller-manager -o jsonpath='{.items[0].metadata.name}')
    ```
    
2.  **Stream the Logs:**
    
    Bash
    
    ```
    kubectl logs $CONTROLLER_POD -n kube-system -f
    ```
    

### What You'll See

Keep this terminal window open. You will see logs scroll by as the controller is running its loops. Now, move to the next step.

* * *

## 4\. 🚀 Seeing the Deployment Flow in Action

Open two *new* terminal windows for this part. You will create a Deployment and see the chain of events unfold across all three terminals.

### Terminal A (Controller Logs)

- Keep the logs streaming from **Step 3**.

### Terminal B (Watch ReplicaSets)

- This terminal will show the **Deployment Controller's** output—the creation of the ReplicaSet.
    
    Bash
    
    ```
    kubectl get rs --watch
    ```
    

### Terminal C (Watch Pods)

- This terminal will show the **ReplicaSet Controller's** output—the creation of the Pods.
    
    Bash
    
    ```
    kubectl get pods --watch
    ```
    

### 5\. Execute the Deployment

Now, in a **fourth** terminal (or the same one used for watching Pods once you stop the watch), run this command to create a Deployment:

Bash

```
kubectl create deployment my-app --image=nginx:latest --replicas=2
```

### What You'll See (The Flow)

1.  **Terminal B (ReplicaSets):** You will see a new ReplicaSet object instantly appear (e.g., `my-app-76d97c7674`). This is the **Deployment Controller's** work.
    
2.  **Terminal C (Pods):** You will see two new Pods appear with **STATUS** as `Pending`.
    
3.  **Terminal A (Controller Logs):** You will see entries confirming that the Deployment Controller received the `ADDED` event for the Deployment and the ReplicaSet Controller is now trying to reconcile the desired replica count.
    
4.  **Terminal C (Pods) - Scheduler Action:** The Pods will quickly transition from `Pending` to `ContainerCreating` after the **Scheduler** finds a Node (in Minikube, this is the single Minikube node) and binds the Pod to it.
    

* * *

## 6\. 📝 Confirming the Scheduler's Decision

After the Pods are running, you can look at the historical events logged on the Pod to confirm which component did what.

1.  **Get a Pod Name:**
    
    Bash
    
    ```
    POD_NAME=$(kubectl get pods -l app=my-app -o jsonpath='{.items[0].metadata.name}')
    ```
    
2.  **Describe the Pod:**
    
    Bash
    
    ```
    kubectl describe pod $POD_NAME
    ```
    

### What You'll See in the Description

Look at the bottom of the output in the **Events** section:

- You will see an event logged by the **`default-scheduler`** with the message: `Successfully assigned default/my-app-... to minikube`. This confirms the scheduler's role.
    
- You will see subsequent events logged by the **`kubelet`** (or the Node) showing `Pulling image`, `Created container`, and `Started container`. This confirms the Kubelet's role as the worker.
    

&nbsp;

&nbsp;

&nbsp;

Great — **Minikube is the best way to *see* every control-plane component working** because it's your own local cluster, and you can inspect everything.

I’ll show you **exact commands** to view:

✔ kube-apiserver  
✔ etcd  
✔ scheduler  
✔ controller-manager  
✔ kubelet  
✔ kube-proxy  
✔ logs, configs, certificates  
✔ how to test their live working

All commands work exactly on Minikube.

* * *

# ✅ **1\. See All Control Plane Components (pods)**

In Minikube, all control-plane components run inside the VM/container.

Run:

```bash
kubectl -n kube-system get pods -o wide
```

You will see something like:

```
NAME                                 READY   STATUS
kube-apiserver-minikube             1/1     Running
kube-controller-manager-minikube    1/1     Running
kube-scheduler-minikube             1/1     Running
etcd-minikube                        1/1     Running
coredns-xxxxx                        1/1     Running
kube-proxy-xxxxx                     1/1     Running
```

* * *

# ✅ **2\. See Logs of Each Control Plane Component**

### **API Server Logs**

```bash
kubectl -n kube-system logs kube-apiserver-minikube
```

### **etcd Logs**

```bash
kubectl -n kube-system logs etcd-minikube
```

### **Scheduler Logs**

```bash
kubectl -n kube-system logs kube-scheduler-minikube
```

### **Controller Manager Logs**

```bash
kubectl -n kube-system logs kube-controller-manager-minikube
```

This shows you **real-time scheduling**, **pod creation**, **controller actions**, etc.

* * *

# ✅ **3\. See kubelet (runs ON node, not inside pod)**

kubelet is NOT a pod; it's a systemd process on the Minikube node.

First, SSH into the minikube node:

```bash
minikube ssh
```

Then:

### **Check kubelet service**

```bash
systemctl status kubelet
```

### **View kubelet logs**

```bash
journalctl -u kubelet -f
```

You’ll see:

- image pulling
    
- pod sandboxes created
    
- container started events
    
- probe failures
    

* * *

# ✅ **4\. See kube-proxy rules (iptables)**

kube-proxy **runs as a pod**, so:

### **View logs:**

```bash
kubectl -n kube-system logs -l k8s-app=kube-proxy
```

### **Check iptables rules added by kube-proxy**

SSH into node:

```bash
minikube ssh
```

Then:

```bash
sudo iptables-save | grep KUBE
```

You will see chains like:

```
-KUBE-SERVICES
-KUBE-NODEPORTS
-KUBE-PORTALS
-KUBE-MARK-MASQ
```

These are the **actual rules that route Service → Pod** traffic.

* * *

# ✅ **5\. See etcd Data / Cluster State**

You can inspect the **actual cluster data** stored in etcd.

SSH into minikube:

```bash
minikube ssh
```

Find etcd pod data directory:

```bash
sudo ls /var/lib/minikube/etcd
```

To query etcd:

Inside minikube, exec into etcd container:

```bash
kubectl -n kube-system exec -it etcd-minikube -- sh
```

Then view keys:

```bash
ETCDCTL_API=3 etcdctl get / --prefix --keys-only
```

You will see:

- `/registry/pods/...`
    
- `/registry/services/...`
    
- `/registry/deployments/...`
    

This is the **raw cluster state**.

* * *

# ✅ **6\. See Scheduler Decisions**

Watch how scheduler assigns pods:

```bash
kubectl -n kube-system logs kube-scheduler-minikube | grep "Scheduling"
```

Or live:

```bash
kubectl get events -w
```

You’ll see events like:

```
Scheduled pod nginx-deployment-5c5cbbcfd7-xrrmj to minikube
```

* * *

# ✅ **7\. See Controller Manager Activity**

```bash
kubectl -n kube-system logs kube-controller-manager-minikube
```

You will see:

- ReplicaSet created
    
- Pods scaled up
    
- Nodes marked ready
    
- Endpoints updated
    

* * *

# ✅ **8\. See Control Plane Configuration Files**

SSH into Minikube:

```bash
minikube ssh
```

Then:

### **API Server config & certs**

```bash
ls /etc/kubernetes/manifests/
```

You will find:

```
kube-apiserver.yaml
kube-scheduler.yaml
kube-controller-manager.yaml
etcd.yaml
```

These are static Pod manifests.

To view:

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

You will see:

- ports
    
- certificates
    
- admission plugins
    
- feature gates
    

* * *

# 🚀 **9\. See Everything Working Live: Deployment Flow**

Open 3 terminals:

* * *

### **Terminal 1 — Watch Pods**

```bash
kubectl get pods -A -w
```

* * *

### **Terminal 2 — Watch Events**

```bash
kubectl get events -w
```

* * *

### **Terminal 3 — Watch Control Plane Logs**

#### Scheduler

```bash
kubectl -n kube-system logs -f kube-scheduler-minikube
```

#### Controller Manager

```bash
kubectl -n kube-system logs -f kube-controller-manager-minikube
```

#### API Server

```bash
kubectl -n kube-system logs -f kube-apiserver-minikube
```

* * *

# 🧪 **10\. Test by applying deployment**

```bash
kubectl create deployment demo --image=nginx
kubectl scale deployment demo --replicas=3
```

Now watch:

- API server receives request
    
- Deployment controller creates ReplicaSet
    
- Scheduler assigns pods
    
- kubelet pulls image
    
- kube-proxy updates rules
    
- Pods go Ready
    

**You can literally see the entire flow live.**

* * *

# 🎯 **If you want, I can also give:**

✔ A guided lab: “Life of a Pod in Minikube (Full Hands-on)”  
✔ Commands to visualize API calls in real time  
✔ How to break control-plane components on purpose for learning  
✔ How to inspect CNI networking

* * *

If you want, I can prepare a **full lab script** where you run 20+ commands and see everything step-by-step.

&nbsp;

&nbsp;

&nbsp;

&nbsp;

This is **expected behavior**, and you did NOT do anything wrong.

A lot of people get confused when they watch all control-plane logs and expect *every component* to print something every time a Deployment is created.

Let me explain **why only controller-manager and events showed logs**, and **why API server / scheduler / etcd did NOT show anything**, even though they DID work.

* * *

# ✅ **1\. Why kube-apiserver shows no logs (unless there is an error)**

### 🔹 kube-apiserver does NOT log every API call

Every action in Kubernetes goes through the apiserver, but **it does NOT print logs for normal requests**.

By default, API server logs only:

- errors
    
- warnings
    
- authentication issues
    
- audit logs (if enabled, minikube disables these)
    
- admission failures
    

So creating a Deployment:

```bash
kubectl create deployment demo --image=nginx
```

goes through apiserver, but **apiserver does not log normal "create" requests**.

To see API calls, you must enable **audit logs**, which Minikube does NOT enable unless configured.

This is why you saw **nothing** in API server logs.

* * *

# ✅ **2\. Why scheduler shows nothing (for deployments)**

Scheduler logs only when it **schedules PODS**, not when you create a Deployment.

What you did:

- created a Deployment (not a pod)
    
- the scheduler does not schedule Deployments, it schedules Pods
    

You will only see logs like:

```
"Successfully bound pod demo-xxxxxx to minikube"
```

ONLY when:

1.  Pods are created
    
2.  Pods need scheduling
    

But in Minikube **there is only 1 node**, so scheduling is trivial and sometimes logs are very minimal:

- one or two lines
    
- sometimes suppressed unless verbose logging enabled
    

So **it's normal not to see scheduler logs** unless pods are pending or being placed.

* * *

# ✅ **3\. Why etcd shows nothing**

etcd logs **only when:**

- it crashes
    
- it compacts data
    
- it snapshots
    
- there is a write error
    
- it's under heavy load
    

etcd **does NOT log every key write** (it would be massive).

When you create a Deployment, etcd does store data like:

```
/registry/deployments/<namespace>/<name>
```

but it does NOT print any logs for that.

**Normal writes = no logs.**

This is normal.

* * *

# ✅ **4\. Why controller-manager showed logs**

This is correct.

Controller-manager logs things like:

- deployment → replicaset creation
    
- replicaset → pod creation
    
- scaling
    
- updating status
    

Example logs:

```
Created replicaSet demo-76cbb57b68
```

This is exactly what you observed.

* * *

# ✅ **5\. Why events showed logs**

Events are generated by:

- scheduler
    
- controller manager
    
- kubelet
    

Example:

```
ScalingReplicaSet demo-xxxxx to 3
Scheduled demo-xxxxx to minikube
Pulled image nginx
Created container
```

These events appear because they are explicitly emitted.

* * *

# 🧠 **Summary Table — Which Component Logs What?**

| Component | Should you see logs when creating Deployment? | Why |
| --- | --- | --- |
| **kube-apiserver** | ❌ No | Does not log normal API calls |
| **etcd** | ❌ No | Logs only errors/snapshots, not writes |
| **scheduler** | ⚠️ Maybe (very little) | Only logs Pod scheduling, minimal on single-node cluster |
| **controller-manager** | ✅ Yes | Creates RS + Pods, always logs |
| **events** | ✅ Yes | Always logs scaling + scheduling events |

* * *

# 🧪 Want to FORCE logs to appear?

## 1️⃣ Force scheduler logs

Create a pod with **nodeSelector that matches nothing**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: fail-pod
spec:
  nodeSelector:
    fake-label: no-such-node
  containers:
  - name: nginx
    image: nginx
```

Apply:

```bash
kubectl apply -f fail-pod.yaml
```

Now scheduler logs will appear:

```
FailedScheduling: 0/1 nodes are available: node(s) didn't match node selector
```

* * *

## 2️⃣ Force API server logs

Send an **invalid** request:

```bash
kubectl get pods --namespace fake
```

API server logs:

```
namespace "fake" not found
```

* * *

## 3️⃣ Force etcd logs

Cause high writes:

```bash
for i in {1..5000}; do kubectl create configmap cm$i --from-literal=a=b; done
```

etcd logs will show compaction/snapshot activity.

* * *

# 🎉 If you want, I can give you a complete:

### ✔ "Life of a Deployment" visual lab

### ✔ "How to see every API request" (enable audit logging)

### ✔ "How to see raw etcd keys"

### ✔ "How to trace a pod creation step-by-step while watching all logs"

Just tell me.

&nbsp;

&nbsp;

&nbsp;