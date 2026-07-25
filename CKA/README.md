# Policies

Policies are objects that enforce rules and constraints across the cluster to ensure security, fairness, and adherence to best practices.

## Core Policy Resources

-   **LimitRange**: Provides constraints to limit resource consumption (like CPU and memory) per-Pod or per-Container in a namespace. It can also set default requests/limits for containers that don't specify them.
-   **ResourceQuota**: Provides constraints that limit aggregate resource consumption per namespace. It can limit the total amount of CPU, memory, or storage a namespace can use, as well as the number of objects (like Pods, Services, etc.) that can be created.
-   **Pod Security Admission (PSA)**: A built-in admission controller that enforces the Pod Security Standards (`Privileged`, `Baseline`, `Restricted`) at the namespace level. It is the recommended way to enforce pod security in modern clusters.

*See also: Policy Concepts*