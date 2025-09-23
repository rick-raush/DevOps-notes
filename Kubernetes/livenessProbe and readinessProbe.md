**livenessProbe** and **readinessProbe** settings for a container.

### What are these probes?

- **livenessProbe**: Checks if the container is alive/running. If this probe fails, Kubernetes will restart the container because it considers the container unhealthy or stuck.
    
- **readinessProbe**: Checks if the container is ready to accept traffic. If this probe fails, Kubernetes will stop sending traffic to the pod but won’t restart it.
    

* * *

### Breakdown of your config:

#### livenessProbe:

```yaml
failureThreshold: 4
successThreshold: 1
initialDelaySeconds: 60
httpGet:
  path: /actuator/health
  port: http
```

- **failureThreshold: 4**  
    The probe will try 4 times before marking the container as unhealthy and restarting it.
    
- **successThreshold: 1**  
    The probe only needs 1 successful check to consider the container healthy again.
    
- **initialDelaySeconds: 60**  
    Wait 60 seconds after the container starts before performing the first probe. This gives the app time to initialize.
    
- **httpGet:**  
    The probe performs an HTTP GET request to the path `/actuator/health` on the port named `http` inside the container. It expects a successful HTTP response (usually 200) to consider the container healthy.
    

* * *

#### readinessProbe:

```yaml
failureThreshold: 50
successThreshold: 1
initialDelaySeconds: 60
httpGet:
  path: /actuator/health
  port: http
```

- **failureThreshold: 50**  
    This is quite high. It means Kubernetes will try 50 times before marking the pod as "not ready" and stop sending traffic to it. This can be useful if the application takes a long time to become ready.
    
- **successThreshold: 1**  
    Just one successful probe is enough for the container to be considered ready again.
    
- **initialDelaySeconds: 60**  
    Wait 60 seconds after startup before starting readiness checks.
    
- **httpGet:**  
    Same as livenessProbe — HTTP GET on `/actuator/health` on port `http`.
    

* * *

### Summary

- After the container starts, Kubernetes waits 60 seconds before probing.
    
- It checks `/actuator/health` endpoint to determine health and readiness.
    
- For **liveness**, it gives 4 chances before restarting the container.
    
- For **readiness**, it is more tolerant (50 tries) before marking the container as not ready and stopping traffic.
    
- Both probes require only 1 successful response to consider the container healthy or ready again.
    

* * *

&nbsp;

&nbsp;