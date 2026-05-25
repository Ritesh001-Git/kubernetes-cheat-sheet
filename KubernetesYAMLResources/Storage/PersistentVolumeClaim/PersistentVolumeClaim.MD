# Persistent Volume Claim

## 1. What is a Persistent Volume Claim (PVC)?
A PVC is a request for storage by a user. It is very similar to a Pod. Just as a Pod consumes Node resources (CPU/RAM), a PVC consumes PV resources (Disk Space/Performance).

### The "Binding" Process
When you apply a PVC YAML, Kubernetes looks at the available PVs in the cluster. If it finds a PV that matches the size, access mode, and storage class of the claim, it "binds" them together. Once bound, that PV is exclusive to that PVC.

## 2. Key Benefits: Why this resource exists
### A. Developer Self-Service
Developers don't need to be storage experts. They don't need to know if the storage is an SSD, a spinning disk, or a cloud-based block store. They just state their requirements (e.g., "I need 50GB of Fast storage").

### B. Portability
You can take the same PVC YAML and use it on AWS, Azure, or an on-premise cluster. As long as the StorageClass name is the same, Kubernetes handles the underlying infrastructure differences.

### C. Safety
The PVC acts as a protection layer. If a Pod is deleted, the PVC remains. This ensures that the storage (and your data) isn't accidentally wiped out just because a Pod crashed or was scaled down.

## 3. The YAML Structure (The Blueprint)
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: notes-app-pvc
  namespace: notes-app
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```
### Deployment (using the PVC)

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: notes-app-deployment
  namespace: notes-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: notes-app
  template:
    metadata:
      labels:
        app: notes-app
    spec:
      containers:
      - name: notes-app
        image: ritesh1973/notes-app:latest
        ports:
        - containerPort: 8000
        volumeMounts:
        - name: notes-storage
          mountPath: /data   # 👈 inside container
      volumes:
      - name: notes-storage
        persistentVolumeClaim:
          claimName: notes-app-pvc
```

## 4. Essential PVC Commands

| Command | Purpose |
|--------|---------|
| `kubectl get pvc` | List all claims. Check the STATUS (`Pending` = looking for disk, `Bound` = success). |
| `kubectl describe pvc <name>` | Crucial for debugging. If status is `Pending`, events here explain why (e.g., "No PVs match this size"). |
| `kubectl delete pvc <name>` | Deletes the claim. Depending on the PV's reclaim policy, this may also delete the data. |
| `kubectl patch pvc <name> -p '{"spec":{"resources":{"requests":{"storage":"200Gi"}}}}'` | Expand storage. Many environments allow increasing PVC size dynamically. |

## 5. Common Issues in Production
### A. Stuck in "Pending":

- Usually means the storageClassName is misspelled or doesn't exist.
- Or, the requested size (e.g., 500Ti) is larger than what the cloud provider can give you.

### B. Multi-Node Access Error:

- If you use ReadWriteOnce and your Pod tries to move to a new Node while the old Pod is still terminating, the new Node won't be able to "grab" the disk. This is a common cause of "AttachVolume.Detach" errors.

### C. Namespace Mismatch:

- PVs are global, but PVCs are Namespaced. A Pod in the dev namespace cannot use a PVC that exists in the prod namespace.
