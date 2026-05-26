# Vertical Pod Autoscaler

## 1. What is a VPA?
A VPA is a controller that automatically sets container Resource Requests and Limits based on historical usage.

### The Problem it Solves
Developers usually guess their container limits. They often allocate $2\text{ Gi}$ of RAM to a Python app that only uses $200\text{ MiB}$. This wastes massive amounts of cloud budget (Over-provisioning). Conversely, under-provisioning causes out-of-memory (OOMKilled) crashes. VPA finds the absolute "sweet spot."
