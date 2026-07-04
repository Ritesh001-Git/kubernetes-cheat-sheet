# Service

## 1. What is a Service?
A Kubernetes Service is a permanent entry point that provides a stable IP address and DNS name for a group of Pods. Since Pods are temporary and their IPs change when they restart, a Service ensures reliable communication, load-balances traffic across them, and abstracts away their changing IPs.

In Kubernetes, a Service is simply a stable way to access your Pods, even when those Pods change.

### Why do we use it?

- Service Discovery: Allows Pods to find each other using DNS names (e.g., http://my-backend-service) instead of volatile IP addresses.
- Load Balancing: If you have 10 replicas of a Pod, the Service automatically distributes incoming traffic across all of them.
- Decoupling: Your frontend doesn't need to know how many backend Pods exist; it just talks to the Service.

## 2. The Four Types of Kubernetes Services

| Type | Purpose | Use Case |
|------|--------|----------|
| `ClusterIP` (Default) | Internal IP only | Internal microservices communicating within the cluster |
| `NodePort` | Opens a specific port on every node | Quick testing or simple external access |
| `LoadBalancer` | Creates a cloud load balancer (e.g., AWS ELB, GCP LB) | Exposing your app to the internet |
| `ExternalName` | Maps a service to a DNS name (CNAME) | Connecting to external services like AWS RDS |

## 3. How it Works Internally: kube-proxy and Endpoints
When you create a Service, Kubernetes creates an Endpoint object.

- The Service watches for Pods that match its selector.
- It grabs their IP addresses and puts them into the Endpoints list.
- kube-proxy (running on every node) sees the Service and creates "rules" (using IPtables or IPVS) so that any traffic hitting the Service IP gets redirected to one of the Pod IPs.

## 3. Essential Service Commands

| Command | Purpose |
|--------|---------|
| `kubectl get svc` | List all services and their ClusterIPs / ExternalIPs. |
| `kubectl describe svc <name>` | Show details, including which Pod IPs are in the Endpoints list. |
| `kubectl get endpoints` | View the raw list of Pod IPs the service is currently tracking. |
| `kubectl port-forward svc/<name> 8080:80` | Access an internal-only service from your local machine. |

## 4. Key Benefits for Tech Companies

### A. Zero-Downtime Deployments
When a Deployment performs a Rolling Update, the Service is smart. It won't send traffic to a new Pod until that Pod's ReadinessProbe passes. Once the old Pod is deleted, the Service automatically removes it from the rotation.

### B. Internal DNS
Kubernetes runs an internal DNS server (CoreDNS). Any Pod in the cluster can reach the "order-processor-svc" simply by calling http://order-processor-svc. You don't need to manage environment variables for internal URLs.

### C. Session Affinity
If your app is "sticky" (a user needs to stay on the same Pod), you can set sessionAffinity: ClientIP. The Service will then ensure a specific user always hits the same backend Pod.
