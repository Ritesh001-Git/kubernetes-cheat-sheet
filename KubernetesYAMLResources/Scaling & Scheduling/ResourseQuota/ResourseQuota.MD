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

## 4. How it Works (The Admission Controller)
When you run kubectl apply, the Kubernetes Admission Controller checks the Namespace’s ResourceQuota.

- It calculates the current usage.
- It adds the resources requested by your new YAML.
- If the total exceeds the "hard" limit in the ResourceQuota, you get a 403 Forbidden error: "exceeded quota: team-alpha-quota, requested: pods=1, used: pods=20, limited: pods=20"

Command	Purpose
kubectl get quota -n <namespace>	List all quotas in a namespace.
kubectl describe quota <name> -n <namespace>	The most important command. Shows exactly how much is being Used vs. the Hard limit.
kubectl get pods -n <namespace>	If pods are failing to start, check the describe output of the quota first..
