# Deployments, DaemonSets, StatefulSets

This file explains when to use Deployments, DaemonSets, and StatefulSets, and how they differ.

## Deployment
- Used for stateless applications.
- Manages ReplicaSets to keep a desired number of identical pods running.
- Supports rolling updates, rollbacks, scaling, and progressive delivery.

### Use case
- Web frontends, API servers, batch workers, stateless services.

### Key behavior
- Pods are interchangeable.
- Any pod can be replaced by another.
- Pod identity is not stable.

## DaemonSet
- Ensures one pod runs on every selected node.
- Useful for node-level agents.
- Common uses: logging agents, monitoring agents, networking plugins, storage drivers.

### Use case
- Fluentd, Prometheus Node Exporter, Calico node agent.

### Key behavior
- Pod is scheduled on each node that matches the DaemonSet selector.
- New nodes automatically get a pod.
- Removing a node removes the pod.

## StatefulSet
- Used for stateful applications that need stable identity and storage.
- Pod names remain stable: `<statefulset-name>-0`, `<statefulset-name>-1`, etc.
- Persistent storage is provisioned per pod using PVC templates.
- Scaling and updates are ordered and controlled.

### Use case
- Databases, Kafka, Zookeeper, etcd clusters.

### Key behavior
- Stable network identity and storage.
- Ordered startup, termination, and scaling.
- Good for workloads that need consistent identity.

## Comparison
| Feature | Deployment | DaemonSet | StatefulSet |
|---|---|---|---|
| Pod identity | ephemeral | per-node | stable |
| Storage | optional | optional | stable PVC per pod |
| Scaling | parallel | not scale horizontally | ordered |
| Update strategy | rolling | update all/rolling | ordered rolling |
| Typical use | stateless apps | node agents | stateful services |

## Example
- Use Deployment for a frontend service with 5 replicas.
- Use DaemonSet for a cluster-wide logging agent.
- Use StatefulSet for a database cluster with persistent storage.

## Important points
- StatefulSet is not a replacement for Deployment; use it only when stable identity/storage are required.
- DaemonSet is for per-node infrastructure, not user-facing applications.
- Deployments are the default for most application workloads.
