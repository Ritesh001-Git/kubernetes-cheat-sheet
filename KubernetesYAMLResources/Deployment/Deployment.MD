# Deployment

## 1. What is a Deployment?
A Deployment is a Kubernetes controller that ensures a specified number of Pod replicas are running at all times and enables declarative updates , rollbacks, and scaling of applications by managing ReplicaSets according to a desired state defined by the user.
It provides "declarative updates" for your applications—you describe the desired state (e.g., "I want 3 copies of Nginx version 1.2") and the Deployment works to make the actual state match it.

### Why use it over Pods, ReplicaSets, or DaemonSets?

| Resource     | Why use a Deployment instead? |
|--------------|-------------------------------|
| Pod          | Pods are "mortal." If they die, they stay dead. Deployments automatically recreate them and maintain the desired state. |
| ReplicaSet   | ReplicaSets ensure a fixed number of Pods, but they don't handle rollouts. Updating an image won't affect existing Pods. Deployments manage seamless updates and rollbacks. |
| DaemonSet    | DaemonSets run exactly one Pod per node (useful for logging/monitoring). Deployments are designed for scalable workloads where you control the number of replicas across nodes. |
