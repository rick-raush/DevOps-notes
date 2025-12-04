* * *

# ⭐ Summary

| Concept | Meaning |
| --- | --- |
| **Service** | A stable virtual endpoint (ClusterIP) for accessing pods |
| **Endpoints** | Actual pod IPs behind Service |
| **kube-proxy** | Programs routing rules to redirect Service IP → Pod IP |
| **iptables** | Used to implement NAT/load balancing for Services |
| **Proxy Path** | ClusterIP → iptables → PodIP |
| **Logs** | `kubectl logs <pod>` |

&nbsp;

&nbsp;

### **How to view logs of pods**

* * *

### **Basic logs command**

```bash
kubectl logs <pod-name>
```

### Logs from specific container in a multi-container pod

```bash
kubectl logs <pod-name> -c <container-name>
```

### Follow logs (like tail -f)

```bash
kubectl logs -f <pod-name>
```

### View logs for previous crashed instance

```bash
kubectl logs --previous <pod-name>
```

### Logs across all pods (using labels)

```bash
kubectl logs -l app=myapp --all-containers=true
```

* * *

&nbsp;

&nbsp;

&nbsp;

&nbsp;

Your Pod  
   |  
   | DNS Query: auth-service.default.svc.cluster.local  
   v  
/etc/resolv.conf  
   |  
   | Forward to nameserver 10.96.0.10  
   v  
CoreDNS  
   |  
   | Query Kubernetes API  
   | Reads Service ClusterIP = 10.102.45.12  
   v  
Returns 10.102.45.12 to your Pod

&nbsp;

Yes — **the Kubernetes API Server is the single source of truth for all Service and Endpoint mappings.**

Let’s break this down clearly.

* * *

# ✅ **Does the API Server have the actual Service → Pod IP mapping?**

### ✔ YES

The API Server stores:

### **1\. Service objects**

This includes:

- Service name
    
- Namespace
    
- **ClusterIP**
    
- Ports
    
- Selectors
    

Example (from APIServer):

```yaml
spec:
  clusterIP: 10.102.45.12
  selector:
    app: auth-service
```

* * *

### **2\. Endpoints / EndpointSlice objects**

This contains:

- Pod IPs that match the Service selector
    
- Ports for each pod
    

Example:

```yaml
addresses:
  - ip: 10.10.12.45
  - ip: 10.10.12.46
```

* * *

# 🔥 **So the API Server has EVERYTHING for resolution**

| Component | Reads from API Server | What it does |
| --- | --- | --- |
| **CoreDNS** | YES | Builds DNS records from services/endpoints |
| **kube-proxy** | YES | Builds iptables/IPVS rules from services/endpoints |
| **kubectl get svc / get endpoints** | YES | Directly queries API Server |

Both the DNS system **and** the in-cluster networking rules come from the API Server store (etcd).

* * *

# 🧠 Flow: How mapping works

### 1️⃣ Service created

You run:

```bash
kubectl create svc ...
```

API Server stores:

```
Service:
  auth-service → 10.102.45.12 (ClusterIP)
```

### 2️⃣ EndpointSlice updates automatically

Whenever Pods matching selector run:

```
auth-service → {10.10.12.45, 10.10.12.46}
```

API Server stores this too.

### 3️⃣ CoreDNS watches API Server

CoreDNS gets these automatically through **watch API**:

```
auth-service.default.svc → 10.102.45.12
```

### 4️⃣ kube-proxy watches API Server

kube-proxy creates iptables rules:

```
ServiceIP 10.102.45.12 → PodIPs 10.10.12.45, 10.10.12.46
```

* * *

# 🧩 **So who "maps" the DNS name?**

| Component | Maps DNS name? | Uses mapping? |
| --- | --- | --- |
| **CoreDNS** | YES – resolves DNS name → ClusterIP | Uses API Server data |
| **kube-proxy** | NO  | Maps ClusterIP → Pod IP via NAT |
| **API Server** | Stores mapping | Does not resolve DNS directly |

* * *

# 🟦 **Summary**

✔ API Server contains the real mapping of:  
**Service → ClusterIP**  
**Service → Pod IPs (Endpoints/EndpointSlices)**

✔ CoreDNS reads this to create DNS mappings  
✔ kube-proxy reads this to create IP routing rules

APIServer = **source of truth.**  
CoreDNS / kube-proxy = **consumers.**

* * *

&nbsp;

&nbsp;

&nbsp;

&nbsp;

Let’s break this down very clearly and from basics — **Endpoints** and **EndpointSlices** are fundamental Kubernetes objects used to map **a Service → the actual Pod IPs** that implement it.

* * *

# 🟦 **1\. What Are Endpoints? (Older mechanism)**

An **Endpoint** object stores the list of **Pod IPs** that belong to a Service.

For a Service named `auth-service`, Kubernetes automatically creates an Endpoints object:

```bash
kubectl -n raushan get endpoints nginx-service-nodeport -o yaml
```

Example output:

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  creationTimestamp: "2025-11-20T20:03:40Z"
  name: nginx-service-nodeport
  namespace: raushan
  resourceVersion: "11148354"
  uid: a79874d8-2789-4468-a08f-10c578d98150
