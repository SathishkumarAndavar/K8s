# Topology in Kubernetes

This file explains topology concepts for pod placement and failure domains.

## Topology keys
- Kubernetes uses topology keys to represent failure domains.
- Common keys:
  - `kubernetes.io/hostname`
  - `topology.kubernetes.io/zone`
  - `topology.kubernetes.io/region`

## Use cases
- Spread pods across zones or nodes.
- Increase availability by avoiding single points of failure.
- Place workloads close to storage or network resources.

## Topology spread constraints
- Used in pod spec affinity rules.
- Example:
```yaml
spec:
  topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app: my-app
```

## Important points
- Topology controls distribution across zones/nodes.
- It helps avoid concentration of pods in one failure domain.
- Combined with affinity, it provides resilient placement.
