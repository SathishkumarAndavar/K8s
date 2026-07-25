# CKA Hands-On Tasks

This document provides step-by-step guides for common, hands-on tasks that are essential for the CKA exam and real-world Kubernetes administration. Each task includes manifests, commands, and verification steps.

## 1. Task: Configure a Pod with Persistent Storage

**Goal:** Create a PersistentVolumeClaim (PVC) and a Pod that mounts it to persist data.

**Context:** Applications often need storage that survives pod restarts. This task demonstrates the standard pattern of dynamically provisioning storage using a `StorageClass` and a `PersistentVolumeClaim`.

### Steps:

**1. Define the PersistentVolumeClaim:**

This manifest requests 1Gi of storage from the `standard` storage class.

**`pvc-task.yaml`**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard # Note: Your cluster must have a StorageClass named 'standard' or you must change this.
  resources:
    requests:
      storage: 1Gi
```

**2. Define the Pod that uses the PVC:**

This Pod mounts the volume claimed by `task-pvc` at the path `/usr/share/nginx/html`.

**`pod-task.yaml`**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: storage-pod
spec:
  containers:
  - name: web-server
    image: nginx
    ports:
      - containerPort: 80
    volumeMounts:
    - name: storage
      mountPath: /usr/share/nginx/html
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: task-pvc
```

**3. Apply and Verify:**

```sh
# Create the PVC first
kubectl apply -f pvc-task.yaml

# Verify the PVC is created and becomes 'Bound'
# The status will be 'Pending' until the storage is provisioned.
kubectl get pvc task-pvc -w

# Once Bound, create the Pod
kubectl apply -f pod-task.yaml

# Verify the Pod is running
kubectl get pod storage-pod -w

# Write a file to the persistent volume from within the Pod
kubectl exec -it storage-pod -- sh -c "echo 'Hello from persistent storage' > /usr/share/nginx/html/index.html"

# Verify the file is there
kubectl exec -it storage-pod -- cat /usr/share/nginx/html/index.html

# Delete the pod
kubectl delete pod storage-pod

# Recreate the pod
kubectl apply -f pod-task.yaml

# Verify the data still exists after the pod is recreated
kubectl exec -it storage-pod -- cat /usr/share/nginx/html/index.html
```

### Cleanup:

```sh
kubectl delete pod storage-pod
kubectl delete pvc task-pvc
```

---

*This task covers concepts from the "Manage Storage" section of the Kubernetes Tasks documentation.*