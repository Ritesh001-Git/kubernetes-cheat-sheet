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

## 2. Key Benefits: The "Production" Features

- Rolling Updates (Zero Downtime)
When you update your app version, the Deployment doesn't kill all old Pods at once. It spins up one new Pod, waits for it to be healthy, then kills one old Pod. This continues until the whole fleet is updated. Users never see a "404 Not Found" page.

- Self-Healing
If a Node crashes and takes 5 Pods with it, the Deployment controller notices the "Actual Count" is lower than the "Desired Count" and immediately schedules 5 new Pods on healthy Nodes.

- Scaling (Stateless)
Since Deployments are for stateless apps (apps that don't care which specific Pod they are running on), scaling is as simple as changing a number. Need to handle a Black Friday sale? Change replicas: 3 to replicas: 50.

- Declarative model
You define the desired state (e.g., image version, replica count), and Kubernetes continuously reconciles the actual state to match it.

- ReplicaSet management
The Deployment creates and updates ReplicaSets to handle Pod scaling and version transitions.

- Rollback capability
Allows reverting to previous versions if a deployment fails.

## 3. The YAML Structure (The Blueprint)

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

```

#### 🔹 replicas
##### What it is:
- replicas: 3 tells the Deployment how many Pod instances should be running.

##### Why it matters:

- Ensures high availability and load distribution
- If a Pod crashes, the Deployment (via ReplicaSet) creates a new one to maintain the count
- You can scale horizontally by changing this number

#### 🔹 selector

##### What it is:
- Defines how the Deployment identifies which Pods it manages.

##### Why it is required:

- It links the Deployment → ReplicaSet → Pods
- Without a selector, Kubernetes wouldn’t know which Pods belong to this Deployment
- It must match the labels defined in the Pod template

#### 🔹 matchLabels

##### What it is:
- A simple key-value label matcher.

##### How it works:

- Matches Pods that have app: nginx in their labels
- Acts like a filter: “manage all Pods with this label”
- Note: There are also more advanced selectors (matchExpressions), but matchLabels is the simplest form.

#### 🔹 template

##### What it is:
- The blueprint for creating Pods.

##### Why it is needed:

- A Deployment doesn’t run containers directly—it creates Pods
- The template defines what each Pod should look like (containers, images, ports, etc.)
- Every replica is created from this template

## Deployment Management Commands

| Category        | Command                                                                 | Description |
|-----------------|-------------------------------------------------------------------------|-------------|
| Management      | `kubectl apply -f deployment.yaml`                                     | Create or update a Deployment from a YAML file. |
| Management      | `kubectl scale deployment payment-gateway-api --replicas=10`           | Instantly scale the number of Pod replicas. |
| Management      | `kubectl rollout status deployment/payment-gateway-api`                | Check the rollout status of a Deployment. |

## Inspection Commands

| Category        | Command                                                                 | Description |
|-----------------|-------------------------------------------------------------------------|-------------|
| Inspection      | `kubectl get deploy`                                                   | List all Deployments in the current namespace. |
| Inspection      | `kubectl rollout history deployment/payment-gateway-api`              | View rollout history and previous revisions. |
| Inspection      | `kubectl describe deploy payment-gateway-api`                         | Show detailed information (useful for debugging rollout issues). |

## Emergency Commands

| Category        | Command                                                                 | Description |
|-----------------|-------------------------------------------------------------------------|-------------|
| Emergency       | `kubectl rollout undo deployment/payment-gateway-api`                 | Roll back to the previous stable version. |
| Emergency       | `kubectl rollout restart deployment/payment-gateway-api`              | Restart all Pods (useful after ConfigMap/Secret updates). |
