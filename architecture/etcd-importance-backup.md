# Why `etcd` is critical and how to back it up

This file covers why `etcd` is the most important data store in Kubernetes and how to create backups.

## Why `etcd` is critical
- `etcd` is the single source of truth for all Kubernetes objects.
- The API server reads from and writes to `etcd` for every cluster change.
- It stores objects such as:
  - Pods, Deployments, ReplicaSets
  - Services, ConfigMaps, Secrets
  - Nodes, RBAC policies, Namespaces
- Without `etcd`, the control plane cannot recover the desired state.
- `etcd` quorum is required for consistent writes in HA control planes.

## Backup strategy
- Use `etcdctl snapshot save` to take consistent snapshots.
- Back up from a healthy cluster member.
- Use the same TLS credentials as the cluster.
- Store snapshots off-node or in remote storage.

### Backup example
```bash
export ETCDCTL_API=3
etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /backup/etcd-snapshot.db
```

## Restore strategy
- Stop API server / control plane components before restore.
- Run `etcdctl snapshot restore`.
- Recreate member configuration with correct cluster settings.

### Restore example
```bash
etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-from-backup
```

## Best practices
- Backup regularly on a schedule.
- Keep multiple snapshots with retention.
- Encrypt snapshots and protect access.
- Validate restore procedures periodically.
- Back up control plane certificates and config as well.

## Interview points
- `etcd` is critical because it holds cluster state; losing it can make a cluster unrecoverable.
- A healthy `etcd` snapshot is the simplest path to restore cluster state.
- Backups should be automated, tested, and stored outside the cluster.
