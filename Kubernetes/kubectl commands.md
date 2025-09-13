List of all the `kubectl` commands  commonly used

* * *

### 🧰 **Basic Commands (Beginner)**

- `kubectl create`
- `kubectl expose`
- `kubectl run`
- `kubectl set`

* * *

### 📘 **Basic Commands (Intermediate)**

- `kubectl explain`
- `kubectl get`
- `kubectl edit`
- `kubectl delete`

* * *

### 🚀 **Deploy Commands**

- `kubectl rollout`
- `kubectl scale`
- `kubectl autoscale`

* * *

### 🛠️ **Cluster Management Commands**

- `kubectl certificate`
- `kubectl cluster-info`
- `kubectl top`
- `kubectl cordon`
- `kubectl uncordon`
- `kubectl drain`
- `kubectl taint`

* * *

### 🐞 **Troubleshooting & Debugging Commands**

- `kubectl describe`
- `kubectl logs`
- `kubectl attach`
- `kubectl exec`
- `kubectl port-forward`
- `kubectl proxy`
- `kubectl cp`
- `kubectl auth`
- `kubectl debug`
- `kubectl events`

* * *

### 🧪 **Advanced Commands**

- `kubectl diff`
- `kubectl apply`
- `kubectl patch`
- `kubectl replace`
- `kubectl wait`
- `kubectl kustomize`

* * *

### ⚙️ **Settings Commands**

- `kubectl label`
- `kubectl annotate`
- `kubectl completion`

* * *

### 🔌 **Plugin & Miscellaneous Commands**

- `kubectl plugin`
- `kubectl api-resources`
- `kubectl api-versions`
- `kubectl config`
- `kubectl version`

* * *

### 🧾 **Imperative Pod Creation Example**

```bash
kubectl run my-pod --image=nginx --restart=Never
```

### 📄 **Declarative Pod Creation Example**

```yaml
kubectl apply -f pod.yaml
```

* * *

&nbsp;

kubectl run my-pod --image=nginx --restart=Never

kubectl get pods

kubectl get pods -o wide

kubectl explain &lt;resource&gt;

kubectl describe pod my-pod  
kubectl logs my-pod

&nbsp;

kubectl expose resource name --port --target-port --name --type

&nbsp;