subsets:
- addresses:
  - ip: 10.244.1.28
    nodeName: oauth-test-worker2
    targetRef:
      kind: Pod
      name: nginx-deployment-ddd55bc54-hdj4d
      namespace: raushan
      uid: fbce0c59-1b54-47ed-aa03-976ea01638a8
  - ip: 10.244.2.31
    nodeName: oauth-test-worker
    targetRef:
      kind: Pod
      name: nginx-deployment-ddd55bc54-dqvns
      namespace: raushan
      uid: d35af4e6-f126-43aa-ab8d-1c723d486a73
  ports:
  - port: 443
    protocol: TCP
```

### **Meaning:**

- The Service `nginx-service-nodeport`has **2 backend pods**
    
    - Pod IPs: `10.10.12.45`, `10.10.12.46`
- kube-proxy uses this list to do load-balancing
    
- CoreDNS uses Service name → ClusterIP; kube-proxy uses ClusterIP → Pod IPs
    

* * *

# 🟩 **2\. What Are EndpointSlices? (Newer mechanism)**

Introduced in Kubernetes 1.16+  
Replaces Endpoints for scalability.

### Why needed?

- Old Endpoints object could get **too large**
    
- 1 big list containing **all IPs**, up to 1000s of pods
    
- Frequent updates → API Server load
    

### EndpointSlices solve this.

* * *

# 🟩 **EndpointSlice = Sharded Endpoints**

Each slice stores only **up to 100 endpoints**.

Example:

```bash
kubectl get endpointslice -n default | grep auth-service
```

Output:

```
auth-service-abcde
auth-service-fghij
```

Each contains a subset:

```yaml
endpoints:
  - addresses: ["10.10.12.45"]
  - addresses: ["10.10.12.46"]
ports:
  - port: 8080
```

### Advantages:

| Endpoints | EndpointSlices |
| --- | --- |
| Single large object | Multiple small objects |
| Updated entirely | Only slices that changed are updated |
| Poor scaling | Designed for large-scale clusters |
| Max ~1000 endpoints | Scales to 100k+ pods |

* * *

# 🧠 **3\. Relationship Between Service, Endpoints, EndpointSlices**

### When you create a Service:

1.  Kubernetes looks for pods based on selector:
    
    ```
    selector:
      app: auth-service
    ```
    
2.  It creates:
    
    - **Endpoints** (legacy)
        
    - **EndpointSlices** (modern default)
        
3.  kube-proxy reads EndpointSlices and programs iptables/ipvs.
    

* * *

# 🟦 **4\. How Kubernetes Decides Which One to Use?**

👉 **EndpointSlices are default and primary**  
👉 The old Endpoints object is still created for backward compatibility  
👉 kube-proxy uses EndpointSlices (newer and more efficient)

* * *

# 🟦 **5\. How to View Both**

### **Endpoints**

```bash
kubectl get endpoints <svc-name> -n <ns> -o wide
```

### **EndpointSlices**

```bash
kubectl get endpointslice -n <ns> --selector=kubernetes.io/service-name=<svc-name>
```

* * *

# 🟦 **6\. Which One Does CoreDNS Use?**

CoreDNS uses the **Service → ClusterIP** mapping only.  
It does not care about Pods.

**CoreDNS → ClusterIP**  
**kube-proxy → Pod IPs (from EndpointSlices)**

* * *

# 🟠 SUMMARY — Simple Explanation

### **Endpoints (old)**

A single object listing all Pod IPs behind a Service.

### **EndpointSlices (new)**

Multiple small objects, each containing ~100 Pod IPs.

### Why?

- Faster
    
- More scalable
    
- Less API load
    

### Who uses them?

- kube-proxy uses EndpointSlices
    
- Service discovery uses both for backward compatibility
    

* * *

&nbsp;

&nbsp;

`--selector=kubernetes.io/service-name=<svc-name>` is a **label selector** used to filter **EndpointSlice** resources that belong to a specific Service.

Let’s break it down **very clearly**.

* * *

# ✅ **1\. EndpointSlices have labels**

Every EndpointSlice object automatically gets labels added by Kubernetes, including:

```yaml
labels:
  kubernetes.io/service-name: auth-service
