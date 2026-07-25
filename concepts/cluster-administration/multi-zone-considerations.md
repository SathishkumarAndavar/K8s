# Running in Multiple Zones

Summary:
- Design clusters and apps to tolerate zone failures by distributing nodes and replicas across availability zones (AZs).

When to use:
- For high availability and reduced blast radius in cloud environments.

Key considerations:
- **Control plane:** Run control plane across zones (managed services usually handle this).
- **etcd:** Keep etcd healthy; avoid spanning a single etcd cluster across high-latency zones unless properly sized.
- **Scheduling:** Use topology-aware scheduling (`topologyKeys`, `topologySpreadConstraints`), node labels (zone), and anti-affinity to distribute pods.
- **Load balancing:** Use zone-aware external LBs or multi-zone cloud LB; ensure health checks are zone-aware.
- **Storage:** Use multi-zone storage classes (replicated volumes) or zone-local PVs with replication at the application layer.

Examples:
- topologySpreadConstraints example:
```yaml
topologySpreadConstraints:
- maxSkew: 1
  topologyKey: topology.kubernetes.io/zone
  whenUnsatisfiable: ScheduleAnyway
  labelSelector:
    matchLabels:
      app: web
```

Notes:
- Prefer stateless services across zones and stateful services with explicit replication.
- Test zone failure scenarios and rolling upgrades.
