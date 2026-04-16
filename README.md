# 🚀 Kubernetes Cheat Sheet

---

## 🔹 What is Kubernetes?

### 📌 Definition

Kubernetes is an open-source container orchestration platform used to automate the deployment, scaling, and management of containerized applications.

### ⚙️ Key Points

* Developed by Google, now maintained by CNCF
* Works with container tools like Docker
* Handles scaling, load balancing, and self-healing
* Declarative configuration using YAML

### 🧠 Explanation

Instead of manually managing containers across multiple machines, Kubernetes automates everything—ensuring your application runs reliably even if nodes fail.

Container orchestration is the automated management of the entire lifecycle of containerized applications, including deployment, scaling, networking, and health monitoring.

---

## 🔹 Why Do We Use Kubernetes?

### 📌 Definition

Kubernetes is used to efficiently manage containerized applications at scale by automating deployment, scaling, networking, and availability.

### ⚙️ Key Benefits

* Auto-scaling (Horizontal Pod Autoscaling)
* Self-healing (restarts failed containers)
* Load balancing
* Rolling updates & rollbacks
* Service discovery

### 🧠 Real-World Use Case

In production, applications run across multiple servers. Kubernetes ensures:

* Zero downtime deployments
* High availability
* Efficient resource utilization

---

## 🔹 Monolithic vs Microservices Architecture

### 📌 Monolithic Architecture

A monolithic application is built as a single, tightly coupled unit where all components are interconnected.

### 📌 Microservices Architecture

A microservices architecture breaks an application into small, independent services that communicate over APIs.

### ⚖️ Key Differences

| Feature         | Monolithic          | Microservices                   |
| --------------- | ------------------- | ------------------------------- |
| Structure       | Single codebase     | Multiple independent services   |
| Scalability     | Hard to scale       | Easy to scale individually      |
| Deployment      | Entire app deployed | Services deployed independently |
| Fault Isolation | Poor                | High                            |
| Flexibility     | Low                 | High                            |

### 🧠 Interview Tip

Kubernetes is mainly designed for **microservices architecture**, where multiple services need orchestration.

---

# 🏗️ Kubernetes Architecture

## 🔹 High-Level Overview

* Kubernetes cluster consists of:

  * **Control Plane (Master Node)** → Manages the cluster
  * **Worker Nodes** → Run applications (Pods)

### 📌 Key Concept

> "In Kubernetes, the control plane manages the cluster state, while worker nodes run the actual application workloads."

---

## 🔹 Control Plane (Master Node)

### 📌 Definition

The control plane is responsible for managing the overall state of the Kubernetes cluster, making decisions like scheduling, scaling, and maintaining desired state.

---

### ⚙️ Components of Control Plane

## 1. kube-apiserver

### 📌 Definition

The API Server is the central communication hub of Kubernetes. All components interact with the cluster through the API server.

### ⚙️ Key Points

* Exposes Kubernetes API (REST)
* Handles authentication & authorization
* Acts as entry point to cluster

---

## 2. etcd

### 📌 Definition

etcd is a distributed key-value store that stores all cluster data and state.

### ⚙️ Key Points

* Stores configuration data
* Critical for cluster consistency
* Backup of etcd = backup of cluster

---

## 3. kube-scheduler

### 📌 Definition

The scheduler assigns pods to appropriate worker nodes based on resource availability and constraints.

### ⚙️ Key Points

* Considers CPU, memory, affinity rules
* Chooses best node for pod placement

---

## 4. kube-controller-manager

### 📌 Definition

it runs continuously and watches the actual and desired state of objects. If there is a difference in the actual and desired state, it ensures that the kubernetes resource/object is in the desired state.

### ⚙️ Key Controllers

* Node Controller
* Replication Controller
* Endpoint Controller

### 🧠 Example

If a pod crashes, controller ensures a new one is created.

---

## 5. cloud-controller-manager

### 📌 Definition

When kubernetes is deployed in cloud environments, the cloud controller manager acts as a bridge between Cloud Platform APIs and the Kubernetes cluster.

This way, the core kubernetes core components can work independently and allow the cloud providers to integrate with kubernetes using Cloud Controller binaries.
### ⚙️ Key Points

* Manages cloud resources
* Handles Load Balancers, storage, nodes
* Separates cloud-specific logic from core Kubernetes

---

## 🔹 Worker Node

### 📌 Definition

Worker nodes are the machines where containerized applications (pods) are actually deployed and run.

---

### ⚙️ Components of Worker Node

## 1. kubelet

### 📌 Definition

The kubelet is an agent that runs on each worker node and ensures containers are running as expected.

### ⚙️ Key Points