```

This label tells Kubernetes:

**“This EndpointSlice belongs to Service = auth-service”**

* * *

# ✅ **2\. What does the selector do?**

When you run:

```bash
kubectl get endpointslice -n default --selector=kubernetes.io/service-name=auth-service
```

You are telling Kubernetes:

> “Show me only those EndpointSlices where the label  
> `kubernetes.io/service-name` is equal to `auth-service`.”

It is exactly like filtering pods:

```bash
kubectl get pods -l app=myapp
```

But here we are filtering EndpointSlices.

* * *

# 🔍 **3\. Why does Kubernetes add this label?**

Because many Service → EndpointSlice mappings can exist.

Example:

A Service may have:

- 1 EndpointSlice
    
- or 5 EndpointSlices (large deployment with 500 pods)
    
- or 20+ slices in huge clusters
    

All slices for that Service have the label:

```
kubernetes.io/service-name: <service-name>
```

This makes it easy for:

- kube-proxy
    
- CoreDNS
    
- kubectl
    
- controllers
    

to fetch only the slices related to one Service.

* * *

# 🧠 **4\. Example Output**

Run:

```bash
kubectl get endpointslice -n default -l kubernetes.io/service-name=auth-service
```

You may see:

```
NAME                                ADDRESSTYPE   PORTS   ENDPOINTS   AGE
auth-service-jh56x                  IPv4          8080    10.10.12.45  10h
auth-service-k92ms                  IPv4          8080    10.10.12.46  10h
```

These are the **two EndpointSlices** for `auth-service`, each listing some Pod IPs.

* * *

# 🟦 **5\. Why EndpointSlices have multiple objects?**

Because Kubernetes keeps **up to 100 endpoints per slice** for performance.

Large service example:

- 500 pods behind a service → 5 EndpointSlices created
    
- Each slice has the same label:  
    `kubernetes.io/service-name: auth-service`
    

Using a selector lets you fetch all slices for that one Service.

* * *

# 🟢 **6\. Summary**

| Thing | Meaning |
| --- | --- |
| `kubernetes.io/service-name` | Label added by Kubernetes to EndpointSlices |
| `--selector=...` | Tells kubectl to filter only resources with that label |
| Why needed? | A Service may have many EndpointSlices |

* * *

# 🟢 **Final Simple Explanation**

`--selector=kubernetes.io/service-name=<svc-name>`  
means:

👉 **Get only EndpointSlices that belong to Service `<svc-name>`**.

* * *

If you want, I can show:

- How to see the actual Pod IPs inside EndpointSlices
    
- How kube-proxy reads EndpointSlices
    
- How to debug if traffic is not reaching pods
    

Just ask!

&nbsp;

&nbsp;

Great question — and yes, **Endpoints** also showed only the Pod IPs related to one Service.  
So why do we need selectors for EndpointSlices?

Let’s compare **Endpoints vs EndpointSlices** and explain the difference **very clearly**.

* * *

# ✅ **1\. OLD: Endpoints (1 object per Service)**

Before EndpointSlices existed:

- Every Service had **exactly one** Endpoints object.
    
- This object contained **all Pod IPs** behind that service.
    

Example:

```
kubectl get endpoints auth-service
```

You always got the correct object because:

### ✔ The Endpoints object name **exactly matches the Service name**

`auth-service` → Endpoints `auth-service`

So there was no need to “filter” — there was always one 1:1 object.

**Structure:**

```
Endpoints:
  name: auth-service
