# API Access Control

Every request to the Kubernetes API server goes through a series of stages to ensure it is secure and valid.

## 1. Authentication ("AuthN")

**Goal**: To verify the identity of the user, group, or service account making the request.
-   **Mechanisms**: Client Certificates, Bearer Tokens (including Service Account tokens), OpenID Connect (OIDC), and Webhook Token Authentication.
-   The result is a `user`, `group`, and `extra` info that is passed to the next stage.

## 2. Authorization ("AuthZ")

**Goal**: To determine if the authenticated user is allowed to perform the requested action on the requested resource.
-   **Mechanisms**:
    -   **Role-Based Access Control (RBAC)**: The standard and most common method. Uses `Roles`, `ClusterRoles`, `RoleBindings`, and `ClusterRoleBindings` to grant permissions.
    -   **Node Authorization**: A special-purpose authorizer that grants permissions to kubelets based on the pods they are scheduled to run.
    -   **Webhook Mode**: Calls an external service to make an authorization decision.

## 3. Admission Control

**Goal**: To validate or mutate a request after it has been authenticated and authorized, but before it is persisted to `etcd`.
-   **Mechanisms**:
    -   **Mutating Admission Webhooks**: Can modify objects. For example, injecting a sidecar container.
    -   **Validating Admission Webhooks**: Can reject objects that violate policy. For example, blocking pods that use a `hostPath` volume.
    -   **Built-in Controllers**: `LimitRanger`, `ResourceQuota`, and `PodSecurityAdmission` are all examples of built-in admission controllers.