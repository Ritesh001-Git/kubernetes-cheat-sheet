# StatefulSet

## What is a StatefulSet?
A StatefulSet is the Kubernetes resource used to manage stateful applications. It manages the deployment and scaling of a set of Pods and provides guarantees about their ordering and uniqueness.

While Deployments are perfect for "stateless" apps (like a web server where every Pod is identical and replaceable), StatefulSets are designed for the heavy hitters: Databases, Distributed Systems, and Message Queues.

Why use it over Deployments?
In a Deployment, if a Pod dies, K8s replaces it with a random name (e.g., web-xy12). In a StatefulSet, the identity is persistent.
- Stable Network ID: If your Pod is named mysql-0, when it restarts, it will still be named mysql-0.
- Stable Storage: If mysql-0 writes data to a disk, and the Pod dies, the new mysql-0 will automatically re-attach to that exact same disk.
- Ordered Deployment: Pods are created one by one ($0$, then $1$, then $2$). They are also deleted in reverse order ($2$, then $1$, then $0$).

## 2. Key Benefits: Why this resource exists
### A. Predictable DNS
StatefulSets require a Headless Service (a service with clusterIP: None). This gives every Pod a unique DNS address.

- Example: web-0.nginx.default.svc.cluster.local
- This allows other Pods to talk specifically to the "Leader" or a "Follower" in a database cluster.

### B. Persistent Volume Claim Templates
In a Deployment, all Pods share the same volume or get no volume at all. In a StatefulSet, each Pod gets its own unique Persistent Volume (PV). If you have 3 replicas, K8s creates 3 separate disks.

### C. Graceful Transitions
Because distributed systems (like Kafka or ElasticSearch) are sensitive to data corruption, StatefulSets ensure that an update doesn't happen all at once. It ensures the "Quorum" is maintained by updating one Pod at a time.

## 3. The YAML Structure (The Blueprint)
```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: my-stateful-app
spec:
  serviceName: "my-service"
  replicas: 2
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
          volumeMounts:
            - name: data
              mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```

## 4. Essential StatefulSet Commands

| Command | Description |
|--------|-------------|
| `kubectl get sts` | Lists all StatefulSets. |
| `kubectl describe sts <name>` | Displays detailed information including creation order and storage-related events/errors. |
| `kubectl scale sts <name> --replicas=5` | Scales the StatefulSet by adding pods (e.g., `-3`, `-4`) in strict sequential order. |
| `kubectl rollout status sts/<name>` | Monitors the progress of an ordered rollout/update. |
| `kubectl delete sts <name> --cascade=orphan` | Deletes the StatefulSet controller but preserves the existing Pods. |

## 5. The Golden Rule of Deletion
Kubernetes will never automatically delete the Storage (Persistent Volumes) of a StatefulSet.

If you delete the StatefulSet, the disks remain. This is a safety feature to prevent accidental data loss. If you want to wipe the data, you must manually delete the PersistentVolumeClaims (PVCs) after the StatefulSet is gone.

### When to use what? (A Quick Summary)

- Deployment: Your Go/NodeJS/Python API.
- StatefulSet: Your MySQL, Redis, MongoDB, or Kafka cluster.
- DaemonSet: Your Log Collector (Fluentd) or Monitoring Agent (Prometheus Node Exporter).