```

That's why you simply did:

```
kubectl get endpoints auth-service
```

* * *

# ❗ Problem with Old Endpoints

A single object had potentially **thousands of Pod IPs** — this caused:

- large API objects
    
- large etcd writes
    
- slow updates
    
- controller bottlenecks
    
- scalability issues in clusters > 5,000 pods behind a service
    

Therefore, Kubernetes created **EndpointSlices** to replace Endpoints.

* * *

# 🆕 **2\. NEW: EndpointSlices (1 service → MANY objects)**

This is the key difference:

### ✔ A Service can have **multiple EndpointSlice objects**

Each EndpointSlice stores **up to 100 Pod IPs**.

Example: 450 Pods = 5 slices

```
auth-service-abcd1
auth-service-abcd2
auth-service-abcd3
auth-service-abcd4
auth-service-abcd5
```

Now the objects **no longer match the Service name**.

So you CANNOT do:

```
kubectl get endpointslice auth-service
```

Because names are random.

* * *

# 🧠 **Why do we need selectors for EndpointSlices?**

Because **many EndpointSlice objects exist**, so we filter by labels.

All EndpointSlices for a Service have a label:

```
kubernetes.io/service-name: auth-service
```

Therefore we fetch them using:

```bash
kubectl get endpointslice -l kubernetes.io/service-name=auth-service
```

### ✔ This returns *all slices* belonging to that service

### ✔ Without knowing their random auto-generated names

* * *

# 🟦 **3\. Summary of the difference**

| Feature | Endpoints (Old) | EndpointSlices (New) |
| --- | --- | --- |
| Number per service | **1** | **Many** (1 to hundreds) |
| Name | Same as service | Auto-generated |
| Fetch method | `kubectl get endpoints svc-name` | `selector=kubernetes.io/service-name=svc-name` |
| Scalability | Poor | Excellent |
| Used by kube-proxy | No (legacy) | Yes (default) |

* * *

# 🟢 **4\. Simple Explanation**

### ✔ Before (Endpoints):

One object → easy to fetch → matched service name.

### ✔ Now (EndpointSlices):

Many objects → must filter with label → label links slice to service.

* * *

# 🟣 VISUAL EXAMPLE

### With 250 pods:

### Old:

```
Endpoints (1 object)
- 250 pod IPs
```

### New:

```
EndpointSlice A → 100 pods  (kubernetes.io/service-name=auth-service)
EndpointSlice B → 100 pods  (kubernetes.io/service-name=auth-service)
EndpointSlice C →  50 pods  (kubernetes.io/service-name=auth-service)
```

To get all slices for this service:

```
kubectl get endpointslice -l kubernetes.io/service-name=auth-service
```

&nbsp;

&nbsp;

&nbsp;

Great — this is the **right question**, and the answer is NOT about the number of IPs…  
It’s about **HOW** Kubernetes stores, updates, and distributes them.

Let’s explain it clearly.

* * *

# ✅ \*\*Key Idea: The number of IPs did NOT change.

What changed is how Kubernetes stores and updates them.\*\*

### ❗ You still have 250 Pod IPs

BUT the difference is:

| Old (Endpoints) | New (EndpointSlices) |
| --- | --- |
| All 250 IPs inside ONE object | 250 IPs split across MANY smaller objects |
| Huge object | Several small objects |
| Heavy API writes | Light, parallel writes |
| Slow | Fast & scalable |
| API server bottlenecks | Distributed load |
| NOT indexed | Fully indexed labels |
| kube-proxy reads 1 giant blob | kube-proxy reads small slices |

The **content** didn’t change → the **architecture** changed.

* * *

# 🧠 WHY CHANGE IF IP COUNT IS SAME?

Because **scalability and API performance** depend NOT on the number of items,  
but on:

### ✔ Size of API objects

### ✔ Frequency of updates

### ✔ How controllers watch/notify changes

### ✔ How kube-proxy generates routing rules

Let’s break these down.

* * *

# 🟥 1. Old Endpoints: ONE GIANT API OBJECT

Example for 250 pods:

```
Endpoints(auth-service)
- IP1
- IP2
- …
- IP250
```

**Problems:**

### ❌ Huge object (hundreds of KB)

### ❌ Any Pod update rewrites entire object

A pod restarts → the ENTIRE endpoints object is rewritten.

### ❌ kube-proxy must reprocess ALL 250 IPs

Even if only 1 changed.

### ❌ etcd becomes slow

Large writes → heavy replication → latency → failure domains.

* * *

# 🟩 2. New EndpointSlices: MANY small objects

For 250 pods:

```
Slice-1 → 100 IPs
Slice-2 → 100 IPs
Slice-3 → 50 IPs
```

### ✔ Small objects (usually < 20 KB)

### ✔ Only the affected slice updates

1 Pod moves nodes → ONLY the slice containing that Pod is updated.

### ✔ kube-proxy receives incremental updates

Not the whole dataset.

### ✔ API server & etcd handle small writes better

More stable, more scalable.

### ✔ Slices can be sharded by Node topology

Better locality → fewer cross-node hops.

* * *

# 🧠 Analogy

### **Old system = one very large Excel sheet**

Every time 1 cell changes → you must save the whole file.

### **New system = 3 smaller Excel sheets**

You only update the sheet where the change happened.

Same amount of data overall, but far easier and faster to manage.

* * *

# 🟦 3. Without reopening the entire monster object

With EndpointSlices:

- Updates are **partial**
    
- kube-proxy recalculates **only what changed**
    
- Node-local caching is better
    
- Load is distributed
    

This is massively important in clusters with:

- many services
    
- many pods
    
- frequent scaling
    
- autoscaling events
    
- node failures
    
- rolling updates
    

* * *

# 🟧 4. Kubernetes performance issues disappeared

Before EndpointSlices, large clusters hit:

- etcd timeouts
    
- API server throttling
    
- kube-proxy CPU spikes
    
- > > 10s delays in load balancer updates
    

With EndpointSlices, these problems disappeared.

* * *

# 🟢 SUMMARY

The number of Pod IPs didn’t change.  
The **data model and update mechanism** changed.

### Endpoints (old):

- 1 giant object
    
- full rewrites
    
- poor scalability
    

### EndpointSlices (new):

- many small objects
    
- partial, efficient updates
    
- scalable
    
- kube-proxy optimization
    

* * *

&nbsp;

&nbsp;

Let’s go step-by-step and explain **“Updates are partial – kube-proxy recalculates only what changed”** in a clear and basic way.

* * *

# ✅ **kube-proxy Partial Updates — Full Explanation**

Earlier (before EndpointSlices) kube-proxy watched **Endpoints** objects only.

Now kube-proxy watches **EndpointSlices**.

But the important point is:

# 🚀 \*\*kube-proxy does NOT rebuild all iptables rules from zero every time something changes.

It only updates the *exact rules* affected by the change.\*\*

* * *

# 📌 **Before: Full Sync (Old Behavior)**

Originally, kube-proxy would:

1.  Watch the **Endpoints** object for a Service.
    
2.  If *any* endpoint changed (added/removed), kube-proxy rebuilt the **entire iptables chain** for that Service.
    

Example:  
If a Service had **250 endpoints**, and one pod died:

- kube-proxy rebuilt ALL 250 endpoint iptables rules again
    
- This repeated for EVERY node
    
- Massive CPU usage + high churn on large clusters
    

* * *

# 📌 **Now: Partial Updates (New Behavior with EndpointSlices)**

kube-proxy watches **EndpointSlices**, where endpoints are split into chunks (default 100 per slice).

If 1 endpoint changes:

👉 Only the **specific slice** that contains that endpoint is updated  
👉 kube-proxy updates **only the exact iptables rules** related to that slice

### ✔ No need to rebuild all 250 entries

### ✔ No need to recalc rules for other slices

### ✔ Much faster and scalable

* * *

# 🎯 **Concrete Example**

### Example Service

- Total endpoints = **250**
    
- EndpointSlice size = **100**
    

So Kubernetes creates:

- slice-1 → 100 endpoints
    
- slice-2 → 100 endpoints
    
- slice-3 → 50 endpoints
    

* * *

### Scenario: 1 pod dies

### **Before (Endpoints object)**

Endpoints object changed → kube-proxy recalculates all 250 iptables rules again.

* * *

### **Now (EndpointSlices)**

Only **slice-3** (which contains the dead endpoint) changes.

kube-proxy:

- Does NOT touch slice-1 rules
    
- Does NOT touch slice-2 rules
    
- ONLY updates iptables rules for slice-3
    

* * *

# 🔥 Why This Matters?

### **1\. Lower CPU usage**

Because iptables restore of large chains is expensive.

### **2\. Faster convergence**

Updating a single slice with 50 entries is fast.

### **3\. Lower load on API server**

Because fewer large objects change.

### **4\. Scalability**

Clusters with 10,000+ pods benefit a lot.

* * *

# 💡 **So… What changed?**

You asked:

> "Earlier we also had to see between 250 IPs, now also there are 250 IPs. What changed?"

✔ The **number of real endpoints (pod IPs) is same**  
❌ But the **way updates are sent & processed is different**

| Old (Endpoints) | New (EndpointSlices) |
| --- | --- |
| All endpoints in 1 object | Endpoints split into multiple slices |
| Updating 1 endpoint → large object updated | Updating 1 endpoint → only one small slice updated |
| kube-proxy recalculates entire iptables chain | kube-proxy updates only the affected slice |
| High CPU | Low CPU |
| Bad scalability | Excellent scalability |

* * *

# 📌 kube-proxy partial update: Summary in One Line

> kube-proxy no longer rebuilds the entire iptables chain when endpoints change — it only updates the rules for the specific EndpointSlice that changed.

* * *

&nbsp;

&nbsp;

&nbsp;

✔ How to see Pod IPs from EndpointSlices 

✔ How kube-proxy reads EndpointSlices

✔ How kube-proxy creates iptables rules from them

✔ Full cluster traffic flow: DNS → ClusterIP → iptables → Pod

Nice — I’ll walk through all four topics from the ground up and show exact commands you can run to inspect each piece in *your* cluster. I’ll keep it practical: concept → how it works → what to run → what to look for.

* * *

# 1) How to see Pod IPs from EndpointSlices

### Concept (short)

An **EndpointSlice** is a Kubernetes object that contains a small list (up to ~100) of endpoints (pod IPs + optional topology/port info). Many slices together represent all backends for a Service.

### What EndpointSlice contains

Example fields in an EndpointSlice:

```yaml
metadata:
  name: auth-service-xyz
  labels:
    kubernetes.io/service-name: auth-service
