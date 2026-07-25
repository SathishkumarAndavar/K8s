# Init Containers, Sidecars, and Probes

This file explains init containers, sidecars, and probes in Kubernetes.

## Init containers
- Run before application containers start.
- Useful for setup tasks like configuration generation, waiting for services, or migrations.
- They must complete successfully before pod startup continues.

### Example
```yaml
spec:
  initContainers:
  - name: init-db
    image: busybox
    command: ['sh', '-c', 'until nc -z db 5432; do sleep 1; done']
```

## Sidecars
- Containers that run alongside the main application container in the same pod.
- Common uses: logging, proxying, monitoring, configuration reload.
- They share the same network namespace and storage volumes.

### Example
- A logging agent sidecar reads application logs from a shared volume.
- An Envoy sidecar handles ingress traffic for the main app container.

## Probes
- Used to check container health.
- Types:
  - `livenessProbe`: restarts the container if it becomes unhealthy.
  - `readinessProbe`: controls whether the pod is ready to receive traffic.
  - `startupProbe`: checks whether the application has started.

### Example
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 10

readinessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

## Important points
- Init containers are sequential and must finish before app containers start.
- Sidecars are long-lived helpers in the same pod.
- Readiness probes keep unhealthy pods out of service load balancing.
- Liveness probes can recover a stuck container by restarting it.
- StartupProbe is useful for slow-starting applications.
