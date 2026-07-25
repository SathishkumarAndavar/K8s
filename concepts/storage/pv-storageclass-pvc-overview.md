# PersistentVolume (PV), StorageClass, PersistentVolumeClaim (PVC)

Summary:
- PV: Cluster resource representing storage; PVC: a claim to bind to a PV; StorageClass: dynamic provisioning policy.

When to use:
- Use **PVCs** for workloads that require persistent storage (databases, stateful apps). Use **StorageClasses** for dynamic provisioning, performance tiers, and reclaim policies.

Key considerations:
- **AccessModes:** ReadWriteOnce (RWO), ReadWriteMany (RWX), ReadOnlyMany (ROX).
- **ReclaimPolicy:** Retain vs Delete — choose based on backup/restore practices.
- **VolumeBindingMode:** Immediate vs WaitForFirstConsumer (useful for topology-aware provisioning).
- **Provisioner:** Choose cloud provider CSI drivers or on-prem drivers; use CSI for production.
- **Capacity & IOPS:** Match PV type to workload (SSD for DBs, HDD for archival).

Example (StorageClass + PVC):
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  iopsPerGB: "10"
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-data
spec:
  accessModes: ["ReadWriteOnce"]
  resources:
    requests:
      storage: 50Gi
  storageClassName: fast-ssd
```

Notes:
- For multi-zone clusters, prefer zone-local PVs with `WaitForFirstConsumer` or use multi-zone replicated storage.
- Backup and restore PVs using volume snapshots (CSI snapshotter) and application-consistent backups for DBs.