endpoints:
  - addresses: [ "10.244.1.15" ]
    conditions:
      ready: true
  - addresses: [ "10.244.2.20" ]
ports:
  - name: http
    port: 8080
    protocol: TCP
```

### Commands to list and inspect EndpointSlices for a Service

(Replace `<ns>` and `<svc>` with your namespace and service.)

- List all EndpointSlices for a service:

```bash
kubectl -n <ns> get endpointslice -l kubernetes.io/service-name=<svc>
```

- Show full YAML of all slices (readable):

```bash
kubectl -n <ns> get endpointslice -l kubernetes.io/service-name=<svc> -o yaml
```

- Show them one-by-one:

```bash
kubectl -n <ns> get endpointslice -l kubernetes.io/service-name=<svc> -o custom-columns=NAME:.metadata.name,ENDPOINTS:.endpoints,PORTS:.ports
```

- Compare with the legacy Endpoints object:

```bash
kubectl -n <ns> get endpoints <svc> -o yaml
```

### What to look for

- `metadata.labels.kubernetes.io/service-name` — ties slice to service
    
- `endpoints[].addresses` — pod IPs
    
- `ports[]` — port(s) for each endpoint
    
- `conditions.ready` — if an endpoint/pod is considered ready
    

* * *

# 2) How kube-proxy reads EndpointSlices

### Concept (short)

`kube-proxy` watches the Kubernetes API (via the informer/watch mechanism). It watches Service and EndpointSlice events and maintains a local in-memory view (cache) of Services and endpoints. When EndpointSlices change, kube-proxy computes only the incremental changes and programs the networking stack (iptables or IPVS).

### How it works (internals, simplified)

1.  kube-proxy starts and creates informers for Services and EndpointSlices (and Endpoints if needed).
    
2.  Informers receive ADD/UPDATE/DELETE events from the API server.
    
3.  kube-proxy has a local store (cache) of current Service → endpoints mappings.
    
4.  On an event, it computes a **delta** (what changed) and calls the mode-specific handler:
    
    - iptables mode: update iptables chains/entries for changed service/endpoints.
        
    - IPVS mode: update IPVS virtual server and real server entries for changed endpoints.
        
5.  kube-proxy applies the minimal set of changes to the kernel rules (no full rebuild unless required).
    

### Commands to inspect kube-proxy behavior

- Find kube-proxy pods:

```bash
kubectl -n kube-system get pods -l k8s-app=kube-proxy -o wide
```

- View kube-proxy command line (to see mode iptables/ipvs):

```bash
kubectl -n kube-system get pod -l k8s-app=kube-proxy -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{.spec.containers[0].command}{"\n\n"}{end}'
```

or inspect one pod:

```bash
kubectl -n kube-system get pod <kube-proxy-pod> -o yaml | yq '.spec.containers[0].args'
```

(look for `--proxy-mode=iptables` or `--proxy-mode=ipvs`)

- Look at kube-proxy logs to see reactions to changes:

```bash
kubectl -n kube-system logs -l k8s-app=kube-proxy --tail=200
```

Search for endpoint/slice-related messages (ADD/UPDATE/DELETE).

### What to look for in logs

- Messages about syncing services/endpoints
    
- Warnings about failures to program rules
    
- If you change or scale pods, you should see corresponding UPDATE events in kube-proxy logs
    

* * *

# 3) How kube-proxy creates iptables rules from EndpointSlices

### Concept (short)

In **iptables** mode, kube-proxy translates the Service+Endpoint mapping into NAT rules in the kernel `nat` table. It creates a front-end chain per service (`KUBE-SVC-xxxx`) and per-endpoint chains (`KUBE-SEP-xxxx`) that DNAT traffic to backend pod IP:port. With EndpointSlices kube-proxy manages these chains incrementally: add/remove the `KUBE-SEP` chains for endpoints that changed.

### High-level rule layout (iptables nat table)

- `KUBE-SERVICES` — top-level chain for all services
    
    - Jumps to `KUBE-SVC-<hash>` for a particular service IP/port
- `KUBE-SVC-<hash>` — service-specific chain; load-balances by jumping to backend `KUBE-SEP-<hash>` chains (round-robin/random)
    
- `KUBE-SEP-<hash>` — per-endpoint chain that DNATs to `podIP:port`
    
- `KUBE-NODEPORTS` / `KUBE-EXTERNAL-SERVICES` — additional chains for NodePort / ExternalTrafficPolicy
    

### Commands to inspect iptables rules kube-proxy created

(On the node where kube-proxy runs; you may need `sudo`.)

- Dump all nat rules and grep KUBE:

```bash
sudo iptables-save | grep -n "KUBE-"
```

- Show the top-level KUBE-SERVICES chain:

```bash
sudo iptables -t nat -L KUBE-SERVICES -n -v --line-numbers
```

- Identify the KUBE-SVC chain for a specific ClusterIP:
    
    1.  Get ClusterIP of service:
        
        ```bash
        kubectl -n <ns> get svc <svc> -o jsonpath='{.spec.clusterIP}'
        ```
        
    2.  Search in KUBE-SERVICES:
        
        ```bash
        sudo iptables -t nat -L KUBE-SERVICES -n -v | grep <ClusterIP>
        ```
        
    3.  Note the target chain name `KUBE-SVC-XXXX`. List that chain:
        
        ```bash
        sudo iptables -t nat -L KUBE-SVC-XXXX -n -v --line-numbers
        ```
        
    4.  That chain will reference `KUBE-SEP-YYYY` entries. Inspect a KUBE-SEP chain:
        
        ```bash
        sudo iptables -t nat -L KUBE-SEP-YYYY -n -v
        ```
        
        You will see DNAT rules to the Pod IP:port.
        
- Show all KUBE-SEP chains (each corresponds to an endpoint):
    

```bash
sudo iptables -t nat -L | grep KUBE-SEP -n
```

### What to watch for

- When endpoints are added/removed, *only* the related `KUBE-SEP-...` chains should be created/removed or updated — not all chains rebuilt.
    
- If kube-proxy is misconfigured, you may see high-frequency full `iptables-restore` operations (costly).
    
- Confirm that DNAT targets in KUBE-SEP point to actual Pod IPs listed in EndpointSlices.
    

* * *

# 4) Full cluster traffic flow: DNS → ClusterIP → iptables → Pod

I’ll show the step-by-step flow for a typical request originating from a Pod inside the cluster calling a Service.

### Example: Pod A in namespace `foo` calls `http://auth-service.default.svc.cluster.local:8080`

