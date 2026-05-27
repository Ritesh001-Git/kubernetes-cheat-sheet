# Real-World Kubernetes RBAC Workflow in Companies

In real companies, engineers usually do **NOT** manually create Kubernetes users one-by-one.

Instead, organizations use:

- SSO (Single Sign-On)
- IAM systems
- Groups
- RBAC mappings

Common enterprise identity providers:

- Okta
- Azure AD
- Google Workspace
- AWS IAM
- LDAP
- GitHub Teams

Kubernetes handles:
- Authentication
- Authorization (RBAC)

But the actual user identities are managed externally.

---

# Real-World Access Flow

```text
Employee Joins Company
        │
        ▼
Identity Created in Company SSO
(Okta / Azure AD / Google Workspace)
        │
        ▼
Employee Added to Team Group
(dev-team / interns / sre / qa)
        │
        ▼
Kubernetes RBAC Bound to Group
        │
        ▼
Access Automatically Granted
```

---

# Example Company Scenario

Suppose:

- You are an Engineering Manager
- A new intern joins
- The intern should:
  - view pods
  - read logs
  - NOT delete deployments
  - access only the `intern-dev` namespace

---

# Step 1 — Create Employee Identity

The identity team creates an account:

```text
rahul@company.com
```

This account exists in:
- Okta
- Azure AD
- Google Workspace

---

# Step 2 — Add User to a Group

Example group:

```text
k8s-interns
```

Companies usually manage:
- Groups
NOT
- Individual users

because groups scale much better.

---

# Step 3 — Bind Kubernetes RBAC to the Group

## Example RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding

metadata:
  name: interns-readonly
  namespace: intern-dev

subjects:
  - kind: Group
    name: k8s-interns
    apiGroup: rbac.authorization.k8s.io

roleRef:
  kind: Role
  name: readonly-role
  apiGroup: rbac.authorization.k8s.io
```

Now everyone inside:

```text
k8s-interns
```

automatically receives access.

---

# Step 4 — Intern Uses kubectl

The intern logs in using company SSO credentials.

Example:

```bash
kubectl get pods -n intern-dev
```

Works successfully.

But:

```bash
kubectl delete deployment nginx -n intern-dev
```

Fails with:

```text
Error from server (Forbidden)
```

because RBAC blocks unauthorized actions.

---

# Common Enterprise RBAC Structure

| Team | Typical Access |
|---|---|
| Interns | Read-only namespace access |
| Developers | Deploy apps in dev/staging |
| QA | Read logs + restart pods |
| SRE / DevOps | Cluster-wide admin |
| Security Team | Audit + policy access |
| CI/CD Systems | ServiceAccounts only |

---

# Important Enterprise Rule

## Humans authenticate as:

```text
Users / Groups
```

## Applications authenticate as:

```text
ServiceAccounts
```

---

# Identity Types in Kubernetes

| Identity Type | Used For |
|---|---|
| User | Real employees |
| Group | Team-based permissions |
| ServiceAccount | Pods, automation, CI/CD |

---

# Example: CI/CD Pipeline Access

GitHub Actions or Jenkins pipelines usually authenticate using a ServiceAccount.

Example:

```yaml
serviceAccountName: github-actions-deployer
```

NOT a human user.

This improves:
- security
- access isolation
- credential management

---

# Enterprise RBAC Best Practices

## 1. Principle of Least Privilege

Never grant:

```text
cluster-admin
```

unless absolutely necessary.

Users should only receive minimum required access.

---

## 2. Namespace Isolation

Interns usually get access only to:

```text
intern-dev
```

NOT production namespaces.

---

## 3. Prefer Group-Based RBAC

Avoid:

```yaml
subjects:
- kind: User
```

Prefer:

```yaml
subjects:
- kind: Group
```

Groups are easier to maintain and scale.

---

## 4. Default to Read-Only Access

Typical intern permissions:

- get
- list
- watch

Usually NOT:
- delete
- patch
- update

---

# Typical Enterprise Lifecycle

```text
Intern joins company
        │
        ▼
Added to "k8s-interns" group in Okta
        │
        ▼
Kubernetes RBAC already mapped
        │
        ▼
Access automatically granted
        │
        ▼
Intern leaves company
        │
        ▼
Removed from group
        │
        ▼
Access automatically revoked
```

This automation is why enterprises strongly prefer:

- SSO
- IAM
- Groups
- RBAC mappings
- centralized identity management

instead of manually creating Kubernetes users.
