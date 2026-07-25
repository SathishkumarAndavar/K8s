# Deployment, StatefulSet, DaemonSet

Summary:
- **Deployment:** Replica management for stateless apps; rolling updates and rollbacks.
- **StatefulSet:** Ordered, stable network IDs and persistent storage for stateful apps.
- **DaemonSet:** Run one pod per node (or subset) for node-level services.

When to use:
- **Deployment:** Web servers, APIs, stateless services, jobs requiring rollouts.
- **StatefulSet:** Databases, clustered systems (Zookeeper, Kafka brokers), any workload that needs stable identity and persistent volume claims.
- **DaemonSet:** Log collectors, monitoring agents, network plugins, node-local proxies.

Key differences:
- **Identity & storage:** StatefulSet provides stable pod names and PVC templates; Deployment pods are interchangeable.
- **Scaling:** Deployments scale by replica count; StatefulSet scales with ordinal semantics and may require special handling for scale-down/up.
- **Scheduling:** DaemonSet ensures coverage across nodes; Deployment/StatefulSet rely on replicas and affinity/taints.

Examples (minimal):
- Deployment:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: nginx:stable
```

- StatefulSet (with PVC template):
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: db
spec:
  serviceName: "db"
  replicas: 3
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
      - name: postgres
        image: postgres:14
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

- DaemonSet:
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-agent
spec:
  selector:
    matchLabels:
      app: node-agent
  template:
    metadata:
      labels:
        app: node-agent
    spec:
      containers:
      - name: agent
        image: myorg/node-agent:latest
        securityContext:
          privileged: true
```

Notes:
- Use PodDisruptionBudgets with StatefulSets for safe maintenance.
- Prefer Deployments for stateless services for easier rollbacks and scaling.
