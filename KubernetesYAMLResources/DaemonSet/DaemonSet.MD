# DaemonSet

## 1. What is a DaemonSet?
A DaemonSet is a controller that ensures a copy of a Pod is running on all (or some) Nodes.

In Kubernetes, a DaemonSet ensures that a specific Pod runs on every single Node (or a specific subset of Nodes) in your cluster. If you add a new server to your cluster, the DaemonSet automatically spots it and spins up a Pod there.

### Why use it?
You use a DaemonSet when the service you are running is a background infrastructure task rather than a user-facing application. These tasks need to be "local" to the machine to do their job.

- Log Collection: You need a Pod on every node to grab logs from all the other containers on that specific machine (e.g., Fluentd, Logstash).
- Monitoring: You need a Pod on every node to report CPU/RAM usage of the physical hardware (e.g., Prometheus Node Exporter, Datadog agent).
- Networking: Running the actual "plumbing" of the cluster (e.g., kube-proxy or CNI plugins like Calico/Flannel).

## 2. Key Benefits: Why this resource exists
### A. Automatic Scalability
Unlike a Deployment, where you have to say replicas: 5, a DaemonSet has no replica count. Its "count" is equal to the number of nodes. If your cluster grows from 10 nodes to 100 nodes, the DaemonSet automatically scales to 100 Pods.

### B. Node Affinity
You can use nodeSelector or affinity to tell a DaemonSet: "Only run on nodes that have a GPU" or "Only run on nodes in the SSD storage tier." It will then ensure every eligible node has that Pod.

### C. Priority
DaemonSet pods are often critical for the cluster to function. Kubernetes can be configured to ensure these Pods are started before your regular app Pods.

## 3. The YAML Structure (The Blueprint)

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: my-daemonset
  labels:
    app: my-app
spec:
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-container
          image: nginx:latest
          ports:
            - containerPort: 80
```

## 4. Essential DaemonSet Commands

| Command | Description |
|--------|-------------|
| `kubectl get ds` | Lists all DaemonSets. Shows **DESIRED**, **CURRENT**, and **READY** counts based on node availability. |
| `kubectl describe ds <name>` | Displays detailed info, including which nodes Pods are failing to start on. |
| `kubectl rollout status ds/<name>` | Checks the progress of a DaemonSet update across the cluster. |
| `kubectl rollout undo ds/<name>` | Rolls back the DaemonSet to a previous version if issues occur (e.g., crashing nodes). |
| `kubectl get pods -o wide` | Verifies deployment across nodes; you should see one Pod IP per unique Node Name. |

## 5. Summary of the "Big Three" Controllers

- Deployment: "I need 5 copies of this API. I don't care where they sit."
- StatefulSet: "I need 3 databases named db-0, db-1, db-2. They must never switch names or disks."
- DaemonSet: "I need one of these on every single machine in the cluster, no exceptions."
