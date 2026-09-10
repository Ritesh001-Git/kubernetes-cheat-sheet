# 🚨 Kubernetes Common Problems and Debugging Commands

This guide covers common Kubernetes problems and the commands used to diagnose and debug them.

---

# 📦 1. Pod Problems

| Problem                      | Possible Cause                               | Debug Command                        |
| ---------------------------- | -------------------------------------------- | ------------------------------------ |
| `Pending`                    | Insufficient resources, scheduling issue     | `kubectl describe pod <pod-name>`    |
| `CrashLoopBackOff`           | Application keeps crashing                   | `kubectl logs <pod-name> --previous` |
| `ImagePullBackOff`           | Wrong image or registry authentication issue | `kubectl describe pod <pod-name>`    |
| `OOMKilled`                  | Container exceeded memory limit              | `kubectl describe pod <pod-name>`    |
| `ContainerCreating`          | Volume, network, or image issue              | `kubectl describe pod <pod-name>`    |
| `CreateContainerConfigError` | Missing ConfigMap or Secret                  | `kubectl describe pod <pod-name>`    |

### Debug Commands

```bash
# List Pods
kubectl get pods -n <namespace>

# Get detailed Pod information
kubectl describe pod <pod-name> -n <namespace>

# Check Pod logs
kubectl logs <pod-name> -n <namespace>

# Check logs from a crashed container
kubectl logs <pod-name> -n <namespace> --previous

# Watch Pod status
kubectl get pods -n <namespace> -w
```

---

# 🚀 2. Deployment Problems

Common issues:

* Pods are not being created
* Pods are not updating
* Deployment rollout is stuck
* Replica count is incorrect

### Check Deployment

```bash
kubectl get deployments -n <namespace>
```

### Describe Deployment

```bash
kubectl describe deployment <deployment-name> -n <namespace>
```

### Check Deployment Rollout Status

```bash
kubectl rollout status deployment/<deployment-name> -n <namespace>
```

### Check Deployment History

```bash
kubectl rollout history deployment/<deployment-name> -n <namespace>
```

### Roll Back Deployment

```bash
kubectl rollout undo deployment/<deployment-name> -n <namespace>
```

---

# 🌐 3. Service Problems

Common issues:

* Service cannot reach Pods
* Incorrect selector
* Incorrect target port
* No endpoints available

### Check Services

```bash
kubectl get services -n <namespace>
```

### Describe Service

```bash
kubectl describe service <service-name> -n <namespace>
```

### Check Endpoints

```bash
kubectl get endpoints -n <namespace>
```

Or:

```bash
kubectl get endpoints <service-name> -n <namespace>
```

### Check Pod Labels

```bash
kubectl get pods --show-labels -n <namespace>
```

> Make sure the Service selector matches the Pod labels.

Example:

```yaml
# Service
selector:
  app: nginx
```

```yaml
# Pod
labels:
  app: nginx
```

---

# 🔗 4. Kubernetes Networking Problems

Common issues:

* Pods cannot communicate
* Service is unreachable
* DNS resolution fails
* NetworkPolicy blocks traffic

### Check Pod IPs

```bash
kubectl get pods -o wide -n <namespace>
```

### Test Connection from Another Pod

```bash
kubectl exec -it <pod-name> -n <namespace> -- sh
```

Inside the Pod:

```bash
curl http://<service-name>:<port>
```

### Check Network Policies

```bash
kubectl get networkpolicy -n <namespace>
```

### Describe Network Policy

```bash
kubectl describe networkpolicy <policy-name> -n <namespace>
```

---

# 🌍 5. Kubernetes DNS Problems

Common issues:

* Service name cannot be resolved
* CoreDNS is not running
* DNS lookup fails

### Check CoreDNS Pods

```bash
kubectl get pods -n kube-system
```

### Check CoreDNS Logs

```bash
kubectl logs -n kube-system -l k8s-app=kube-dns
```