* Communicates with API server
* Monitors pod health
* Ensures desired state is maintained

---

## 2. kube-proxy

### 📌 Definition

kube-proxy manages networking rules on nodes and enables communication between services and pods.

### ⚙️ Key Points

* Handles service networking
* Implements load balancing
* Uses iptables or IPVS

---

## 3. Container Runtime

### 📌 Definition

The container runtime is responsible for running containers. It is responsible for pulling images from container registries, running containers, allocating and isolating resources for containers, and managing the entire lifecycle of a container on a host.

### ⚙️ Examples

* containerd
* CRI-O
* Docker (deprecated but still used)

---

## 🔹 CNI (Container Network Interface)

### 📌 Definition

CNI is a networking plugin that enables communication between pods across different nodes in a Kubernetes cluster.

### ⚙️ Key Points

* Provides pod-to-pod communication
* Works across nodes
* Assigns IP addresses to pods

### ⚙️ Popular CNI Plugins

* Flannel
* Calico
* Weave Net

---

## 🔹 Communication Flow

* External systems → communicate via **API Server (HTTPS)**
* Control plane components → communicate via **mTLS**
* Worker nodes ↔ Control plane → via API Server
* Pods communicate via **CNI network**

---

## 🔹 One-Line Summary (Must Remember)

> "Kubernetes follows a master-worker architecture where the control plane manages the cluster state, and worker nodes execute containerized applications using components like kubelet, kube-proxy, and container runtime."

---

## 🔹 Pro Interview Tips

* Always mention **API Server is the heart of Kubernetes**
* Highlight **etcd as the single source of truth**
* Emphasize **desired state vs current state**
* Use terms like:

  * *declarative model*
  * *self-healing*
  * *scalability*
  * *high availability*

---

## Kubernetes Universal Workflow

The universal workflow of Kubernetes—whether you are deploying a simple Pod, a complex Deployment, a StatefulSet, or a CronJob—is governed by a single architectural pattern: the Asynchronous Control Loop (Reconciliation).

In Kubernetes, you never "execute" a command; you "record a desire." Here is the universal internal flow for any resource.

---

### 1. The "Submission" Phase (Persisting the Intent)
No matter the resource type (Deployment, DaemonSet, etc.), the first step is always the same.

- Request: You send a YAML to the API Server via kubectl.
- Validation: The API Server checks if the syntax is correct and if the resource version is supported.
- The Ledger: The API Server writes the entry into etcd.
  - Status: At this point, the resource is "Born" but "Idle." It is just a row in a database.

---

### 2. The "Controller" Phase (The Logic Loop)
This is where resource-specific logic happens. Every resource type has a specific Controller inside the Controller Manager that "watches" for its type.

- For a Deployment: The Deployment Controller sees the record, creates a ReplicaSet, and then the ReplicaSet Controller creates Pods.
- For a Job/CronJob: The Job Controller sees the record and creates a Pod that is designed to exit upon completion.
- For a DaemonSet: The DaemonSet Controller ensures exactly one Pod is created for every eligible node in the cluster.
- For PersistentVolumeClaims (PVC): The PV-Controller looks for a matching PersistentVolume (PV) and "binds" them together.

Universal Rule: The Controller constantly compares Desired State (what you wrote in YAML) vs. Actual State (what is happening in the cluster). If they don't match, it creates a "Work Item" to bridge the gap.

---

### 3. The "Scheduling" Phase (Placement)
Once the controllers have broken down high-level objects (like Deployments) into the smallest unit—the Pod—the Scheduler steps in.

- The Watch: The Scheduler watches the API Server for any Pod that has a nodeName field that is empty.
- nFiltering/Scoring: It looks at all worker nodes. It filters out nodes that are full or don't match labels (Affinity).
- The Binding: It picks the best node and sends a "Bind" request to the API Server, which updates the Pod's definition in etcd.

---

### 4. The "Node" Phase (Physical Execution)
Now the master components are done. The work moves to the specific Worker Node identified by the Scheduler.

- Kubelet Recognition: The Kubelet on that node sees a Pod assigned to it via the API Server.
- CRI (Container Runtime): Kubelet tells the runtime (e.g., containerd) to pull the image and start the container.
- CNI (Networking): The network plugin assigns the Pod an IP.
- CSI (Storage): If it's a StatefulSet or using a PVC, the Storage Interface mounts the physical disk to the node and then into the container.

---

### 5. The "Feedback" Phase (Status Update)
The Kubelet monitors the health of the container.

- It reports the status (Running, Succeeded, Failed) back to the API Server.
- The API Server updates etcd.
- The Controllers see the update. If the Pod failed and it's part of a Deployment, the loop starts all over again at Step 2 to recreate it.
