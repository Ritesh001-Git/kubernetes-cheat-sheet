# Horizontal Pod Autoscaler

## 1. What is an HPA?
An HPA is a controller that adjusts the shape of your replication controllers (Deployments or ReplicaSets) by adding or removing Pods.

The Horizontal Pod Autoscaler (HPA) is the cluster's automatic autopilot. It monitors your pods and automatically increases or decreases the number of replicas based on real-time resource usage.

- Horizontal Scaling (Scaling Out): Adding more Pods of the same type (e.g., going from 2 Pods to 10 Pods). This is what HPA does.
- Vertical Scaling (Scaling Up): Making an existing Pod bigger by feeding it more CPU/RAM. (This is handled by a different tool called VPA).

## 2. How it Works Under the Hood
The HPA operates on a continuous control loop (checking by default every 15 seconds).

`[ Pod Metrics ] ---> [ Metrics Server ] ---> [ HPA Controller ] ---> [ Scaled Deployment ]`

- kubelet collects CPU and RAM metrics from the containers.
- The Metrics Server (a cluster-wide add-on) aggregates these metrics.
- The HPA Controller constantly queries the Metrics Server API.
- If the average metric exceeds your target, it calculates the new number of replicas using this core formula:

```math
\text{Desired Replicas} = \left\lceil \text{Current Replicas} \times \frac{\text{Current Metric Value}}{\text{Target Metric Value}} \right\rceil
```

## 3. The Prerequisite: You MUST define Resources
An HPA cannot work unless you tell it what "$100\%$ usage" means. If your Deployment does not have resources.requests defined on its containers, the HPA will fail and stay in an <unknown> state.

## The YML Structure (The Blueprint)

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler

metadata:
  name: apache-hpa
  namespace: apache

spec:
  scaleTargetRef:
    kind: Deployment
    name: apache-deployment
    apiVersion: apps/v1
  
  minReplicas: 1
  maxReplicas: 5

  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
```

## 5. Advanced Production tuning: "Cooldown" Windows
A major issue in high-traffic environments is Flapping—where traffic spikes for 5 seconds, HPA scales up 10 pods, traffic drops for 5 seconds, HPA kills them, and then traffic spikes again. This destroys performance.

Modern HPAs use Behavior Policies to control how fast it scales down:
```
spec:
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300 # Wait 5 minutes after a spike before killing pods
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
```

## 6. Essential HPA Commands

| Command | Purpose |
|---|---|
| `kubectl get hpa` | List your HPAs. Check the **TARGETS** column (should show `Current% / Target%`). |
| `kubectl describe hpa <name>` | Best for debugging. Look at the events at the bottom to see when and why it scaled. |
| `kubectl autoscale deployment web --min=2 --max=10 --cpu-percent=80` | Quick shortcut: Create an HPA instantly from the CLI without a YAML file. |

