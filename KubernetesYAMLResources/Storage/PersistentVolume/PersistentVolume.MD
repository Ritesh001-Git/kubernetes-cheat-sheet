# Persistent Volume

## 1. What is a Persistent Volume (PV)?
A Persistent Volume is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes. It is a resource in the cluster just like a Node is a cluster resource.

Think of a PV as a physical piece of storage (like an SSD or an NFS share) that has been plugged into the cluster and is waiting for someone to use it.

### PV vs. PVC (The most important distinction)
To use storage, you need two things:

- Persistent Volume (PV): The actual disk (The "Supply").
- Persistent Volume Claim (PVC): A request for storage by a user (The "Demand").

Analogy: A PV is a physical book on a library shelf. A PVC is your library card and the request to borrow a book of a certain genre.

## 2. Key Benefits: Why this resource exists
### A. Data Persistence
If your database Pod dies and a new one starts on a different server, Kubernetes will unmount the PV from the old server and mount it to the new one. Your data is still there.

### B. Decoupling
Developers don't need to know if the storage is an AWS EBS volume, a Google Cloud Disk, or a local physical hard drive. They just ask for a "10Gi disk" (PVC), and K8s finds a matching PV.

### C. Reclaim Policies
You can decide what happens to the data when you're done with it:

= Retain: Keep the data for manual cleanup.
- Delete: Wipe the data and delete the physical disk automatically.
- Recycle: Scrub the data (rm -rf *) so the disk can be used again.

## 3. The YAML Structure (The Blueprint)
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: notes-app-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data
```

### Attaching to a Pod/Deployment
```
spec:
  containers:
    - name: mysql
      image: mysql:8.0
      volumeMounts:
      - name: mysql-persistent-storage
        mountPath: /var/lib/mysql # Where the data lives inside the container
  volumes:
    - name: mysql-persistent-storage
      persistentVolumeClaim:
        claimName: mysql-pvc # Linking the pod to the claim
```

## 4. Essential PV/PVC Commands

| Command | Purpose |
|--------|---------|
| `kubectl get pv` | List all disks. Check the STATUS (`Available`, `Bound`, `Released`). |
| `kubectl get pvc` | List all storage requests in your namespace. |
| `kubectl describe pv <name>` | See which PVC is currently "holding" this disk. |
| `kubectl get storageclass` | View storage "templates" (e.g., `standard`, `premium-ssd`). |

## 5. Understanding Access Modes
This is a frequent interview question and a critical design choice:

- ReadWriteOnce (RWO): The volume can be mounted as read-write by a single Node. (Common for databases like MySQL).
- ReadOnlyMany (ROX): The volume can be mounted read-only by many Nodes. (Good for static assets/config files).
- ReadWriteMany (RWX): The volume can be mounted as read-write by many Nodes. (Requires specialized storage like NFS or Azure Files).
