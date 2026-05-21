# Probes

### 1. What are Probes?

In Kubernetes, Probes are the "Health Checks" that the cluster uses to monitor your containers. Without probes, Kubernetes only knows if the container process is running; it doesn't know if the application inside is actually working or frozen.

A probe is a diagnostic performed periodically by the kubelet on a container. To perform a diagnostic, the kubelet either:

- Executes a command inside the container.
- Sends an HTTP request to a specific path.
- Opens a TCP socket to check if a port is open.

## 2. The Three Types of Probes
### A. Startup Probe (The "Waiting" Probe)

- Purpose: Used for legacy or "slow" applications that take a long time to boot up (e.g., a Java app loading heavy libraries).
- How it works: It disables Liveness and Readiness checks until the container has successfully started.
- Key Benefit: It prevents Kubernetes from killing a Pod that is simply slow to start. Once the Startup probe passes once, it hands over control to the other probes.

### B. Readiness Probe (The "Traffic" Probe)

- Purpose: Tells Kubernetes when a container is ready to accept user traffic.
- How it works: If this probe fails, the Pod's IP address is removed from the Service Endpoints. The Pod stays running, but no traffic is sent to it.
- Real-Life Use: Waiting for a database connection to be established or a large cache to be loaded into memory.

### C. Liveness Probe (The "Self-Healing" Probe)

- Purpose: Tells Kubernetes if the container is still alive.
- How it works: If this probe fails multiple times, Kubernetes kills the container and restarts it.
- Real-Life Use: Fixing "deadlock" situations where the app is technically running but frozen and unable to do work.

## 3. The YAML Structure (The Blueprint)
```
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment
  namespace: nginx

spec:
  replicas: 2

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx
          image: nginx:latest

          ports:
            - containerPort: 80

          resources:
            requests:
              cpu: 100m
              memory: 128Mi

            limits:
              cpu: 200m
              memory: 256Mi

          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 10
            periodSeconds: 5
            timeoutSeconds: 2
            failureThreshold: 3

          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 3
            timeoutSeconds: 2
            failureThreshold: 3
```

### Probe Parameters (The "Knobs")

| Parameter | Meaning |
|---|---|
| `initialDelaySeconds` | How long to wait after the container starts before checking. |
| `periodSeconds` | How often to perform the check (Frequency). |
| `timeoutSeconds` | How long to wait for a response before timing out. |
| `successThreshold` | Minimum consecutive successes to be considered "healthy." |
| `failureThreshold` | Number of retries before Kubernetes gives up and restarts/stops traffic. |

## 4. Implementation Methods
### 1. HTTP Get: Most common. K8s checks for a 200-399 status code.
```
httpGet:
  path: /health
  port: 80
```

### 2. Exec Command: Runs a script. If the exit code is 0, it's healthy.

```
exec:
  command:
  - cat
  - /tmp/healthy
```

### 3. TCP Socket: Just checks if the port is open (good for DBs or non-web apps).

```
    tcpSocket:
      port: 5432
```

## 6. Essential Troubleshooting Commands

| Command | Purpose |
| :--- | :--- |
| `kubectl get pods` | Check the **RESTARTS** column. A high number usually means a failing Liveness probe. |
| `kubectl describe pod <name>` | **Look at the "Events" section.** It will explicitly say "Liveness probe failed" or "Readiness probe failed." |
| `kubectl logs <name> --previous` | See the logs from the container *before* it was restarted by a Liveness probe failure. |

### The "Death Spiral" Warning
Be careful with **Liveness Probes**. If your database goes down and your app starts failing its Liveness probe because it can't talk to the DB, Kubernetes will restart all your app Pods at once. This creates a "thundering herd" effect that can crash your cluster further. 

**Pro-Tip:** Make your Liveness probe check if the *app* is alive, and use the Readiness probe to check if *external dependencies* (like DBs) are available.
