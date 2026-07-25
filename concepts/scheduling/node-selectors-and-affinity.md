# Node Selectors, Node Affinity, and Pod Affinity

This file explains the Kubernetes scheduling controls for selecting nodes.

## Node selector
- Simplest way to constrain pods to nodes.
- Uses exact label matches.

### Example
```yaml
spec:
  nodeSelector:
    disktype: ssd
```

### Important points
- Easy but inflexible.
- A pod is scheduled only on nodes with matching labels.

## Node affinity
- More expressive version of node selection.
- Supports `requiredDuringSchedulingIgnoredDuringExecution` and `preferredDuringSchedulingIgnoredDuringExecution`.
- Allows operators such as `In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, `Lt`.

### Example
```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: zone
            operator: In
            values:
            - us-east-1a
            - us-east-1b
```

## Pod affinity / anti-affinity
- Controls colocation of pods with other pods.
- `podAffinity`: prefer or require pods to be scheduled near matching pods.
- `podAntiAffinity`: avoid scheduling near matching pods.

### Example
```yaml
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - web
        topologyKey: kubernetes.io/hostname
```

## Important points
- Node selector is a strict filter.
- Node affinity adds flexibility and topology awareness.
- Pod affinity/anti-affinity helps with application-level placement and failure domain separation.
- Use affinity when you need more than a single label match.