### Test DNS from a Pod

```bash
kubectl exec -it <pod-name> -n <namespace> -- nslookup <service-name>
```

Example:

```bash
kubectl exec -it nginx-pod -n logging -- nslookup elasticsearch
```

---

# 💾 6. Persistent Volume Problems

Common issues:

* PVC stuck in `Pending`
* Volume not mounting
* StorageClass missing
* Access mode mismatch

### Check Persistent Volumes

```bash
kubectl get pv
```

### Check Persistent Volume Claims

```bash
kubectl get pvc -n <namespace>
```

### Describe PVC

```bash
kubectl describe pvc <pvc-name> -n <namespace>
```

### Check Storage Classes

```bash
kubectl get storageclass
```

### Check Pod Events

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Look for messages such as:

```text
FailedMount
FailedAttachVolume
VolumeBinding
```

---

# 🖥️ 7. Node Problems

Common issues:

* Node is `NotReady`
* Pods cannot be scheduled
* Node has insufficient resources
* Node has taints

### Check Nodes

```bash
kubectl get nodes
```

### Get Detailed Node Information

```bash
kubectl describe node <node-name>
```

### Check Node Resource Usage

```bash
kubectl top nodes
```

### Check Pods Running on a Node

```bash
kubectl get pods -A -o wide
```

### Check Node Taints

```bash
kubectl describe node <node-name>
```

---

# 📊 8. Resource Problems (CPU and Memory)

Common issues:

* Insufficient CPU
* Insufficient memory
* Pods getting evicted
* `OOMKilled`

### Check Node Resources

```bash
kubectl top nodes
```

### Check Pod Resources

```bash
kubectl top pods -A
```

### Check Pod Resource Configuration

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Look for:

```text
Requests:
Limits:
```

---

# 🔐 9. ConfigMap Problems

Common issues:

* ConfigMap does not exist
* Wrong ConfigMap name
* Wrong key
* Pod cannot start

### List ConfigMaps

```bash
kubectl get configmap -n <namespace>
```

### Describe ConfigMap

```bash
kubectl describe configmap <configmap-name> -n <namespace>
```

### View ConfigMap YAML

```bash
kubectl get configmap <configmap-name> -n <namespace> -o yaml
```

---

# 🔑 10. Secret Problems

Common issues:

* Secret does not exist
* Wrong Secret name
* Missing key
* ImagePullSecret problem

### List Secrets

```bash
kubectl get secrets -n <namespace>
```

### Describe Secret

```bash
kubectl describe secret <secret-name> -n <namespace>
```

### View Secret YAML

```bash
kubectl get secret <secret-name> -n <namespace> -o yaml
```

> Kubernetes Secret values are Base64 encoded, not encrypted by default.

---

# 🚪 11. Ingress Problems

Common issues:

* Application is not accessible
* Incorrect hostname
* Incorrect path
* Ingress Controller is not running
* TLS certificate problem

### Check Ingress

```bash
kubectl get ingress -A
```

### Describe Ingress

```bash
kubectl describe ingress <ingress-name> -n <namespace>
```

### Check Ingress Controller

```bash
kubectl get pods -A | grep ingress
```

### Check Ingress Controller Logs

```bash
kubectl logs <ingress-controller-pod> -n <namespace>
```

---

# 🔄 12. Namespace Problems

Common issues:

* Resource deployed in the wrong namespace
* Namespace does not exist
* Namespace stuck in `Terminating`

### List Namespaces

```bash
kubectl get namespaces
```

### Check Resources in a Namespace

```bash
kubectl get all -n <namespace>
```

### Describe Namespace

```bash
kubectl describe namespace <namespace>
```

---

# ⚙️ 13. Kubernetes Events

Events are one of the most important debugging tools.

### Check Events in a Namespace

```bash
kubectl get events -n <namespace>
```

### Sort Events by Time

```bash
kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp
```

