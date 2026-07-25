# Pod Security Admission

**Pod Security Admission (PSA)** is a built-in admission controller that enforces the security policies defined by the **Pod Security Standards (PSS)**. It is the recommended, in-tree mechanism for enforcing baseline security for pods in a cluster.

## Pod Security Standards (PSS)

PSS define three policies, offering a broad spectrum of security hardening:

-   **`privileged`**: Unrestricted. Provides the widest level of permissions, allowing for known privilege escalations.
-   **`baseline`**: Minimally restrictive. Prevents known privilege escalations while allowing most default Pod configurations.
-   **`restricted`**: Heavily restricted. Follows current Pod hardening best practices, at the expense of some compatibility.

## How to Enforce PSS

PSA can be enforced at two levels: at the namespace level (most common) or at the cluster level.

### 1. Namespace-Level Enforcement

This is the most flexible and common approach. You apply labels to a namespace to define which PSS policy to enforce, audit, or warn on.

-   `enforce`: Policy violations will cause the pod to be rejected.
-   `audit`: Policy violations will be recorded in the audit log, but the pod will be allowed.
-   `warn`: Policy violations will generate a user-facing warning, but the pod will be allowed.

**Example:**

To enforce the `restricted` policy on all new pods in the `production` namespace and warn on `baseline` violations:

```sh
# Create the namespace
kubectl create ns production

# Apply the labels to enforce the 'restricted' policy
kubectl label ns production pod-security.kubernetes.io/enforce=restricted

# Apply labels to warn if a pod violates the 'baseline' policy
kubectl label ns production pod-security.kubernetes.io/warn=baseline
```

Now, any attempt to create a privileged pod in the `production` namespace will be rejected by the API server.

### 2. Cluster-Level Enforcement

You can configure the `PodSecurity` admission plugin directly in the API server's configuration file to set cluster-wide defaults. This is useful for ensuring a default security posture across all namespaces.

**Example `admission-control.yaml`:**

This configuration enforces the `restricted` policy cluster-wide by default, while exempting the `kube-system` namespace.

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: AdmissionConfiguration
plugins:
- name: PodSecurity
  configuration:
    apiVersion: pod-security.k8s.io/v1
    kind: PodSecurityConfiguration
    defaults:
      enforce: "restricted"
      enforce-version: "latest"
    exemptions:
      # Exempt system namespaces
      namespaces: ["kube-system"]
```

This file would be referenced by the `kube-apiserver` using the `--admission-control-config-file` flag.