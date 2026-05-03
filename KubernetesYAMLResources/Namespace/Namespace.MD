# Namespace

## 1. What is a Namespace?
A Namespace is a logical partition used to isolate groups of resources. By default, when you start a cluster, Kubernetes creates a few namespaces for you (like default and kube-system).

### Why do we use it?
Without Namespaces, your cluster would be a mess. We use them for:

- Isolation: Team A can’t accidentally delete Team B's Pods.
- Resource Quotas: You can limit Team A to only 4 CPUs so they don't starve the rest of the cluster.
- Organization: Separating environments like development, staging, and production within the same cluster.
- Naming Collision: You can have a Pod named api-server in Namespace A and another Pod named api-server in Namespace B without any conflict.

## 2. Key Benefits: Why this resource exists
### A. Soft Multi-tenancy
It allows multiple teams or projects to share the same hardware. You don't need to buy a separate cluster for every project; you just give each project its own Namespace.

### B. Network Policies
You can set rules that say "Pods in the frontend namespace can talk to the backend namespace, but nothing can talk to the database namespace except the backend."

### C. Resource Limiting
This is the biggest benefit for tech companies. You can attach a ResourceQuota to a namespace to prevent a buggy app from scaling infinitely and driving up your cloud bill.

### 3. Real-World Tech Company YAML Structure
In a professional setting, a Namespace YAML is simple, but it is almost always accompanied by Labels (for billing/tracking) and Resource Quotas.

## 3. The YAML Structure (The Blueprint)

```
apiVersion: v1
kind: Namespace
metadata:
  name: payments-production
  labels:
    team: fintech
    billing-center: "101"
    env: prod
```

### The "Company Guardrail" (ResourceQuota):
This is what tech companies actually use to manage their Namespaces. This file ensures the payments-production namespace doesn't crash the whole cluster.

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: payments-production # Applied specifically to this office
spec:
  hard:
    requests.cpu: "10"      # Total CPU all pods can request
    requests.memory: 20Gi  # Total RAM
    limits.cpu: "20"
    limits.memory: 40Gi
    pods: "50"             # Max 50 pods allowed in this namespace
```

## 4. Essential Namespace Commands

| Command | Description |
|--------|-------------|
| `kubectl get ns` | Lists all namespaces in the cluster. |
| `kubectl create ns <name>` | Quickly creates a new namespace. |
| `kubectl get pods -n <name>` | Lists pods within a specific namespace. |
| `kubectl config set-context --current --namespace=<name>` | **Pro Tip:** Sets the default namespace so you don’t need to use `-n` repeatedly. |
| `kubectl api-resources --namespaced=true` | Shows which Kubernetes resources are namespaced (e.g., Pods, Services) and which are not (e.g., Nodes, PersistentVolumes). |

## 5. Important: What is NOT in a Namespace?
Not everything in Kubernetes can be isolated by a Namespace. Some things are "Cluster-Scoped":

- Nodes: The physical servers are shared by everyone.
- PersistentVolumes: The actual storage disks.
- Namespaces themselves: You can't put a namespace inside a namespace.

### The "Stuck in Terminating" Gotcha
If you delete a Namespace (kubectl delete ns <name>), Kubernetes will delete everything inside it (Pods, Services, Secrets). Sometimes it gets stuck in "Terminating" status. This usually means there is a resource (like a networked storage volume) that refuses to let go.
