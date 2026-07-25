# Enforcing Pod Security Standards

Summary:
- Pod Security Standards (PSS) define recommended security posture levels (Privileged, Baseline, Restricted). Enforce with admission controllers.

When to use:
- Use PSS to reduce attack surface and enforce cluster-wide defaults for workloads.

How to enforce:
- Use the built-in `PodSecurity` admission (label-based enforcement) or a policy engine (OPA/Gatekeeper) for more complex rules.
- Labels on namespaces determine the mode and enforcement: `pod-security.kubernetes.io/enforce: restricted`.

Minimal example (namespace label):
```bash
kubectl label namespace prod pod-security.kubernetes.io/enforce=restricted
kubectl label namespace prod pod-security.kubernetes.io/audit=baseline
```

Common controls to apply:
- `runAsNonRoot`, `runAsUser`, and `fsGroup` policies
- `readOnlyRootFilesystem`
- Drop all Linux capabilities and add only those required
- Seccomp and AppArmor profiles
- Disallow privileged containers and hostPath mounts unless necessary

Notes:
- Test policies in `audit` mode before `enforce`.
- Use mutating admission (e.g., PodSecurity admission, Kyverno) to inject safe defaults.
