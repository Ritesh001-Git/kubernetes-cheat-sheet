# Horizontal Pod Autoscaler

## 1. What is an HPA?
An HPA is a controller that adjusts the shape of your replication controllers (Deployments or ReplicaSets) by adding or removing Pods.

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

3. The Prerequisite: You MUST define ResourcesAn
HPA cannot work unless you tell it what "$100\%$ usage" means. If your Deployment does not have resources.requests defined on its containers, the HPA will fail and stay in an <unknown> state.
