# Scheduling, Preemption and Eviction

This section covers how Kubernetes decides where to run Pods and how it manages resource contention on nodes.

## Core Scheduling Concepts

-   **Kubernetes Scheduler**: The control plane component responsible for assigning newly created Pods to Nodes. It makes decisions based on a filtering and scoring process.
-   **Taints and Tolerations**: Taints are applied to nodes to repel Pods, while tolerations are applied to Pods to allow them to be scheduled on nodes with matching taints. This is used to dedicate nodes for specific workloads.
-   **Node Affinity/Anti-Affinity**: Rules that attract or repel Pods to/from nodes with certain labels. This provides more advanced control over placement than `nodeSelector`.
-   **Pod Affinity/Anti-Affinity**: Rules that attract or repel Pods to/from nodes based on the labels of other Pods already running on those nodes. This is used for co-locating or spreading out related services.
-   **Pod Topology Spread Constraints**: Provides fine-grained control over how Pods are spread across failure domains like regions, zones, and nodes.
-   **Pod Priority and Preemption**: Allows assigning priorities to Pods. If a high-priority Pod cannot be scheduled, the scheduler may preempt (evict) lower-priority Pods to make room.
-   **Node-pressure Eviction**: A process where the `kubelet` proactively evicts Pods from a node when it is running low on resources like memory or disk space.

*See also: Scheduling Concepts*