# Taints and Tolerations

## 1. What are Taints and Tolerations?
In Kubernetes, Taints and Tolerations work together to ensure that application Pods do not land on inappropriate physical nodes.

### Taint: Applied to a Node. It marks the node so that no Pods can schedule onto it unless they explicitly "tolerate" that taint.
### Toleration: Applied to a Pod. It allows (but does not force) the Pod to schedule onto a node with a matching taint.

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
kubectl taint nodes node1 hardware=gpu:NoSchedule
```















