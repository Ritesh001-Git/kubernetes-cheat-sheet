# Helm
If Kubernetes resources (Deployments, Services, Ingress, ConfigMaps) are the individual ingredients, Helm is the Recipe Book and the Chef.

In the tech industry, nobody manually runs kubectl apply -f on 50 different YAML files to deploy an application. Instead, we use Helm, which is the official package manager for Kubernetes (essentially the apt, yum, or npm of the cloud-native world).

## 1. What is Helm?
Helm allows you to package all of your Kubernetes YAML files into a single, reusable folder called a Chart. It then templates those files so you can dynamically change values (like image versions, replicas, or passwords) depending on whether you are deploying to Dev, Staging, or Production.

### Core Concepts:
- Chart: A blueprint bundle of Kubernetes resources (a package).
- Values: A configuration file (values.yaml) containing the variables that customize the template.
- Release: A specific instance of a Chart running inside your cluster. For example, you can deploy the "WordPress" chart twice, creating a blog-release and a shop-release.

## 2. The Anatomy of a Helm Chart
When you create a new Helm chart, it generates a standard directory structure:
```
Plaintext
my-app-chart/
├── Chart.yaml          # Metadata about your chart (Name, version, description)
├── values.yaml         # Default configuration variables for your templates
├── templates/          # The actual Kubernetes YAML files with template markers
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── charts/             # Sub-charts (Dependencies that your app relies on)
```

## 3. How Templating Works (Real-World Setup)
Instead of hardcoding a specific number of replicas or an image tag inside your deployment, you use Go templating syntax {{ .Values... }}.

#### The Template: templates/deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-api
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: web
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

#### The Variables: values.yaml
This is the file developers edit when configuring their app:

```
replicaCount: 3

image:
  repository: company/payment-api
  tag: v2.4.1
```
## 4. Key Benefits: Why Tech Companies Use It

### A. One-Click Upgrades and Rollbacks
If a deployment fails or crashes your application, Helm tracks the entire history of that release. You can revert to a perfectly healthy previous state instantly with a single command.
`
helm rollback my-app-release 1
`

### B. Avoid YAML Duplication
Instead of maintaining a separate copy of your YAML files for dev, staging, and prod, you keep one single Chart. You simply feed it different environment files:
```
helm install payment-prod ./my-app-chart -f values-prod.yaml
helm install payment-dev ./my-app-chart -f values-dev.yaml
```

### C. Public Repositories (Artifact Hub)
Don't write charts from scratch for common tools. If you need to install complex software like Prometheus, Grafana, Jenkins, or MongoDB, you can pull production-ready charts built by the community from the global registry (Artifact Hub).
When you run Helm, it automatically substitutes the values from values.yaml into the template placeholders and generates valid Kubernetes configurations.

## 7. Helm Architecture Evolution (Helm 2 vs Helm 3)
If you read older documentation online, you will hear a lot about a component called Tiller.

- Helm 2 (Old): Used a server-side pod called Tiller that ran inside your cluster with full admin rights. It was a massive security nightmare because anyone who could talk to Tiller could hijack the entire cluster.

- Helm 3 (Modern): Completely removed Tiller. It is now a pure client-side command-line tool. It uses your local kubeconfig authentication (RBAC) to securely talk straight to the Kubernetes API server.

Helm is the standard foundation for modern GitOps tools like ArgoCD. Everything you deploy at scale in a real-world engineering team will almost certainly pass through a Helm template engine!
