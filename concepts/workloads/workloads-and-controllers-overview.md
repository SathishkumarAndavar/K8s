# Workload Management (Controllers)

Summary of core workload controllers and when to use them.

- **Deployment:** Declarative updates for stateless applications; supports rolling updates, rollbacks, and versioned deployments. Use for most web services and stateless apps.
- **ReplicaSet:** Ensures a set number of pod replicas; typically managed by Deployments.
- **StatefulSet:** Ordered, stable identity and persistent storage for stateful apps (databases, clustered services).
- **DaemonSet:** Run one pod per node (or selected nodes) for node-level agents (logging, monitoring, network plugins).
- **Job:** Run pods to completion (batch work); use when a task must complete successfully once (or up to retries).
- **CronJob:** Schedule Jobs on a time-based schedule; use for periodic batch tasks.
- **ReplicationController:** Legacy controller similar to ReplicaSet; rarely used in modern clusters.

Other workload-related topics:
- **PodDisruptionBudget (PDB):** Control voluntary eviction to keep minimum available replicas during maintenance.
- **Init containers vs Sidecars:** `initContainers` run before app containers for setup; sidecars run alongside main container to provide auxiliary functionality (logging, proxies).
- **Pod Hostname & Subdomain:** Configure pod networking identity when needed for DNS-based discovery (useful with StatefulSets).
- **Pod QoS classes:** `Guaranteed`, `Burstable`, `BestEffort` determined by requests/limits; affects eviction ordering.
- **Ephemeral Containers:** Debug-only containers that can be attached to running pods for troubleshooting.
- **Scheduling groups / PodGroup:** Workload API concepts for grouping pods for scheduling and priority/disruption handling.
- **Downward API:** Expose pod metadata (labels, annotations, namespace) into containers via env or files.
- **Advanced Pod Configuration:** Init/sidecar lifecycles, lifecycle hooks, probes, securityContext, topologySpreadConstraints.
- **User namespaces & rootless containers:** Run containers with remapped UIDs for improved host protection.
- **Static Pods:** Pods managed directly by kubelet on a node (manifests on disk), not via the API server — used for bootstrapping control plane.
- **Workload API / PodGroup Priority & Scheduling Policies:** Use the Workload API to express group priorities, disruption policies, and topology-aware scheduling strategies.

See Kubernetes docs:

- https://kubernetes.io/docs/concepts/workloads/controllers/
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
- https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/
- https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/
- https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/
- https://kubernetes.io/docs/concepts/workloads/controllers/job/
- https://kubernetes.io/docs/concepts/workloads/controllers/ttlafterfinished/
- https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/
- https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/
