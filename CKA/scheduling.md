# CKA Scheduling — Answers and Practice

## 1. How scheduling works

The `kube-scheduler` is the component responsible for assigning Pods to Nodes. It makes its decisions through a two-phase process for each Pod.

```mermaid
graph TD
    A[New Pod created, no nodeName] --> B{Scheduler watches for Pod};
    B --> C{Phase 1: Filtering};
    C --> D["Node 1 (Insufficient CPU) --> Filtered Out"];
    C --> E["Node 2 (Taint not tolerated) --> Filtered Out"];
    C --> F["Node 3 (Feasible)"];
    C --> G["Node 4 (Feasible)"];
    
    subgraph Feasible Nodes
        F & G
    end

    Feasible_Nodes --> H{Phase 2: Scoring};
    H --> I{Node 3 Score: 50, Node 4 Score: 80};
    I --> J[Select Node 4 (Highest Score)];
    J --> K[Bind Pod to Node 4];
```

The scheduler watches for Pods without `spec.nodeName`. It filters nodes that cannot run the Pod—resource requests, node selector/affinity, taints, volume topology, ports, and other constraints—then scores the remaining nodes and binds the Pod to the best node. The kubelet on that node then creates the Pod.

For a Pending Pod, start with:

```sh
kubectl describe pod <pod>
kubectl get nodes -o wide
kubectl top nodes
```

Events usually state the failing constraint, such as `Insufficient cpu`, an untolerated taint, or unmet affinity.

## 2. Affinity, taints, and spread

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disk
            operator: In
            values: [ssd]
  tolerations:
  - key: dedicated
    operator: Equal
    value: payments
    effect: NoSchedule
  topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app: payments
```

Label a node with `kubectl label node <node> disk=ssd`. Taint it with `kubectl taint node <node> dedicated=payments:NoSchedule`. A toleration permits scheduling onto the tainted node; it does not force it—use affinity too when you need placement.

## 3. Priority and preemption

Create a `PriorityClass`, then set `priorityClassName` on critical Pods. If no node can fit a high-priority Pod, the scheduler can preempt lower-priority Pods when that would make the Pod schedulable. Preemption is not guaranteed: PDBs, affinity, and topology can still prevent a workable placement.

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: platform-critical
value: 100000
globalDefault: false
description: Critical platform workload
```

For critical workloads, combine an appropriate PriorityClass, resource requests, multiple replicas, PDBs, and topology spread. Do not rely on preemption as normal capacity management.
