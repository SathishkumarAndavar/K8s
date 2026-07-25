# Control Plane HA vs Worker Node HA

This file explains the difference between Control Plane High Availability and Worker Node High Availability in Kubernetes.

## Control Plane HA

### Purpose
- Keep the cluster management plane available.
- Ensure the API server, scheduler, controller manager, and etcd remain reachable.

### What it includes
- Multiple `kube-apiserver` instances.
- Multiple `etcd` members arranged in a quorum.
- Multiple `kube-scheduler` instances with leader election.
- Multiple `kube-controller-manager` instances with leader election.

### What it protects
- API availability for kubectl and automation.
- Cluster state consistency.
- Scheduling and reconciliation control.

## Worker Node HA

### Purpose
- Keep application workloads available.
- Ensure pods remain running when worker nodes fail.

### What it includes
- Multiple worker nodes.
- Replicated workloads using Deployments / ReplicaSets.
- Pod anti-affinity, topology spread, and PodDisruptionBudgets.
- Resilient networking and storage.

### What it protects
- Application availability.
- Workload redundancy and failure tolerance.
- Node-level disruption and resource failure.

## Key differences
- Control Plane HA protects cluster control, not individual application pods.
- Worker Node HA protects actual running workloads.
- Control Plane HA is about API and state management.
- Worker Node HA is about pod placement and redundancy.

## Example
- With Control Plane HA, the API server can survive a master node failure, but if all worker nodes fail, pods still go down.
- With Worker Node HA, pods are replicated across nodes, but if the API server is down you cannot make new changes.

## Interview summary
- Control Plane HA = highly available cluster management.
- Worker Node HA = highly available workload execution.
- Both are needed for a resilient Kubernetes environment.
