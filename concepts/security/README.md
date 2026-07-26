# Security

This section covers the concepts and objects used to secure a Kubernetes cluster and the applications running on it.

## Core Security Concepts

-   **Authentication**: The process of verifying the identity of a user or service account making a request to the API server.
-   **Authorization (RBAC)**: The process of determining if an authenticated user has permission to perform the requested action. Role-Based Access Control (RBAC) is the standard mechanism, using `Roles`, `ClusterRoles`, `RoleBindings`, and `ClusterRoleBindings`.
-   **Pod Security Standards (PSS)**: A set of security policies (`Privileged`, `Baseline`, `Restricted`) that define different levels of isolation for Pods.
-   **Pod Security Admission (PSA)**: A built-in admission controller that enforces the Pod Security Standards at the namespace level.
-   **Service Account**: Provides an identity for processes that run in a Pod, which can be used to grant permissions to the API server or other services.
-   **NetworkPolicy**: A specification of how groups of pods are allowed to communicate with each other and other network endpoints, acting as a firewall for pods.

---
*See also: Security Concepts*