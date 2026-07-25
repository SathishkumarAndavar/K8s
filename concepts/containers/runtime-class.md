# RuntimeClass

Summary:
- `RuntimeClass` allows selection of the container runtime configuration for pod classes (e.g., runc, gVisor, Kata).

When to use:
- Use `RuntimeClass` to run untrusted workloads in sandboxed runtimes (gVisor/Kata) or to choose a tuned runtime for latency-sensitive workloads.

Example:
```yaml
apiVersion: node.k8s.io/v1
kind: RuntimeClass
metadata:
  name: gvisor
handler: runsc

---
apiVersion: v1
kind: Pod
metadata:
  name: sandboxed
spec:
  runtimeClassName: gvisor
  containers:
  - name: app
    image: busybox
    command: ["sleep", "3600"]
```

Notes:
- Ensure node-level runtimeClass handlers are installed and nodes labeled/tainted appropriately for scheduling.
