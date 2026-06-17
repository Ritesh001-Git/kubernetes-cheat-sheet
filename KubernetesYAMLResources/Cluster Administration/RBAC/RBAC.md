# RBAC

In Kubernetes, RBAC (Role-Based Access Control) is the cluster's internal passport and security system. It regulates who (users, groups, or automated scripts) can do what (get, create, delete) to which resources (Pods, Services, Secrets).

By default, Kubernetes follows a Zero-Trust model. If you don't explicitly grant permission, access is completely blocked.

## 1. The Core Components of RBAC
RBAC is built by combining three things: The Who, The What, The Binding.

- Subjects (The Who): Can be a human user, a group, or a ServiceAccount (an application pod trying to talk to the K8s API).
- API Groups & Verbs (The What): The resources you want to access (pods, deployments, secrets) and the actions you want to take on them (get, list, watch, create, update, delete).
- Bindings (The Link): The glue that connects the "Who" to the "What."

## 2. Namespace-Level vs. Cluster-Level
Kubernetes splits resources into two categories, which means RBAC is split into two levels:

- Namespace-Level: Applies only inside a single sandbox (e.g., production or dev).
- Cluster-Level: Applies across the entire physical cluster, including global resources like Nodes, PersistentVolumes, and Namespaces.

| Level | Component (The "What") | Binding (The "Link") | Scope |
|---------|------------------------|----------------------|--------|
| Namespace | Role | RoleBinding | Restricted to 1 Namespace |
| Cluster | ClusterRole | ClusterRoleBinding | Entire Cluster (All Namespaces) |

3. What is a ServiceAccount?
While humans authenticate using certificates or single sign-on (SSO), Pods use a ServiceAccount to authenticate against the Kubernetes API.

If your application needs to dynamically look up other Pod IPs, read a ConfigMap, or trigger a backup, it cannot just run arbitrary kubectl commands. You must create a ServiceAccount, grant it permissions via a Role, and attach it to your Pod.