#### Step A — DNS resolution

1.  Application calls `auth-service.default.svc.cluster.local`.
    
2.  Pod uses `/etc/resolv.conf` nameserver (usually CoreDNS ClusterIP, e.g. `10.96.0.10`) to resolve.
    
    ```bash
    kubectl exec -it <client-pod> -- nslookup auth-service.default.svc.cluster.local
    ```
    
3.  CoreDNS looks at Service objects in API server and returns the Service ClusterIP (e.g. `10.102.45.12`).
    

#### Step B — Application connects to ClusterIP:Port

- Application opens TCP to `10.102.45.12:8080`.
    
- Traffic leaves the pod’s network namespace destined for ClusterIP.
    

#### Step C — iptables NAT interception (kube-proxy rules)

- **If traffic originates on a different node than the destination pod**, the kernel processes NAT PREROUTING hooks.
    
- **If traffic originates on the same node**, kernel may traverse the OUTPUT/PREROUTING paths — kube-proxy installs rules to handle both scenarios correctly.
    
- In the `nat` table, packet hits the `KUBE-SERVICES` chain and is directed to `KUBE-SVC-<hash>`, which uses per-endpoint chains `KUBE-SEP-<hash>` to DNAT the destination IP:port to the chosen backend Pod IP:port.
    
    The transformation looks like:
    
    ```
    dst: 10.102.45.12:8080  ---> DNAT ---> 10.244.1.15:8080
    ```
    

#### Step D — Connection tracking and routing

- conntrack records the mapping so reply packets are correctly SNATed/unsnat.
    
- The kernel routes the packet towards the node hosting the chosen Pod IP (if remote) or to the local Pod (if on same node).
    
- If the Service has `externalTrafficPolicy=Cluster`, source IP may be SNATed; `Local` preserves client source IP but requires local endpoints.
    

#### Step E — Pod receives connection

- The backend Pod’s process (e.g., Spring Boot) accepts connection on port 8080 and replies.
    
- Return packets follow conntrack to be reverse-translated and delivered to the original client Pod.
    

### Commands to trace the flow (practical)

From a client pod, test and observe:

- Resolve DNS:

