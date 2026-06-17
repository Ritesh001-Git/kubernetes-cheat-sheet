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

## 3. What is a ServiceAccount?
While humans authenticate using certificates or single sign-on (SSO), Pods use a ServiceAccount to authenticate against the Kubernetes API.

If your application needs to dynamically look up other Pod IPs, read a ConfigMap, or trigger a backup, it cannot just run arbitrary kubectl commands. You must create a ServiceAccount, grant it permissions via a Role, and attach it to your Pod.

## 4. The YML Structure (The Blueprint)

### Step 1: Create the ServiceAccount (The Identity)
```
apiVersion: v1
kind: ServiceAccount

metadata:
  name: apache-user
  namespace: apache
```

### Step 2: Create the Role (The Permissions)
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role

metadata:
  name: apache-manager
  namespace: apache

rules:
  - apiGroups: ["","apps","batch","rbqc.authorization.k8s.io"] # ["*"] All
    resources: ["pods","services"] # ["*"] All
    verbs: ["get","list","watch","create","update","patch","delete"] # ["*"] All
```

### Step 3: Create the RoleBinding (The Connection)
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding

metadata:
  name: apache-manager-binding
  namespace: apache

subjects:
- kind: User
  name: apache-user
  namespace: apache
roleRef:
  kind: Role
  name: apache-manager
  apiGroup: rbac.authorization.k8s.io
```
## 5. How to attach the ServiceAccount to a Pod
To make your application use the identity you created, assign it inside the Deployment specification using serviceAccountName:
```
spec:
  template:
    spec:
      serviceAccountName: pod-sa-name # Pod now inherits these API powers
      containers:
      - name: user
        image: company/cleaner-script:latest
```
## 7. Essential RBAC Debugging Commands

The best way to verify permissions without breaking anything is using `kubectl auth can-i`.

| Command | Purpose |
|----------|---------|
| `kubectl auth can-i delete pods --namespace=testing` | Check if your current user can perform an action. |
| `kubectl auth can-i list secrets --as=system:serviceaccount:testing:pod-cleaner-sa -n testing` | **Security Check:** Impersonate a ServiceAccount to verify that its permissions are correctly isolated. |
| `kubectl get clusterrolebinding` | List all ClusterRoleBindings (global access bindings) in the cluster. |

### ⚠️ Security Warning: Default ServiceAccounts
Every namespace has a default ServiceAccount named default. If you don't explicitly specify a serviceAccountName in your Pod YAML, Kubernetes automatically attaches the default account. Never grant RBAC permissions directly to the default ServiceAccount, or any rogue pod deployed in that namespace will inherit those exact powers! Always create a dedicated ServiceAccount for specific tasks.
