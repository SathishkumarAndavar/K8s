# Kubelet Reference

The `kubelet` is the primary agent running on each worker node. It is responsible for managing the lifecycle of pods assigned to its node.

## Key Responsibilities

-   **Pod Lifecycle Management**: Communicates with the API server to get pod specifications and ensures the containers described in those specs are running and healthy.
-   **Container Runtime Interface (CRI)**: Interacts with the container runtime (like `containerd` or `CRI-O`) to start, stop, and manage containers.
-   **Node and Pod Status Reporting**: Reports the health of the node and the status of its pods back to the control plane.
-   **Volume Management**: Mounts and unmounts volumes for containers.
-   **Resource Management**: Enforces resource constraints and handles node-pressure eviction when resources like memory or disk are low.

## Configuration

The kubelet's behavior is configured via a `KubeletConfiguration` file, command-line flags, and a dynamic configuration mechanism. Key settings include cgroup driver, eviction thresholds, and feature gates.

*See also: Kubelet Reference*