```bash
kubectl exec -it <client-pod> -- nslookup auth-service.default.svc.cluster.local
```

- Curl ClusterIP (to force iptables path):

```bash
kubectl exec -it <client-pod> -- curl -svI http://10.102.45.12:8080/health
```

- On the node, watch iptables counters (to see hits):

```bash
# Find the KUBE-SVC chain and check packet counts:
sudo iptables -t nat -L KUBE-SVC-XXXX -n -v
```

You should see the packet/byte counters increase as you curl.

- Observe conntrack entries for the flow:

```bash
sudo conntrack -L | grep 10.102.45.12
```

(or filter by pod IP)

- If you want to capture traffic:

```bash
# On node, capture traffic to a pod IP:
sudo tcpdump -n -i any host 10.244.1.15 and port 8080
```

- If IPVS mode, inspect with:

```bash
sudo ipvsadm -ln --stats
```

You’ll see virtual servers for `10.102.45.12:8080` and real servers (pod IPs) listed.

* * *

# Extra notes / advanced details

- **Source of truth:** API server (etcd) stores Services & EndpointSlices. kube-proxy and CoreDNS read from API server.
    
- **Where rules live:** iptables/IPVS rules live in the kernel on each node — kube-proxy programs those locally.
    
- **Conntrack:** long-lived conntrack entries can keep old mappings; clearing conntrack may be needed when debugging service moves.
    
- **L7 vs L4:** Services + kube-proxy are L4 (IP:port). Ingress/Gateway handle L7 (HTTP host/path routing).
    
- **ExternalTrafficPolicy:** affects SNAT behavior for traffic coming to NodePort/LoadBalancer and whether source IP is preserved.
    
- **Hairpin traffic:** kube-proxy supports hairpin for Pods connecting to services that route back to the same node — can be configured/disabled depending on CNI.
    

* * *

# Quick checklist (commands recap)

```bash
# list EndpointSlices for service
kubectl -n <ns> get endpointslice -l kubernetes.io/service-name=<svc>

# show endpoints (legacy)
kubectl -n <ns> get endpoints <svc> -o yaml

# check kube-proxy pods & mode
kubectl -n kube-system get pods -l k8s-app=kube-proxy -o wide
kubectl -n kube-system logs -l k8s-app=kube-proxy --tail=200

# on node: inspect iptables
sudo iptables-save | grep KUBE-
sudo iptables -t nat -L KUBE-SERVICES -n -v

# find KUBE-SVC chain for ClusterIP
kubectl -n <ns> get svc <svc> -o jsonpath='{.spec.clusterIP}'
sudo iptables -t nat -L KUBE-SERVICES -n -v | grep <ClusterIP>

# on node: if ipvs mode
sudo ipvsadm -ln

# debug flows
kubectl exec -it <client-pod> -- nslookup <svc>.<ns>.svc.cluster.local
kubectl exec -it <client-pod> -- curl -v http://<clusterIP>:<port>/
sudo tcpdump -n -i any host <pod-ip> and port <port>
sudo conntrack -L | grep <clusterIP or pod-ip>
```

* * *

# TL;DR (one-paragraph)

EndpointSlices split the full list of pod IPs into many small objects. kube-proxy watches those objects and applies only the incremental changes to the kernel (iptables/IPVS) so it updates or removes only the specific per-endpoint chains (KUBE-SEP) instead of rebuilding everything. DNS (CoreDNS) maps service names to ClusterIP, and when a pod calls that ClusterIP the kernel NAT rules (programmed by kube-proxy) DNAT the packet to an actual Pod IP and route the connection there. Use `kubectl get endpointslice -l kubernetes.io/service-name=<svc>`, `kubectl -n kube-system logs -l k8s-app=kube-proxy`, `sudo iptables -t nat -L KUBE-SERVICES -n -v`, and `ipvsadm -ln` to inspect each component.

* * *

If you want, I can:

- run through a live example with the exact service name from your cluster (paste service name & namespace), or
    
- show the exact iptables chains on a node and explain each rule line-by-line. Which do you want next?
    

&nbsp;

Here’s a **concise, practical summary** of the full flow from EndpointSlices → kube-proxy → iptables → Pod:

* * *

## 1) **See Pod IPs from EndpointSlices**

- EndpointSlices split a Service’s endpoints into small objects (~100 IPs each).
    
- Commands:
    

```bash
kubectl -n <ns> get endpointslice -l kubernetes.io/service-name=<svc>
kubectl -n <ns> get endpointslice -l kubernetes.io/service-name=<svc> -o yaml
kubectl -n <ns> get endpointslice -l kubernetes.io/service-name=<svc> -o custom-columns=NAME:.metadata.name,ENDPOINTS:.endpoints,PORTS:.ports
```

- Look for `endpoints[].addresses` (Pod IPs), `ports[]`, `conditions.ready`.

* * *

## 2) **How kube-proxy reads EndpointSlices**

- Watches Service & EndpointSlice events.
    
- Maintains a local cache.
    
- On changes, calculates **delta** (what changed) and updates kernel rules incrementally.
    
- Commands:
    

```bash
kubectl -n kube-system get pods -l k8s-app=kube-proxy -o wide
kubectl -n kube-system logs -l k8s-app=kube-proxy --tail=200
```

- Look for ADD/UPDATE/DELETE events.

* * *

## 3) **kube-proxy → iptables rules**

