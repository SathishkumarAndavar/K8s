# Pod Lifecycle and Probes

This document explains the lifecycle phases of a Pod and the probes used to manage its health.

## Pod Lifecycle Phases

A Pod's `status.phase` field provides a high-level summary of where the Pod is in its lifecycle.

-   **`Pending`**: The Pod has been accepted by the cluster, but one or more of its containers has not been created yet. This includes time spent waiting to be scheduled and time spent downloading images.
-   **`Running`**: The Pod has been bound to a node, and all of its containers have been created. At least one container is still running, or is in the process of starting or restarting.
-   **`Succeeded`**: All containers in the Pod have terminated in success, and will not be restarted.
-   **`Failed`**: All containers in the Pod have terminated, and at least one container has terminated in failure.
-   **`Unknown`**: For some reason the state of the Pod could not be obtained, typically due to a communication error with the kubelet on the node.

```mermaid
stateDiagram-v2
    [*] --> Pending
    Pending --> Running: All containers created
    Running --> Succeeded: All containers exit with 0
    Succeeded --> [*]
    Running --> Failed: At least one container exits with non-zero
    Failed --> [*]

    state Running {
        direction LR
        [*] -> Healthy: Readiness probe passes
        Healthy -> Unhealthy: Readiness probe fails
        Unhealthy -> Healthy: Readiness probe passes
        Healthy -> Restarting: Liveness probe fails
        Restarting -> Healthy
    }
```

## Container Probes

The `kubelet` can perform three types of probes on running containers to check their health.

### 1. Liveness Probe

-   **Purpose**: To determine if a container is running correctly.
-   **Action**: If the liveness probe fails, the `kubelet` kills the container, and the container is subject to its `restartPolicy`.
-   **Use Case**: To restart a container that has become unresponsive or deadlocked, even though the process is still running.

### 2. Readiness Probe

-   **Purpose**: To determine if a container is ready to serve traffic.
-   **Action**: If the readiness probe fails, the endpoints controller removes the Pod's IP address from the endpoints of all Services that match the Pod.
-   **Use Case**: To prevent traffic from being sent to a Pod that is still starting up, or is temporarily unable to handle requests (e.g., loading data).

### 3. Startup Probe

-   **Purpose**: To determine if a containerized application has finished starting up.
-   **Action**: If a startup probe is configured, all other probes are disabled until it succeeds. If the startup probe fails, the `kubelet` kills the container, and it is subject to its `restartPolicy`.
-   **Use Case**: For slow-starting applications that need extra startup time before they are ready for liveness checks. This prevents the liveness probe from killing the container before it has a chance to start.

### Probe Configuration Example

```yaml
spec:
  containers:
  - name: my-app
    image: my-app-image
    ports:
    - containerPort: 8080
    readinessProbe:
      httpGet:
        path: /readyz
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

**Key takeaway**: Readiness probes control whether a Pod receives traffic; liveness probes control whether a container is restarted.