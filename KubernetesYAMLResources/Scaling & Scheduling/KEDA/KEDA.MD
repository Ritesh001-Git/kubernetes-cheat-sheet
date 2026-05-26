# KEDA (Kubernetes Event-driven Autoscaling)

## 1. What is KEDA?
KEDA is a single-purpose, lightweight component that acts as an extension to standard Kubernetes. It sits on top of the HPA and allows you to scale workloads down to 0 replicas when there is no work to do, and scale up to $N$ replicas based on data from external event sources.

In standard Kubernetes, the Horizontal Pod Autoscaler (HPA) only scales your app based on resource utilization metrics like CPU and Memory. But in the modern tech industry, architectural workflows are event-driven.

If an application is processing jobs from a messaging queue (like Kafka, RabbitMQ, or AWS SQS), its CPU usage might be completely flat at 1%, but there could be 500,000 messages backed up waiting to be processed. Standard HPA would look at that 1% CPU and do nothing.

KEDA (Kubernetes Event-driven Autoscaling) fixes this exact problem.

### The KEDA Architecture
KEDA works by using two main roles:

- Agent: It activates and deactivates your Deployments. Critically, it allows your Pods to scale to 0 when there is zero traffic, completely saving your cloud budget. (Standard HPA cannot scale to 0).
- Metrics Adapter: It translates external metrics (like the number of unread messages in a queue) into a format that the native HPA can understand.

```text
[ External Event Source ]
          │
          ▼
   [ KEDA Operator ] ───► Scales 0 ◄──► 1
          │
          ▼
[ KEDA Metrics Adapter ] ───► [ Native K8s HPA ] ───► Scales 1 ◄──► N Pods
```

## 2. Key Benefits: Why Tech Companies Use It

### A. Scale-To-Zero (Serverless Style)
Standard HPA can only drop to 1 replica minimum. If you have 50 microservices waiting for occasional cron jobs or rare webhooks, keeping at least 1 pod running for each service wastes massive server memory. KEDA kills the pods completely and spins them up the second an event arrives.

### B. Massive Ecosystem of Scalers
KEDA knows how to talk directly to over 60 external systems out of the box. It can read:

- Databases: PostgreSQL, MySQL, MongoDB query results.
- Queues/Streams: Apache Kafka, RabbitMQ, AWS SQS, Azure Service Bus.
- Monitoring Platforms: Prometheus queries, Datadog alerts.

## 3. The YML Structure (The Blueprint)
Instead of managing an HPA resource, KEDA introduces a custom resource called a ScaledObject.

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: webapp-scaledobject
  namespace: default
spec:
  scaleTargetRef:
    name: webapp

  minReplicaCount: 1
  maxReplicaCount: 10

  triggers:
    - type: cpu
      metricType: Utilization
      metadata:
        value: "70"
```

## 4. Native HPA vs. KEDA

| Feature | Native HPA | KEDA |
|---|---|---|
| Minimum Replicas | 1 | 0 (Saves severe cloud cash) |
| Metric Focus | Internal system metrics (CPU/RAM) | External business events (Queues, DB counts, HTTP volume) |
| Primary Hook | Queries native cluster Metrics Server | Queries third-party system APIs via native adapters |
| Implementation | Built into Kubernetes core | Custom Resource Definition (CRD) addon |

## 5. Essential KEDA Commands

Because KEDA creates its own custom components inside your cluster, you get a few extra monitoring tools:

| Command | Purpose |
|---|---|
| `kubectl get scaledobject -n <namespace>` | Check the status of your event scalers and see current replica targets. |
| `kubectl get hpa` | Pro-Tip: KEDA automatically creates a standard underlying HPA under the hood. You can still see it operating here. |
| `kubectl logs -n keda deployment/keda-operator` | View KEDA's internal engine logging if your apps are refusing to scale up from zero. |

