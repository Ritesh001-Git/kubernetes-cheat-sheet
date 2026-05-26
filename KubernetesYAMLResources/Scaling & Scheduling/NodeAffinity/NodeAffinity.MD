# Node Affinity

## 1. What is Node Affinity?
Node Affinity is a set of scheduling rules that tells the Kubernetes scheduler exactly how to match your Pods to the labels on your physical or virtual Nodes.

It is the advanced, modern evolution of an older feature called nodeSelector. While nodeSelector only allowed hard, exact matches, Node Affinity introduces complex logic operators (like In, NotIn, Exists, DoesNotExist) and "soft/flexible" rules.

It allows you to say: "I want this specific application to run only on our high-performance SSD machines," or "Keep this frontend pod inside the US-East availability zone."

## 2. The Two Types of Node Affinity
Kubernetes breaks Node Affinity down into two categories: the Hard rules (Must fulfill) and the Soft rules (Best effort).

### A. Hard Rules: requiredDuringSchedulingIgnoredDuringExecution

- The Logic: This is a strict requirement. If Kubernetes cannot find a node that matches this criteria, your Pod will fail to schedule and sit in a Pending state forever.
- Real-World Use Case: A video rendering application must have a physical GPU node to function. Running it on a standard CPU node is completely pointless.

### B. Soft Rules: preferredDuringSchedulingIgnoredDuringExecution
- The Logic: This is a preference or a "wishlist." Kubernetes will try its absolute best to find a matching node. However, if all preferred nodes are full or offline, it will gracefully fall back and schedule the Pod on a regular node anyway.
- Real-World Use Case: You prefer your web application to run in availability zone us-east-1a to keep network latency low, but if that zone goes down, running it in us-east-1b is perfectly acceptable.

**What does "IgnoredDuringExecution" mean?**
Both types end with this phrase. It means that if a node's labels change while your Pod is already running on it, Kubernetes won't kick your Pod off. It only evaluates these rules during the initial scheduling phase.

## 3. The YML Structure (The Blueprint)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: disktype
                    operator: In
                    values:
                      - ssd

      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

## 4. Logical Operators Supported

| Operator | Action |
|---|---|
| `In` | Node label value must match one of the items in your list (Most common). |
| `NotIn` | Node label must not be in your list (Great for avoiding old/legacy nodes). |
| `Exists` | The label key must exist on the node. You don't care what the actual value is. |
| `DoesNotExist` | The label key must be entirely absent from the node. |
| `Gt` / `Lt` | Greater than / Less than (Used for numeric comparisons, like kernel versions). |

---

## 5. Essential Node Labeling Commands

Node Affinity depends completely on your nodes having accurate labels. Here is how you manage them:

| Command | Purpose |
|---|---|
| `kubectl get nodes --show-labels` | View all existing labels on every node in your cluster. |
| `kubectl label nodes <node-name> storage-type=ssd` | Apply a label: Add a key-value pair to a physical machine. |
| `kubectl label nodes <node-name> storage-type-` | Remove a label: Note the minus (`-`) sign at the end of the key. |

6. Reminder: Taints vs. Affinity

- Node Affinity draws a Pod toward a Node.
- Taints push a Pod away from a Node.

If you have a dedicated machine learning cluster, you must use both systems simultaneously: Taint the node so regular web applications don't accidentally schedule there, and apply Node Affinity to your ML app so it is actively pulled to that specific GPU node.
