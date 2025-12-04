* * *

## **1️⃣ kubelet**

### **What it is**

- kubelet is the **primary agent running on each Kubernetes node**.
    
- Its job is to **ensure that the containers (pods) described in your manifests are running and healthy**.
    

### **Responsibilities**

1.  Watches the **API server** for Pod specs assigned to its node.
    
2.  Starts/stops containers using the container runtime (e.g., Docker, containerd).
    
3.  Monitors **container health** and reports back to the API server.
    
4.  Executes **liveness/readiness probes**.
    
5.  Manages **Pod lifecycle**: creation, deletion, updates.
    

### **Key points**

- kubelet runs on **every node** (worker or master if it hosts workloads).
    
- Talks to **container runtime** to launch pods.
    
- Reports **status back to API server**, so `kubectl get pods` shows the current state.
    

### **Example command**

```bash
# Check kubelet service on node
systemctl status kubelet

# Get kubelet logs
journalctl -u kubelet -f
```

* * *

## **2️⃣ kube-proxy**

### **What it is**

- kube-proxy is the **networking agent** that runs on **every node**.
    
- Its job is to **route traffic from services to the right pods**, i.e., it implements **Service load-balancing** at L4 (TCP/UDP).
    

### **Responsibilities**

1.  Watches **Services and Endpoints/EndpointSlices** from API server.
    
2.  Maintains **iptables or IPVS rules** to forward Service IP traffic to Pod IPs.
    
3.  Handles **ClusterIP, NodePort, External IPs**.
    
4.  Supports **incremental updates** when endpoints change.
    

### **Key points**

- kube-proxy is purely about **network traffic**, not running containers.
    
- Can run in **iptables mode** (programs NAT chains) or **IPVS mode** (virtual servers + real servers).
    

### **Example command**

```bash
# Check kube-proxy pods
kubectl -n kube-system get pods -l k8s-app=kube-proxy

# Logs for debugging
kubectl -n kube-system logs kube-proxy-xxxx --tail=50

# Check IPVS (if in IPVS mode)
sudo ipvsadm -ln
sudo iptables-show
```

* * *

## **3️⃣ Comparison**

| Feature | kubelet | kube-proxy |
| --- | --- | --- |
| Role | Node agent, manages pods | Node agent, manages networking |
| Works with | Container runtime (Docker, etc.) | Kernel iptables/IPVS |
| Purpose | Ensure pods/containers run | Forward service traffic to pods |
| Reports to | API server | API server (for watching only) |

* * *

### **TL;DR**

- **kubelet** → “Run my pods correctly.”
    
- **kube-proxy** → “Route traffic from Service IPs to the correct pod.”
    

* * *

&nbsp;