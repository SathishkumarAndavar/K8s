# Taints and Tolerations

This file explains how taints and tolerations control pod placement on nodes.

## Taints
- Applied to nodes to repel pods.
- A node with a taint will reject pods unless they tolerate it.
- Format: `key=value:effect`
- Effects:
  - `NoSchedule`: pod is not scheduled unless it tolerates the taint.
  - `PreferNoSchedule`: scheduler avoids the node if possible.
  - `NoExecute`: existing pods are evicted unless they tolerate the taint.

### Example taint
```bash
kubectl taint nodes node1 dedicated=database:NoSchedule
```

## Tolerations
- Added to pod specs to allow scheduling onto tainted nodes.
- A toleration does not guarantee placement; it only permits it.

### Example toleration
```yaml
spec:
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "database"
    effect: "NoSchedule"
```

## Use cases
- Reserve nodes for special workloads.
- Keep user workloads off control plane or GPU nodes.
- Evict pods from nodes undergoing maintenance.

## Important points
- Taints and tolerations are a node-level scheduling control.
- Use `NoExecute` for nodes that should drain pods quickly.
- Combine taints with node selectors/affinity when designing node pools.
