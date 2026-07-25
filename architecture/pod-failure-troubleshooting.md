# Pod stuck in `CrashLoopBackOff` or `Pending`: which component failed?

This file explains system-level troubleshooting logic for pods stuck in `CrashLoopBackOff` or `Pending`.

## CrashLoopBackOff

### What it means
- The pod was scheduled and the kubelet/runtime attempted to start the container.
- The container process started but then exited repeatedly.
- Kubernetes backs off restart attempts.

### Likely failed component(s)
- `kubelet` / container runtime: responsible for starting containers and tracking restarts.
- The application container itself: the root cause if it crashes.
- Probes: failing readiness or liveness probes can trigger restarts.

### What to check
- `kubectl describe pod <pod>`
- `kubectl logs <pod> --previous`
- Pod event reasons like `CrashLoopBackOff`, `Back-off restarting failed container`
- Container exit codes and crash messages

## Pending

### What it means
- The pod is not yet running.
- It may be waiting for scheduling or node-level resources.

### If `nodeName` is not set
- Likely a scheduler-related issue.
- Common causes:
  - insufficient CPU/memory
  - node selectors or affinity/anti-affinity mismatch
  - taints/tolerations conflicts
  - no available nodes

### If `nodeName` is set
- The pod is assigned, but kubelet/node cannot complete startup.
- Common causes:
  - image pull failure (`ErrImagePull`, `ImagePullBackOff`)
  - volume mount or PVC provisioning issue
  - CNI/network problem
  - node conditions like `NotReady`, `DiskPressure`, `MemoryPressure`

### What to check
- `kubectl describe pod <pod>`
- Pod events for reasons like `FailedScheduling`, `ErrImagePull`, `PodInitializing`
- Node readiness and resource status
- PVC/PV status if storage is involved

## Interview-friendly logic
- `CrashLoopBackOff` is usually a kubelet/runtime/application level failure after scheduling.
- `Pending` without `nodeName` is usually a scheduler/control-plane placement issue.
- `Pending` with `nodeName` is usually a node/kubelet/runtime issue.
- Use the pod events and status to determine which component is failing.
