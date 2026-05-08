## 1. What is Ingress?
Ingress is an API object that manages external access to the services in a cluster, typically HTTP/S. It provides routing rules (like a traffic cop) to handle complex traffic patterns.
Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.
Ingress is the Front Gate and Security Guard. While a LoadBalancer service creates one external IP for one service, an Ingress allows you to route traffic to dozens of different services using a single IP address, based on the URL path or domain name.

![Ingress Architecture](images/k8s-architecture.png)

### Why do we use it?

- Cost Efficiency: Instead of paying for 10 LoadBalancer services (which cost money per hour on AWS/GCP), you pay for one Ingress Controller that routes to 10 internal ClusterIP services.
- Path-Based Routing: [example.com/api](https://example.com/api) goes to the Backend, while [example.com/](https://example.com/) goes to the Frontend.
- Host-Based Routing: app.v1.com and app.v2.com can be handled by the same IP.
- SSL/TLS Termination: You can manage your SSL certificates in one place (at the Ingress level) rather than inside every individual application.

## 2. The Two Parts of Ingress
Ingress is unique because the "Resource" (the YAML you write) does nothing by itself. You need a Controller.

- Ingress Resource: The YAML file where you define your rules (The "Map").
- Ingress Controller: The actual software (usually an Nginx, HAProxy, or Traefik pod) that reads the rules and moves the traffic (The "Engine").

## 3. The YAML Structure (The Blueprint)
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80

```
## 4. Key Benefits: Why this resource exists

### A. Centralized SSL Management
Instead of every developer having to figure out how to load an SSL certificate into their Python or Go code, the Ingress Controller handles the "Handshake." The traffic is encrypted from the User to the Ingress, and then usually unencrypted (and fast) inside the cluster.

### B. Canary and Blue/Green Deployments
Advanced Ingress Controllers (like Nginx or Istio) allow you to say: "Send 90% of traffic to the old version and 10% to the new version." This is how big tech companies test new features safely.

### C. Authentication
You can configure Ingress to ask for a password (Basic Auth) or check an OAuth token before the traffic even reaches your application pods.

## 5. Essential Ingress Commands

| Command | Purpose |
|---|---|
| `kubectl get ingress` | List all Ingress rules. Look for the `ADDRESS` column to find your public IP/DNS. |
| `kubectl describe ingress <name>` | See exactly which paths are mapped to which services and whether the SSL certificate is valid. |
| `kubectl logs -n ingress-nginx <nginx-pod-name>` | Crucial for debugging. View actual traffic logs and identify `404`/`500` errors. |

## 6. Comparison: Service vs. Ingress

| Feature | LoadBalancer Service | Ingress |
|---|---|---|
| Layer | Layer 4 (TCP/UDP) | Layer 7 (HTTP/HTTPS) |
| IP Addresses | One per Service | One for many Services |
| Path Routing | No | Yes (e.g., `/shop`, `/blog`) |
| SSL Termination | Hard to manage | Built-in / Easy |

### Summary

- Pod: The app.
- Service: The internal stable address for the app.
- Ingress: The gateway that lets the world access the app via a URL.
