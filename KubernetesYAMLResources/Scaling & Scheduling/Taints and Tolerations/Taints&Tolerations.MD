# Taints and Tolerations

## 1. What are Taints and Tolerations?
In Kubernetes, Taints and Tolerations work together to ensure that application Pods do not land on inappropriate physical nodes.

- Taint: Applied to a Node. It marks the node so that no Pods can schedule onto it unless they explicitly "tolerate" that taint.
- Toleration: Applied to a Pod. It allows (but does not force) the Pod to schedule onto a node with a matching taint.

Think of it like:

- Taint on Node → “Keep pods away from me”
- Toleration on Pod → “This pod is allowed to ignore that restriction”

If Node Affinity is like a magnet that attracts a Pod to a specific node, Taints are like a bug spray that repels Pods, and Tolerations are the protective gear that allows a Pod to ignore the spray.

## 2. The Three Taint Effects
When you taint a node, you must choose one of three "effects" that dictates how strict the restriction is:

- NoSchedule: The strictest rule for new workloads. If a Pod does not have a matching toleration, it cannot be scheduled onto this node. Existing running pods on the node are left alone.
- PreferNoSchedule: A "soft" version of NoSchedule. Kubernetes will try to avoid placing the Pod on this node, but if the cluster is completely out of space elsewhere, it will put it here anyway.
- NoExecute: The most aggressive effect. If you apply a NoExecute taint to a node, any currently running Pod on that node that does not tolerate it will be instantly evicted (kicked off).

## 3. Why do Tech Companies use them? (Real-World Use Cases)

### A. Dedicated Infrastructure / Special Hardware
If a company buys expensive GPU nodes for Machine Learning (ML) teams, they don't want regular Nginx frontend web pods accidentally scheduling on them and stealing memory. They taint the GPU nodes, and only give ML pods the toleration.

### B. Isolating Control Plane / Master Nodes
Kubernetes automatically taints Master nodes with node-role.kubernetes.io/control-plane:NoSchedule. This ensures your application pods don't run on the same server that manages the cluster's brain.

### C. Node Problem Eviction (Self-Healing)
If a physical node loses internet connectivity or its disk is full, the Kubernetes controller will automatically add a node.kubernetes.io/unreachable:NoExecute taint to that node. This safely flushes out all regular pods so they can restart on healthy nodes.

## The YML Structure (The Blueprint)

### Step 1: Taint the Node via CLI

##### You usually apply taints directly to the node using the command line:
```
kubectl taint nodes node1 hardware=gpu:NoSchedule
```

##### To remove the taint added by the command above, you can run:
```
kubectl taint nodes node1 hardware=gpu:NoSchedule -
```

### Step 2: Give the Pod the Toleration in the Deployment YAML

#### Deployment YAML with toleration

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpu-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: gpu-app
  template:
    metadata:
      labels:
        app: gpu-app
    spec:
      containers:
        - name: nginx
          image: nginx

      tolerations:
        - key: "dedicated"
          operator: "Equal"
          value: "gpu"
          effect: "NoSchedule"
```

### Pod YAML with toleration

```
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod
spec:
  containers:
    - name: nginx
      image: nginx

  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "gpu"
      effect: "NoSchedule"
```

## 5. Advanced Matching Operators
In your YAML, you can define how the toleration matches the taint using the operator field:

- Equal (Default): The key, value, and effect must match exactly (as shown in the YAML above).
- Exists: Matches if the key exists on the node, regardless of the value. This is useful for wildcard exceptions:

```
tolerations:
- key: "hardware"
  operator: "Exists"
  effect: "NoSchedule"
```

## 6. Essential Taint Commands

| Command | Purpose |
|---|---|
| `kubectl taint nodes <node-name> key=value:Effect` | Apply a taint to a node |
| `kubectl taint nodes <node-name> key=value:Effect-` | Remove a taint from a node *(note the `-` at the end)* |
| `kubectl describe node <node-name> \| grep Taints` | View taints currently applied to a node |

## 7. Crucial Distinction: Taints vs. Node Affinity
A common trap in interviews is confusing Node Affinity with Taints/Tolerations.

- Node Affinity says: "I want this Pod to go to that specific server." (An attraction system).
- Taints & Tolerations say: "Keep these Pods away from this server unless they are specifically allowed." (An exclusion system).

The Golden Rule: Tolerations do not guarantee a Pod will land on the tainted node. It just unlocks the door. If you want a Pod to only go to that node and nowhere else, you must use both Taints/Tolerations and Node Affinity together.