### Check Events Across All Namespaces

```bash
kubectl get events -A --sort-by=.metadata.creationTimestamp
```

Look for:

```text
FailedScheduling
FailedMount
FailedAttachVolume
BackOff
Unhealthy
Failed
Killing
Evicted
```

---

# 🐳 14. Container Problems

Common issues:

* Container crashes
* Wrong startup command
* Missing environment variables
* Application port is incorrect

### Check Container Logs

```bash
kubectl logs <pod-name> -n <namespace>
```

### Check Specific Container Logs

```bash
kubectl logs <pod-name> -c <container-name> -n <namespace>
```

### Enter a Running Container

```bash
kubectl exec -it <pod-name> -n <namespace> -- sh
```

For containers with Bash:

```bash
kubectl exec -it <pod-name> -n <namespace> -- bash
```

---

# ❤️ 15. Liveness and Readiness Probe Problems

Common issues:

* Pod keeps restarting
* Pod is running but Service cannot send traffic
* Health check endpoint fails

### Describe Pod

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Look for:

```text
Liveness probe failed
Readiness probe failed
Startup probe failed
```

### Check Pod Events

```bash
kubectl get events -n <namespace>
```

### Check Application Logs

```bash
kubectl logs <pod-name> -n <namespace>
```

---

# 🔍 16. Quick Kubernetes Troubleshooting Checklist

When something is not working, follow this order:

### Step 1: Check Resource Status

```bash
kubectl get all -n <namespace>
```

### Step 2: Check Pod Status

```bash
kubectl get pods -n <namespace>
```

### Step 3: Describe the Problematic Resource

```bash
kubectl describe pod <pod-name> -n <namespace>
```

### Step 4: Check Logs

```bash
kubectl logs <pod-name> -n <namespace>
```

### Step 5: Check Previous Logs if Pod Crashed

```bash
kubectl logs <pod-name> -n <namespace> --previous
```

### Step 6: Check Events

```bash
kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp
```

### Step 7: Check Node Health

```bash
kubectl get nodes
```

### Step 8: Check Resource Usage

```bash
kubectl top nodes
kubectl top pods -A
```

---

# 🚀 Most Useful Kubernetes Debugging Commands

| Command                                    | Purpose                        |
| ------------------------------------------ | ------------------------------ |
| `kubectl get all -n <namespace>`           | View all resources             |
| `kubectl get pods -n <namespace>`          | Check Pod status               |
| `kubectl describe pod <pod-name>`          | Get detailed Pod information   |
| `kubectl logs <pod-name>`                  | Check application logs         |
| `kubectl logs <pod-name> --previous`       | Check crashed container logs   |
| `kubectl exec -it <pod-name> -- sh`        | Enter a running container      |
| `kubectl get events`                       | Check Kubernetes events        |
| `kubectl get nodes`                        | Check Node health              |
| `kubectl top nodes`                        | Check Node CPU and memory      |
| `kubectl top pods`                         | Check Pod CPU and memory       |
| `kubectl rollout status deployment/<name>` | Check Deployment rollout       |
| `kubectl get svc`                          | Check Services                 |
| `kubectl get endpoints`                    | Check Service endpoints        |
| `kubectl get pvc`                          | Check Persistent Volume Claims |
| `kubectl get ingress`                      | Check Ingress resources        |

---

# 💡 General Kubernetes Debugging Flow

```text
Application Not Working
        │
        ▼
kubectl get pods
        │
        ▼
kubectl describe pod
        │
        ▼
kubectl logs
        │
        ▼
kubectl get events
        │
        ▼
Check Service / Networking
        │
        ▼
Check ConfigMap / Secrets
        │
        ▼
Check Storage
        │
        ▼
Check Nodes and Resources
```

> 💡 **Pro Tip:** In Kubernetes, `kubectl describe` and `kubectl get events` are usually the best starting points when you don't know what is wrong.
