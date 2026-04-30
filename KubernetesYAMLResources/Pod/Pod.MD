# Pods

## 1. What is a Pod?
A Pod is the smallest deployable unit in Kubernetes. It represents a single instance of a running process in your cluster.

- The Capsule Metaphor: A Pod is like a capsule or a wrapper around one or more containers (like Docker containers).
- The Rule of One: Most of the time, a Pod runs a single container. However, in advanced cases, you might put two containers in one Pod if they are "tightly coupled" and need to share resources directly.

### Why do we use it?
Kubernetes doesn't manage containers directly. It manages Pods so it can provide:

- Shared Network: All containers in a Pod share the same IP address and port space (they talk to each other via localhost).
- Shared Storage: You can mount a volume that all containers in the Pod can see.
- Lifecycle Management: If a container crashes, K8s knows how to restart the Pod to keep your app alive.

## 2. How it Works Internally
When you tell Kubernetes to create a Pod, a few things happen "under the hood":

- The Sandbox (Pause Container): Kubernetes first creates a "Pause" container. This container does nothing but hold the Network Namespace ($net$) and IPC namespace. This is why containers in the same Pod can share an IP.
- Resource Allocation: The kube-scheduler decides which Node (server) in your cluster has enough CPU and RAM to host the Pod.
- The Kubelet: On the chosen Node, an agent called the kubelet talks to the container runtime (like Docker or containerd) to pull your image and start the containers.

## 3. Real-Life Use Cases

- Single Container (Standard): Running a Node.js API or a Python microservice. This is 90% of use cases.
- Sidecar Pattern (Multi-container): You have a main container running your Web App, and a second "Sidecar" container that sits next to it to collect logs or sync configuration files.
- Init Containers: A specialized container that runs before your app starts (e.g., a script that waits for a database to be ready before launching the main app).

## 4. The YAML Structure (The Blueprint)
Every Kubernetes YAML file has four "Root" keys. Here is a breakdown of a standard Pod manifest:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80

```

## Pod Management Commands

### 1. Creating and Updating Pods

| Command                                      | Description |
|----------------------------------------------|-------------|
| `kubectl apply -f filename.yaml`             | Standard approach to create or update a Pod from a YAML file. |
| `kubectl run my-pod --image=nginx`           | Imperatively creates a Pod without needing a YAML file. |
| `kubectl replace --force -f filename.yaml`   | Deletes the existing Pod and recreates it with updated configuration. |

---

### 2. Inspecting and Monitoring

| Command                                      | Description |
|----------------------------------------------|-------------|
| `kubectl get pods`                           | Lists all Pods in the current namespace. |
| `kubectl get pods -o wide`                   | Shows additional details like Node assignment and Pod IP. |
| `kubectl describe pod <pod-name>`            | Provides detailed debugging info including events and failure reasons. |
| `kubectl get pod <pod-name> -o yaml`         | Outputs the running Pod configuration in YAML format. |
| `kubectl get pods --watch`                   | Continuously watches and updates Pod status in real time. |

---

### 3. Debugging and Troubleshooting

| Command                                      | Description |
|----------------------------------------------|-------------|
| `kubectl logs <pod-name>`                    | Displays logs from the Pod. Use `-f` to stream logs in real time. |
| `kubectl exec -it <pod-name> -- /bin/bash`   | Opens an interactive shell inside the container for debugging. |
| `kubectl port-forward <pod-name> 8080:80`    | Forwards local port 8080 to port 80 on the Pod for testing. |

---

### 4. Deleting Pods

| Command                                      | Description |
|----------------------------------------------|-------------|
| `kubectl delete pod <pod-name>`              | Deletes a specific Pod. |
| `kubectl delete -f filename.yaml`            | Deletes the Pod defined in the YAML file. |
| `kubectl delete pod --all`                   | ⚠️ Deletes all Pods in the current namespace. |
