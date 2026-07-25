# Considerations for Large Clusters

Summary:
- Large clusters (hundreds–thousands of nodes) require careful design for control plane, etcd, networking, observability, and operational processes.

Key areas:
- **etcd & control plane:** Shard or isolate etcd for scale; keep etcd size and latency small. Prefer dedicated control-plane nodes or managed control planes.
- **API server load:** Use API server horizontal scaling (where supported), API caches, or aggregation layer to reduce load.
- **Networking:** Choose CNI that supports scale and provides efficient IP management; monitor overlay performance and MTU settings.
- **Node lifecycle & autoscaling:** Use cluster autoscaler with scale-down protections and graceful draining; automate node image/OS updates.
- **Resource quotas & namespaces:** Enforce quotas, limit ranges, and RBAC to partition tenants and control resource consumption.
- **Monitoring & logging:** Centralize metrics (Prometheus remote write), logs, and traces; sample and downsample high-cardinality metrics.
- **CI/CD & GitOps:** Use progressive rollouts, canaries, and automated rollback strategies; limit blast radius with namespaces and feature flags.

Notes:
- Test disaster recovery, etcd backups/restores, and verify control-plane failover.
- Plan for cost: use node pools/instance types, spot instances, and autoscaling policies.
