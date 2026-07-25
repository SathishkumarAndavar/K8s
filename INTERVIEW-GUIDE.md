# Kubernetes Interview Guide

Use this repository in this order: explain the architecture, choose the right Kubernetes object, describe the control loop or traffic path, then show how you would verify it with `kubectl`.

## A strong answer pattern

1. **Define the object or component** in one sentence.
2. **Explain the flow**: which controller or node component reacts, and what state changes.
3. **State the trade-off**: availability, cost, security, topology, or operational risk.
4. **Verify or troubleshoot** with one or two exact commands.

Example: “A Deployment manages ReplicaSets to maintain a desired number of interchangeable Pods. The Deployment controller creates a new ReplicaSet for a rollout and scales old and new ReplicaSets according to its strategy. I would use it for stateless services; for stable identity and storage I would choose StatefulSet. I would verify with `kubectl rollout status deployment/<name>` and `kubectl get rs,pods -l app=<name>`.”

## High-value topic map

| Interview area | Start here | Be ready to explain |
| --- | --- | --- |
| Cluster architecture | [Cluster Architecture](cluster-architecture/README.md) | API server, etcd, scheduler, controller manager, kubelet, kube-proxy, CNI, CRI |
| Pod lifecycle and failures | [Pod lifecycle](cluster-architecture/pod-lifecycle.md) | Pending vs CrashLoopBackOff, probes, restart policy, deletion and graceful termination |
| Workloads | [Workloads](concepts/workloads/workload-controllers.md) | Deployment, ReplicaSet, StatefulSet, DaemonSet, Job, CronJob, rollout and rollback |
| Scheduling | [Scheduling](concepts/scheduling/node-selectors-and-affinity.md) | requests, affinity, taints/tolerations, topology spread, priority/preemption |
| Services and ingress | [Services and networking](concepts/services-networking/services-and-networking.md) | Service selector and EndpointSlice, DNS, kube-proxy, Ingress versus controller, NetworkPolicy |
| Storage | [Persistent storage](concepts/storage/persistent-volumes-storageclasses-and-claims.md) | PV, PVC, StorageClass, CSI, access modes, reclaim policy, topology |
| Security | [Pod Security Standards](concepts/security/pod-security-standards.md) | authentication versus authorization, RBAC, ServiceAccounts, Secrets, admission |
| Autoscaling | [HPA vs VPA](concepts/workloads/autoscaling/hpa-vs-vpa.md) | metrics, requests, replica scaling, rightsizing, node autoscaling |
| Operations | [Cluster administration](concepts/cluster-administration/cluster-and-node-upgrades.md) | draining, upgrades, certificates, etcd backup/restore, node health |
| Hands-on | [CKA runbooks](CKA/README.md) | fast diagnosis, YAML edits, resource discovery, and validation |

## Incident triage order

```text
Symptom → kubectl get / describe → Events → Logs or node signals → Dependency checks → Fix template/config → Verify rollout
```

- **Pod Pending**: events, requests, affinity, taints, PVC binding, quotas.
- **Pod CrashLoopBackOff**: previous logs, command/config, probes, OOM, dependencies.
- **Service unreachable**: selectors → EndpointSlices → Pod readiness → DNS → network policy/CNI.
- **Node NotReady**: node conditions → kubelet/runtime → disk/memory/PID pressure → CNI → API reachability.

## Practice rules

- Use fully qualified, resource-specific commands during an interview: `kubectl describe pod`, `kubectl logs --previous`, `kubectl get events --sort-by=.lastTimestamp`.
- Check the active context and namespace before making changes.
- Prefer `kubectl apply -f` for repeatable work; use `kubectl edit` only when speed matters and validate immediately afterward.
- Treat etcd snapshots and Secrets as sensitive data.

## Canonical references

- [Kubernetes Concepts](https://kubernetes.io/docs/concepts/)
- [Kubernetes API reference](https://kubernetes.io/docs/reference/kubernetes-api/)
- [kubectl reference](https://kubernetes.io/docs/reference/kubectl/)
