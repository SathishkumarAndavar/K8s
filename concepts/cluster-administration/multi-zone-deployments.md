# Running in Multiple Zones

This file explains how to run Kubernetes workloads across multiple zones.

## Why multi-zone?
- Improve application availability.
- Avoid single-zone failures.
- Provide fault tolerance and disaster recovery.

## Kubernetes support
- Use zone-aware node labels: `topology.kubernetes.io/zone`.
- Configure topology spread constraints and affinity.
- Use storage that supports multi-zone access.

## Example
- Deploy pods with topology spread across zones.
- Use a Service with `topologyKeys` or CNI that supports cross-zone routing.

## Important points
- Multi-zone requires a multi-zone cluster.
- Services and storage should be zone-aware.
- Use `PodDisruptionBudgets` to preserve availability during maintenance.
