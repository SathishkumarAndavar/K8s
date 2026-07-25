# Pod Security Standards

This file explains Pod Security Standards (PSS) and how they enforce pod security.

## What are Pod Security Standards?
- PSS are built-in security profiles in Kubernetes.
- They provide a simple way to enforce pod security policies.
- Profiles:
  - `privileged`
  - `baseline`
  - `restricted`

## What each profile means
- `privileged`: allows most capabilities and host access.
- `baseline`: prevents known privilege escalation but allows common workloads.
- `restricted`: enforces the strongest security boundaries.

## How to apply
- Use `PodSecurity` Admission on a namespace.
- Example:
```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  runAsUser:
    rule: MustRunAsNonRoot
```

## Important points
- `restricted` is recommended for security-conscious environments.
- `baseline` is good for most standard applications.
- `privileged` is for trusted system workloads only.
- PSS is the newer, simpler replacement for PodSecurityPolicy.
