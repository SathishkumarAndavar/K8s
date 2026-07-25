# Workloads

Workloads are the objects you create in Kubernetes to manage and run your containerized applications. The most fundamental workload object is the **Pod**. All other workload controllers are abstractions that manage Pods.

## Core Workload Resources

-   **Pod**: The smallest and simplest unit in the Kubernetes object model that you create or deploy. A Pod represents a single instance of a running process in your cluster and can contain one or more containers.
-   **Controllers**: Higher-level objects that manage Pods and provide features like scaling, self-healing, and rollout strategies.
    -   **Deployment**: Manages a set of replicated, stateless Pods. Ideal for web servers and APIs. It handles rolling updates and rollbacks.
    -   **StatefulSet**: Manages a set of stateful Pods that require stable, unique network identifiers and persistent storage. Ideal for databases and other stateful applications.
    -   **DaemonSet**: Ensures that all (or some) Nodes run a copy of a Pod. Ideal for cluster-level agents like log collectors or monitoring agents.
    -   **Job**: Creates one or more Pods and ensures that a specified number of them successfully terminate. Ideal for batch processing and one-off tasks.
    -   **CronJob**: Manages a Job that runs on a repeating schedule.

*See also: [Pods](https://kubernetes.io/docs/concepts/workloads/pods/), [Workload Controllers](https://kubernetes.io/docs/concepts/workloads/controllers/)*