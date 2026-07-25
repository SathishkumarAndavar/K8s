# Kubernetes Operations FAQ

This file answers common cluster operations and troubleshooting questions with step-by-step explanations and diagrams.

## 1. What happens when you run `kubectl apply -f deployment.yaml`?

1. `kubectl` reads the manifest and sends it to the API server.
   - Uses `kubectl apply` to create or patch the `Deployment` object.
2. API server authenticates and authorizes the request.
3. Mutating and validating admission controllers run.
4. The final object is persisted in `etcd`.
5. The API server updates watch caches and emits events.
6. `DeploymentController` sees the new `Deployment` and creates or updates a `ReplicaSet`.
7. `ReplicaSetController` creates the desired number of `Pod` objects.
8. Each pod is stored as a `Pod` object in `etcd` with `status: Pending`.
9. The scheduler watches for unscheduled pods and assigns a node via `spec.nodeName`.
10. The kubelet on the assigned node sees the scheduled pod and creates it.
11. The container runtime pulls images, creates pod sandbox and containers.
12. The kubelet reports pod status back to the API server.

### Key points
- `kubectl apply` is a client-side request to the API server, not a pod creator itself.
- The control plane components coordinate to reach the desired state.
- The actual workload starts on the worker node through kubelet and runtime.

## 2. Difference between Control Plane HA and Worker Node HA

### Control Plane HA
- Goal: keep cluster control and management available.
- Involves multiple instances of:
  - `kube-apiserver`
  - `etcd`
  - `kube-scheduler`
  - `kube-controller-manager`
- Uses leader election for elected controllers and quorum for `etcd`.
- Protects against failure of master nodes.

### Worker Node HA
- Goal: keep application workloads available.
- Involves multiple worker nodes and pod redundancy.
- Uses:
  - Deployments / ReplicaSets for replicated pods
  - pod anti-affinity and topology spread
  - PodDisruptionBudgets and multi-node placement
- Protects against worker node failure or node-level problems.

### Comparison
- Control Plane HA protects cluster governance; Worker Node HA protects running workloads.
- Control Plane HA is about API availability and state consistency.
- Worker Node HA is about pod placement, redundancy, and node failure tolerance.

## 3. How does `kube-proxy` route traffic to Pods?

### Overview
- `kube-proxy` watches `Service` and `Endpoints` objects from the API server.
- It programs the node’s network stack to forward traffic for Service IPs to backend pod IPs.

### Modes
- `iptables` mode:
  - creates NAT rules in the kernel.
  - traffic to `ClusterIP:port` is DNATed to a selected pod IP:port.
  - service load balancing is implemented through chained iptables rules.
- `ipvs` mode:
  - creates IPVS virtual servers.
  - supports multiple scheduling algorithms like round-robin and least connections.

### Traffic flow
1. A client sends traffic to `ServiceIP:port`.
2. Kernel matches rules installed by `kube-proxy`.
3. The destination is rewritten to one of the pod endpoints.
4. Packet is forwarded to the pod.

### Important details
- The service IP is virtual; the real endpoints are pod IPs.
- `kube-proxy` does not inspect application payloads.
- It handles `ClusterIP`, `NodePort`, `LoadBalancer`, and `ExternalIPs`.

## 4. Why is `etcd` so critical, and how do you back it up?

### Why `etcd` is critical
- `etcd` is the single source of truth for Kubernetes state.
- It stores all API objects: pods, deployments, services, ConfigMaps, Secrets, nodes, RBAC definitions, and more.
- The API server reads and writes through `etcd`.
- If `etcd` is corrupted or lost, the control plane cannot recover cluster state.
- Control plane HA depends on `etcd` quorum for consistency.

### How to back up `etcd`
1. Use `etcdctl snapshot save` from a healthy member.
2. Use TLS credentials for secure access.
3. Store snapshots externally.
4. Keep multiple snapshots and test restores.

Example backup command:
```bash
export ETCDCTL_API=3
etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /backup/etcd-snapshot.db
```

Example restore command:
```bash
etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir=/var/lib/etcd-from-backup
```

### Best practices
- Backup regularly and automatically.
- Store off-node or remotely.
- Protect and encrypt snapshots.
- Validate restore process periodically.

## 5. Pod stuck in `CrashLoopBackOff` or `Pending`: which component failed?

### CrashLoopBackOff
- Indicates the container starts but then crashes repeatedly.
- The failure is at the kubelet/runtime/application level, not scheduling.
- Check if the pod is already assigned to a node.
- Inspect `kubectl describe pod` and logs (`kubectl logs --previous`).

### Typical causes
- application process exits with an error
- missing configuration or environment variables
- port conflicts
- probe failures causing restarts

### Pending
- `Pending` means the pod is not yet running.
- If `nodeName` is empty, the scheduler likely cannot place the pod.
- If `nodeName` exists, kubelet or node resources may be the issue.

### Common Pending causes
- `FailedScheduling`: insufficient resources or affinity/taint mismatch.
- `ErrImagePull` / `ImagePullBackOff`: image download failure.
- `PodInitializing` / `ContainerCreating`: volume mount or CNI issue.
- Node conditions: `NotReady`, `DiskPressure`, `MemoryPressure`.

### Troubleshooting logic
- `CrashLoopBackOff` ⇒ kubelet/runtime or application failure.
- `Pending` without `nodeName` ⇒ scheduler/control-plane placement issue.
- `Pending` with `nodeName` ⇒ kubelet/node or runtime issue.
- Use `kubectl describe pod`, node status, events, and logs.

## 6. What happens if the API Server goes down? Will applications stop running?

### What continues
- Existing pods continue running on worker nodes.
- The data plane remains active (`kubelet`, runtime, network) for already scheduled workloads.
- Services and Pod networking continue if node networking is healthy.

### What stops
- No new API requests are accepted.
- `kubectl` cannot communicate with the cluster.
- Controllers cannot reconcile new desired state.
- The scheduler cannot place new pods.
- Status updates may not persist to `etcd`.

### Control plane vs data plane isolation
- The API server is control plane infrastructure.
- It manages cluster state and handles requests.
- Data plane workloads run independently once scheduled.
- So applications do not immediately stop if the API server fails.

## Diagram

![Kubernetes operations flow](k8s-operations-diagram.svg)
