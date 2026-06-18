# Helm Commands Cheat Sheet

## 1. Chart Creation & Packaging

| Command | Description | Example |
|----------|-------------|---------|
| `helm create <chart-name>` | Creates a new Helm chart scaffold with templates, values, and metadata. | `helm create apache-helm` |
| `helm lint <chart-name>` | Validates chart syntax and checks for common issues before deployment. | `helm lint apache-helm` |
| `helm package <chart-name>` | Packages a chart directory into a `.tgz` archive for distribution. | `helm package apache-helm` |
| `helm template <release-name> <chart-name>` | Renders Kubernetes manifests locally without deploying them. | `helm template dev-apache apache-helm` |

---

## 2. Chart Installation

| Command | Description | Example |
|----------|-------------|---------|
| `helm install <release-name> <chart-name>` | Installs a Helm chart into the current namespace. | `helm install dev-apache apache-helm` |
| `helm install <release-name> <chart-name> --namespace <namespace>` | Installs a chart into a specific namespace. | `helm install dev-apache apache-helm --namespace dev-apache` |
| `helm install <release-name> <chart-name> -n <namespace> --create-namespace` | Creates the namespace if it does not exist and installs the chart. | `helm install dev-apache apache-helm -n dev-apache --create-namespace` |
| `helm install <release-name> <chart-name> --dry-run --debug` | Simulates an installation and displays rendered manifests for debugging. | `helm install dev-apache apache-helm --dry-run --debug` |

---

## 3. Release Management

| Command | Description | Example |
|----------|-------------|---------|
| `helm list` | Lists all Helm releases in the current namespace. | `helm list` |
| `helm list -A` | Lists Helm releases across all namespaces. | `helm list -A` |
| `helm status <release-name>` | Displays the status of a Helm release. | `helm status dev-apache` |
| `helm uninstall <release-name>` | Removes a Helm release from the cluster. | `helm uninstall dev-apache` |
| `helm uninstall <release-name> -n <namespace>` | Removes a Helm release from a specific namespace. | `helm uninstall dev-apache -n dev-apache` |

---

## 4. Upgrade & Rollback

| Command | Description | Example |
|----------|-------------|---------|
| `helm upgrade <release-name> <chart-name>` | Upgrades an existing Helm release using updated chart files. | `helm upgrade dev-apache apache-helm` |
| `helm upgrade <release-name> <chart-name> -n <namespace>` | Upgrades a release in a specific namespace. | `helm upgrade dev-apache apache-helm -n dev-apache` |
| `helm upgrade --install <release-name> <chart-name>` | Installs the release if it doesn't exist, otherwise upgrades it. | `helm upgrade --install dev-apache apache-helm` |
| `helm history <release-name>` | Shows revision history for a release. | `helm history dev-apache` |
| `helm history <release-name> -n <namespace>` | Shows release history in a specific namespace. | `helm history dev-apache -n dev-apache` |
| `helm rollback <release-name> <revision>` | Rolls back a release to a previous revision. | `helm rollback dev-apache 1` |
| `helm rollback <release-name> <revision> -n <namespace>` | Rolls back a release in a specific namespace. | `helm rollback dev-apache 1 -n dev-apache` |

---

## 5. Values Management

| Command | Description | Example |
|----------|-------------|---------|
| `helm show values <chart-name>` | Displays default values of a chart. | `helm show values apache-helm` |
| `helm get values <release-name>` | Shows user-supplied values for a release. | `helm get values dev-apache` |
| `helm get values <release-name> -n <namespace>` | Shows values for a release in a specific namespace. | `helm get values dev-apache -n dev-apache` |
| `helm install <release-name> <chart-name> -f <values-file>` | Installs a chart using a custom values file. | `helm install dev-apache apache-helm -f values-prod.yaml` |
| `helm upgrade <release-name> <chart-name> -f <values-file>` | Upgrades a release using a custom values file. | `helm upgrade dev-apache apache-helm -f values-prod.yaml` |
| `helm upgrade <release-name> <chart-name> --set key=value` | Overrides values from the command line. | `helm upgrade dev-apache apache-helm --set replicaCount=2` |

---

## 6. Repository Management

| Command | Description | Example |
|----------|-------------|---------|
| `helm repo add <repo-name> <repo-url>` | Adds a Helm chart repository. | `helm repo add bitnami https://charts.bitnami.com/bitnami` |
| `helm repo list` | Lists configured repositories. | `helm repo list` |
| `helm repo update` | Updates repository indexes. | `helm repo update` |
| `helm search repo <keyword>` | Searches charts in configured repositories. | `helm search repo nginx` |

---

## 7. Troubleshooting & Debugging

| Command | Description | Example |
|----------|-------------|---------|
| `helm lint <chart-name>` | Validates chart structure and syntax. | `helm lint apache-helm` |
| `helm template <release-name> <chart-name>` | Renders manifests without deploying. | `helm template dev-apache apache-helm` |
| `helm install <release-name> <chart-name> --dry-run --debug` | Simulates installation with detailed debugging output. | `helm install dev-apache apache-helm --dry-run --debug` |
| `helm get manifest <release-name>` | Displays deployed Kubernetes manifests. | `helm get manifest dev-apache` |
| `helm get all <release-name>` | Displays all information about a release. | `helm get all dev-apache` |

---

## 8. Useful Kubernetes Commands with Helm

| Command | Description | Example |
|----------|-------------|---------|
| `kubectl get all` | Displays all resources in the current namespace. | `kubectl get all` |
| `kubectl get all -n <namespace>` | Displays all resources in a specific namespace. | `kubectl get all -n dev-apache` |
| `kubectl get pods` | Lists pods in the current namespace. | `kubectl get pods` |
| `kubectl get pods -n <namespace>` | Lists pods in a specific namespace. | `kubectl get pods -n dev-apache` |
| `kubectl describe pod <pod-name>` | Shows detailed pod information and events. | `kubectl describe pod dev-apache-apache-helm-xxxx` |
| `kubectl logs <pod-name>` | Displays pod logs. | `kubectl logs dev-apache-apache-helm-xxxx` |

---

## Typical Helm Workflow

```bash
# Create a chart
helm create apache-helm

# Validate the chart
helm lint apache-helm

# Render manifests locally
helm template dev-apache apache-helm

# Install the chart
helm install dev-apache apache-helm -n dev-apache --create-namespace

# Verify deployment
helm status dev-apache -n dev-apache
kubectl get all -n dev-apache

# Modify values.yaml (e.g., replicaCount: 2)

# Upgrade the release
helm upgrade dev-apache apache-helm -n dev-apache

# Check release history
helm history dev-apache -n dev-apache

# Rollback if needed
helm rollback dev-apache 1 -n dev-apache

# Uninstall
helm uninstall dev-apache -n dev-apache
```
