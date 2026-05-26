# Vertical Pod Autoscaler

## 1. What is a VPA?
A VPA is a controller that automatically sets container Resource Requests and Limits based on historical usage.

### The Problem it Solves
Developers usually guess their container limits. They often allocate $2\text{ Gi}$ of RAM to a Python app that only uses $200\text{ MiB}$. This wastes massive amounts of cloud budget (Over-provisioning). Conversely, under-provisioning causes out-of-memory (OOMKilled) crashes. VPA finds the absolute "sweet spot."

## 2. The Three Operation Modes
You can configure a VPA to operate in different modes depending on how much control you want to give it:

- Off (Recommendation Mode): The safest mode. VPA does absolutely nothing to your running pods. It just observes them and logs its recommendations in the YAML. You can look at them and apply them manually later.

- Initial: VPA only assigns resources when a Pod is first created. It will not touch or restart a pod that is already running.

- Auto / Recreate: The fully automated mode. If VPA detects a pod needs more memory, it will evict (kill) the running Pod, rewrite the Deployment YAML with the new limits, and let Kubernetes recreate it.

⚠️ The Big Catch: Because changing a container's size requires mutating the Pod specification, the Auto mode must restart the Pod to apply changes.

## 3. The YML Structure (The Blueprint)
```
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler

metadata:
  name: apache-vpa
  namespace: apache

spec:
  targetRef:
    apiVersion: "apps/v1"
    name: apache-deployment
    kind: Deployment
  
  updatePolicy:
    updateMode: "Auto"
```

## 4. Essential VPA Commands

| Command | Purpose |
|---|---|
| `kubectl get vpa` | List all VPAs in the namespace. |
| `kubectl describe vpa <name>` | The primary command. Scroll to the bottom to see the **Recommendation** block with the exact values VPA calculated. |

### Reading a VPA Recommendation
When you describe a VPA, it gives you four specific metrics:

- Lower Bound: The bare minimum your app needs. If you drop below this, it will likely degrade or fail.
- Target: The recommended sweet spot. This is what VPA configures your container to use.
- Uncapped Target: What the app wants to use if you hadn't set the maxAllowed safety guardrails.
- Upper Bound: The highest peak VPA expects your app to hit based on recent spikes.

## 5. HPA vs. VPA: The Ultimate Battle

| Feature | HPA | VPA |
|---|---|---|
| Scaling Axis | Horizontal (More instances) | Vertical (Larger instances) |
| Downtime Risk | Zero (Pods are added gracefully) | Possible (Pods must restart in Auto mode) |
| Best For | Stateless traffic targets (Web APIs) | Stateful/Single-replica tasks (DBs, Workers) |
| Metric Source | Live metric spikes (Current traffic) | Historical data trends (Over days/weeks) |

### 🚨 Crucial Architecture Warning: Don't Mix Both!
You cannot run HPA and VPA on the exact same resource if they are both checking the same metrics (like CPU/Memory).

If you do, they will fight each other in a destructive loop:
- Traffic spikes, so HPA adds more Pods to share the load.
- Because the load is shared, the CPU usage per Pod drops.
- VPA sees the low CPU usage and shrinks the individual Pod sizes.
- The smaller Pods instantly choke on the traffic, causing HPA to panic and spawn even more Pods.

The Exception: You can pair them if HPA scales based on a custom event (like a Kafka queue lag or network traffic) while VPA manages the baseline CPU/Memory constraints.
