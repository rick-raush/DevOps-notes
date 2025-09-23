&nbsp;

## What is a HorizontalPodAutoscaler (HPA)?

In Kubernetes, a **HorizontalPodAutoscaler** automatically adjusts the number of pod replicas in a deployment (or other scalable resources) based on observed metrics like CPU utilization or custom metrics.

- **Horizontal** means scaling out/in by increasing or decreasing the number of pods.
    
- This helps your app handle more load when demand increases and saves resources when demand decreases.
    

* * *

## How HPA works:

1.  **What does it monitor?**  
    The HPA continuously monitors one or more metrics related to your running pods. By default, it mainly uses **CPU utilization**, but it can also use memory or custom metrics if configured.
    
2.  **How often does it check?**  
    The Kubernetes control plane’s autoscaler controller queries metrics from the **metrics-server** (or other metric sources) at regular intervals (usually every 15 seconds).
    
3.  **Calculates desired replicas**  
    Based on the metrics collected (e.g., average CPU usage across all pods in the target deployment), HPA calculates how many pod replicas are needed to keep the metric close to the **target value** you set (like 90% CPU utilization).
    
    For example:
    
    - If average CPU usage is 180% across 2 pods, that means each pod is using 90%, which matches the target, so no scaling needed.
        
    - If average CPU usage goes to 270%, HPA calculates that 3 pods are needed (because 270/3 = 90%).
        
4.  **Scaling decision**  
    The HPA compares the desired number of replicas to the current number of replicas.
    
    - If the desired replicas are **more than current**, HPA will scale **up** (add pods).
        
    - If the desired replicas are **less than current**, HPA will scale **down** (remove pods), respecting the minimum and maximum replica limits.
        
5.  **Update the deployment**  
    The HPA updates the target deployment (or StatefulSet, ReplicaSet, etc.) by changing its `.spec.replicas` field to the new desired number.
    
6.  **Kubernetes handles pod lifecycle**  
    Kubernetes then creates or removes pods as requested, maintaining the desired number of pods.
    

* * *

### Summary in simple terms:

- HPA **measures load** on your pods (usually CPU).
    
- It **calculates** how many pods are needed to keep load balanced.
    
- It **adjusts** the number of pods accordingly.
    
- This helps your app **handle more traffic** automatically or save resources during low usage.
    

* * *

### A couple more points:

- HPA requires the **metrics-server** to be running in your cluster (or some other metrics provider).
    
- It can scale based on **multiple metrics** (like memory, request rate, or custom app metrics) in newer Kubernetes versions.
    
- You can set **minReplicas** and **maxReplicas** to control how far the scaling can go.
    
- HPA performs scaling **gradually** to avoid thrashing (constant up/down scaling).
    

* * *

&nbsp;

## Explanation of your YAML snippet:

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: @@app_name@@
  namespace: staging
spec:
  maxReplicas: 1
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: @@app_name@@
  targetCPUUtilizationPercentage: 90
```

### Fields:

- **apiVersion: autoscaling/v1**  
    This means you're using the autoscaling API version 1 for defining the HPA.
    
- **kind: HorizontalPodAutoscaler**  
    The resource kind is HPA.
    
- **metadata:**
    
    - **name: @@app_name@@** — The name of the HPA resource, here templated as `@@app_name@@`. It will be replaced with your actual app name during deployment.
        
    - **namespace: staging** — The Kubernetes namespace where this HPA will be deployed.
        
- **spec:**  
    This is the important part that defines how the autoscaler behaves.
    
    - **maxReplicas: 1**  
        The maximum number of pod replicas the HPA can scale up to is 1.
        
    - **minReplicas: 1**  
        The minimum number of pod replicas is 1. So it will never scale down below 1.
        
    - **scaleTargetRef:**  
        This points to the Kubernetes resource the HPA will manage — in this case, a **Deployment** named `@@app_name@@`.
        
        - **apiVersion: apps/v1** — The API version of the target resource (Deployment).
            
        - **kind: Deployment** — The type of the target resource.
            
        - **name: @@app_name@@** — The name of the deployment being autoscaled.
            
    - **targetCPUUtilizationPercentage: 90**  
        This is the key metric for autoscaling:  
        The HPA will try to maintain the average CPU utilization across all pods in the target deployment at around 90%.
        
        - If CPU utilization goes above 90%, the HPA tries to add more pods (up to maxReplicas).
            
        - If it drops below, it will scale down (not below minReplicas).
            

* * *

## Summary:

- The HorizontalPodAutoscaler automatically adjusts pod count based on CPU usage.
    
- It targets the deployment named `@@app_name@@` in the `staging` namespace.
    
- It aims to keep average CPU usage at 90%.
    

* * *

&nbsp;

&nbsp;

* * *

### What is templating?

Templating means creating a file with **placeholders** (like variables) that can be replaced with actual values when you deploy or generate configs. This lets you reuse one template for many environments or apps by just swapping out the placeholders.

### Why use templating?

- To keep your config DRY (Don’t Repeat Yourself).
    
- To manage multiple environments (dev, staging, prod) with the same template but different values.
    
- To make configs easy to maintain and update.
    

&nbsp;

### How does the replacement usually happen?

1.  **Templating tool or deployment pipeline**  
    Before sending the YAML to Kubernetes, a tool or script processes your template files. It scans for placeholders like `@@app_name@@` and replaces them with real values.
    
2.  **Where do the values come from?**  
    The values usually come from somewhere like:
    
    - A **values file** (e.g., `values.yaml` in Helm)
        
    - Environment variables set in your CI/CD pipeline
        
    - Command line arguments passed to the templating tool
        
    - Configuration management systems (like Ansible, Terraform, etc.)
        
    - Custom scripts that read a config and do `sed` or string replacements
        
3.  **If you don’t provide the value, what happens?**
    
    - The placeholder `@@app_name@@` stays as-is in the final file.
        
    - Kubernetes will try to apply the manifest with `@@app_name@@` literally, which likely results in errors because names must follow Kubernetes naming rules.
        
    - Your deployment will fail or behave unexpectedly.
        

* * *

&nbsp;