- In iptables mode:
    
    - `KUBE-SERVICES` → jumps to `KUBE-SVC-XXXX` (per service)
        
    - `KUBE-SVC-XXXX` → jumps to `KUBE-SEP-YYYY` (per endpoint)
        
    - `KUBE-SEP-YYYY` → DNAT to Pod IP:port
        
- Incremental updates: only changed endpoints’ chains are updated.
    
- Commands:
    

```bash
sudo iptables-save | grep KUBE-
sudo iptables -t nat -L KUBE-SERVICES -n -v
sudo iptables -t nat -L KUBE-SVC-XXXX -n -v
sudo iptables -t nat -L KUBE-SEP-YYYY -n -v
```

* * *

## 4) **Cluster traffic flow: DNS → ClusterIP → Pod**

1.  Pod resolves Service via CoreDNS → ClusterIP.

```bash
kubectl exec -it <client-pod> -- nslookup <svc>.<ns>.svc.cluster.local
```

2.  Connects to ClusterIP:Port → intercepted by iptables (kube-proxy) → DNAT to Pod IP.
    
3.  Kernel routes packet to Pod (local or remote node).
    
4.  Pod responds → conntrack ensures reply reaches source Pod.
    

- Commands to trace:

```bash
kubectl exec -it <client-pod> -- curl -v http://<clusterIP>:<port>/
sudo iptables -t nat -L KUBE-SVC-XXXX -n -v
sudo conntrack -L | grep <clusterIP or pod-ip>
sudo tcpdump -n -i any host <pod-ip> and port <port>
sudo ipvsadm -ln   # if IPVS mode
```

* * *

### ✅ **Key points**

- EndpointSlices = smaller, incremental updates → scalable.
    
- kube-proxy watches slices → updates only changed endpoints → minimal iptables changes.
    
- DNS resolves Service → ClusterIP → iptables → Pod.
    
- Tools: `kubectl endpointslice`, `kubectl logs kube-proxy`, `iptables-save`, `conntrack`, `ipvsadm`.
    

* * *

I can also make a **single diagram showing DNS → ClusterIP → iptables → EndpointSlices → Pod** to visualise this flow.

Do you want me to make that diagram?

&nbsp;

&nbsp;

&nbsp;

Sure! Let’s break down **“KUBE chains”** in Kubernetes iptables mode from the basics.

* * *

## 1️⃣ What are KUBE chains?

When **kube-proxy** runs in **iptables mode**, it programs the Linux kernel’s `iptables` NAT table to handle **Service → Pod traffic**.

It creates a set of chains prefixed with `KUBE-`:

| Chain | Purpose |
| --- | --- |
| **KUBE-SERVICES** | Top-level chain for all services. Handles incoming traffic to any ClusterIP or NodePort. |
| **KUBE-SVC-XXXX** | One per Service. Responsible for load-balancing traffic to the service’s endpoints. |
| **KUBE-SEP-YYYY** | One per **endpoint** (Pod IP:port). Performs **DNAT** to redirect traffic to the actual Pod. |
| **KUBE-NODEPORTS** | Handles NodePort traffic. Maps NodePort → ClusterIP → endpoints. |
| **KUBE-EXTERNAL-SERVICES** | For External IPs / LoadBalancer services. |

* * *

## 2️⃣ How traffic flows through KUBE chains

Let’s say a Pod calls **ClusterIP 10.102.45.12:8080**.

1.  Packet hits the kernel NAT PREROUTING (or OUTPUT if local) chain.
    
2.  It enters **KUBE-SERVICES**, which looks for a matching service.
    
3.  Jumps to **KUBE-SVC-XXXX** for that service.
    
4.  **KUBE-SVC-XXXX** load-balances traffic to **KUBE-SEP-YYYY** chains (one per Pod IP).
    
5.  **KUBE-SEP-YYYY** DNATs packet → actual Pod IP:Port.
    
6.  Kernel routes packet to the Pod.
    

This is **all L4 (TCP/UDP) load-balancing**, managed entirely in iptables.

* * *

## 3️⃣ How to see KUBE chains

On the node:

```bash
sudo iptables -t nat -L KUBE-SERVICES -n -v --line-numbers
sudo iptables -t nat -L KUBE-SVC-XXXX -n -v --line-numbers
sudo iptables -t nat -L KUBE-SEP-YYYY -n -v
```

- `KUBE-SERVICES` → top-level list of services and target chains.
    
- `KUBE-SVC-XXXX` → load-balancing to endpoints.
    
- `KUBE-SEP-YYYY` → DNAT to Pod IP.
    

You can see **packet/byte counters** for each chain — useful to debug which Pods are receiving traffic.

* * *

## 4️⃣ Key points

- Each **Service** has one **KUBE-SVC chain**, each **Pod endpoint** has one **KUBE-SEP chain**.
    
- Updates are **incremental**: kube-proxy modifies only the chains for endpoints that changed.
    
- NodePort / External IP traffic also uses these chains via **KUBE-NODEPORTS** or **KUBE-EXTERNAL-SERVICES**.
    
- If kube-proxy runs in **IPVS mode**, iptables chains may not exist — use `ipvsadm` instead.
    

* * *

If you want, I can make a **small diagram showing KUBE-SERVICES → KUBE-SVC → KUBE-SEP → Pod IP** flow — it makes the concept very clear visually.

Do you want me to do that?

&nbsp;

&nbsp;

&nbsp;