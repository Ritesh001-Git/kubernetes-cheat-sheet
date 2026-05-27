# RBAC (Role-Based Access Control)

## 1. What is RBAC?

In Kubernetes, RBAC (Role-Based Access Control) is the cluster's internal passport and security system. It regulates who (users, groups, or automated scripts) can do what (get, create, delete) to which resources (Pods, Services, Secrets).

By default, Kubernetes follows a Zero-Trust model. If you don't explicitly grant permission, access is completely blocked.

## 2. The Core Components of RBAC
RBAC is built by combining three things: The Who, The What, The Binding.

- Subjects (The Who): Can be a human user, a group, or a ServiceAccount (an application pod trying to talk to the K8s API).
- API Groups & Verbs (The What): The resources you want to access (pods, deployments, secrets) and the actions you want to take on them (get, list, watch, create, update, delete).
- Bindings (The Link): The glue that connects the "Who" to the "What."

## 3. Namespace-Level vs. Cluster-Level
Kubernetes splits resources into two categories, which means RBAC is split into two levels:

- Namespace-Level: Applies only inside a single sandbox (e.g., production or dev).
- Cluster-Level: Applies across the entire physical cluster, including global resources like Nodes, PersistentVolumes, and Namespaces.

```text
                           +--------+
                           |  RBAC  |
                           +--------+
                                |
                             Access
                                |
                +---------------+---------------+
                |                               |
                ▼                               ▼
      +------------------+           +------------------+
      |  Service Account |           |       User       |
      +------------------+           +------------------+
                |                               |
                ▼                               ▼
        +---------------+              +----------------+
        |   Namespace   |              |    Cluster     |
        +---------------+              +----------------+
                |                               |
        +-------+-------+               +-------+--------+
        |               |               |                |
        ▼               ▼               ▼                ▼
   +---------+   +-------------+   +-------------+   +------------------+
   |  Roles  |   | RoleBinding |   | ClusterRole |   | ClusterRoleBinding |
   +---------+   +-------------+   +-------------+   +------------------+
```

### RBAC Components Comparison

| Level | Component (The "What") | Binding (The "Link") | Scope |
|---|---|---|---|
| Namespace | `Role` | `RoleBinding` | Restricted to 1 Namespace |
| Cluster | `ClusterRole` | `ClusterRoleBinding` | Entire Cluster (All Namespaces) |

## 4. What is a ServiceAccount?
While humans authenticate using certificates or single sign-on (SSO), Pods use a ServiceAccount to authenticate against the Kubernetes API.

If your application needs to dynamically look up other Pod IPs, read a ConfigMap, or trigger a backup, it cannot just run arbitrary kubectl commands. You must create a ServiceAccount, grant it permissions via a Role, and attach it to your Pod.

## 5. Namespace-Level RBAC (Real-World Setup)
Imagine you have an automation script running inside a Pod that needs to list and delete Pods inside the testing namespace only.

### Step 1: Create the ServiceAccount (The Identity)
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pod-cleaner-sa
  namespace: testing
```

### Step 2: Create the Role (The Permissions)
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: testing # Strict Namespace boundaries
  name: pod-manager-role
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "list", "delete"] # No 'create' allowed!
```

### Step 3: Create the RoleBinding (The Connection)
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: bind-cleaner-to-role
  namespace: testing
subjects:
- kind: ServiceAccount
  name: pod-cleaner-sa
  namespace: testing
roleRef:
  kind: Role
  name: pod-manager-role
  apiGroup: rbac.authorization.k8s.io
```

## 6. Cluster-Level RBAC (Real-World Setup)
Now imagine you are installing a cluster-wide monitoring system like Prometheus. Prometheus needs to scrape metrics from every node and every pod in every namespace across the entire cluster. A standard Role won't work because it is trapped in one namespace. We need a ClusterRole.

### Step 1: Create the ClusterRole (Global Permissions)
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # Notice: NO namespace field here! ClusterRoles are global.
  name: global-metrics-reader
rules:
- apiGroups: [""]
  resources: ["nodes", "pods", "services"]
  verbs: ["get", "list", "watch"]
```

### Step 2: Create the ClusterRoleBinding (Global Connection)
Even if your ServiceAccount lives inside a specific monitoring namespace, a ClusterRoleBinding elevates its powers globally.
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: bind-prometheus-globally
subjects:
- kind: ServiceAccount
  name: prometheus-sa
  namespace: monitoring # The identity lives here...
roleRef:
  kind: ClusterRole
  name: global-metrics-reader # ...but its power extends everywhere!
  apiGroup: rbac.authorization.k8s.io
```
### 6. How to attach the ServiceAccount to a Pod
To make your application use the identity you created, assign it inside the Deployment specification using serviceAccountName:
```
spec:
  template:
    spec:
      serviceAccountName: pod-cleaner-sa # Pod now inherits these API powers
      containers:
      - name: worker
        image: company/cleaner-script:latest
```

## 8. Essential RBAC Debugging Commands

The absolute best way to verify permissions without breaking things is the `auth can-i` macro tool:

| Command | Purpose |
|---|---|
| `kubectl auth can-i delete pods --namespace=testing` | Check if your current user can perform an action. |
| `kubectl auth can-i list secrets --as=system:serviceaccount:testing:pod-cleaner-sa -n testing` | The #1 Security Check: Spoof/impersonate a ServiceAccount to test if its access is properly isolated. |
| `kubectl get clusterrolebinding` | List all global access bindings in the cluster. |

### ⚠️ Security Warning: Default ServiceAccounts
Every namespace has a default ServiceAccount named default. If you don't explicitly specify a serviceAccountName in your Pod YAML, Kubernetes automatically attaches the default account. Never grant RBAC permissions directly to the default ServiceAccount, or any rogue pod deployed in that namespace will inherit those exact powers! Always create a dedicated ServiceAccount for specific tasks.
