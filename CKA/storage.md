# CKA Storage — Answers and Practice

## 1. Provision and verify a PVC

```sh
kubectl get storageclass
kubectl apply -f pvc.yaml
kubectl get pvc,pv
kubectl describe pvc data
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data
spec:
  storageClassName: fast
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 5Gi
```

The PVC becomes `Bound` when a compatible PV is found or a CSI provisioner dynamically creates one. Mount the PVC in a Pod to prove the full attach/mount path, not just binding.

## 2. Snapshot and restore

First verify the cluster exposes the snapshot API and has a CSI driver that supports snapshots:

```sh
kubectl api-resources | grep -i snapshot
kubectl get volumesnapshotclass
```

Create a `VolumeSnapshot` referencing the source PVC. Wait for `readyToUse: true`, then create a new PVC whose `dataSource` references that snapshot:

```yaml
spec:
  dataSource:
    name: data-snapshot
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
```

The restored PVC must request at least the source snapshot size and use a compatible driver/class. Verify data consistency requirements for the application; a filesystem snapshot is not automatically application-consistent.

## 3. `WaitForFirstConsumer`

With `volumeBindingMode: WaitForFirstConsumer`, the control plane delays PV binding or dynamic provisioning until a Pod using the PVC is scheduled. This lets the scheduler choose a node/zone that satisfies Pod constraints and volume topology. Use it for zonal block storage; it avoids creating a volume in a zone where the Pod cannot run.
