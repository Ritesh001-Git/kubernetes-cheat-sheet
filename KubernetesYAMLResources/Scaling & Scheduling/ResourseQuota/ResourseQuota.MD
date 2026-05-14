# ResourceQuota

## 1. What is a ResourceQuota?

In a shared Kubernetes cluster, a single "hungry" Pod can eat up all the CPU and RAM, starving other applications and causing a cluster-wide crash. This is known as the "Noisy Neighbor" problem.

A ResourceQuota is the solution. It is a policy that limits the total amount of resources a specific Namespace can consume.

A ResourceQuota provides constraints that limit the aggregate resource consumption in a Namespace. It can limit the total number of objects (like Pods or Services) or the total amount of compute resources (CPU/RAM).

### Why do we use it?

- Cost Control: Prevents a development team from accidentally spinning up 100 expensive LoadBalancers.
- Fairness: Ensures Team A doesn't hog all the RAM, leaving nothing for Team B.
- Stability: Prevents "fork bombs" or infinite scaling bugs from crashing the physical Nodes.

## 2. Key Benefits: Why this resource exists

### A. Preventing Resource Exhaustion
Without a quota, one Namespace could request 100% of the cluster's capacity. With a quota, the Kubernetes API will reject any request to create a new Pod if it exceeds the limit.

### B. Enforcing Best Practices
When a ResourceQuota is active in a namespace, every Pod created there MUST have resource requests and limits defined. If a developer tries to deploy a Pod without specifying how much CPU it needs, Kubernetes will reject it. This forces teams to be disciplined.

### C. Object Counting
It's not just about CPU/RAM. You can limit:

- PersistentVolumeClaims: How many disks can they ask for?
- Services: How many IPs can they take?
- ConfigMaps/Secrets: Preventing metadata bloat.

## 3. The YAML Structure (The Blueprint)

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: resource-quota
  namespace: nginx
spec:
  hard:
    pods: "10"
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "4"
    limits.memory: 8Gi
```

## 4. How it Works (The Admission Controller)
When you run kubectl apply, the Kubernetes Admission Controller checks the Namespace’s ResourceQuota.

- It calculates the current usage.
- It adds the resources requested by your new YAML.
- If the total exceeds the "hard" limit in the ResourceQuota, you get a 403 Forbidden error: "exceeded quota: team-alpha-quota, requested: pods=1, used: pods=20, limited: pods=20"

## 5. Essential ResourceQuota Commands
| Command | Purpose |
|---|---|
| `kubectl get quota -n <namespace>` | List all quotas in a namespace. |
| `kubectl describe quota <name> -n <namespace>` | The most important command. Shows exactly how much is being **Used** vs. the **Hard** limit. |
| `kubectl get pods -n <namespace>` | If pods are failing to start, check the `describe` output of the quota first. |

## 6. The "Gotcha": LimitRanges
While a ResourceQuota limits the entire Namespace, a LimitRange limits the individual Pod.

- ResourceQuota: "This whole team can only have 10 CPUs total."
- LimitRange: "Every single Pod created here must be between 0.1 CPU and 2.0 CPU."

Tech Industry Tip: Always pair your ResourceQuota with a LimitRange. This ensures that if a developer forgets to set resource limits on their Pod, the LimitRange will automatically inject "Default" values so the Pod doesn't get rejected by the ResourceQuota.
