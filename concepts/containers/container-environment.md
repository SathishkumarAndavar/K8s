# Container Environment

Summary:
- Best practices for providing environment to containers: environment variables, ConfigMaps, Secrets, and downward API.

When to use:
- **ConfigMaps:** Non-sensitive config (feature flags, config files).
- **Secrets:** Sensitive values (credentials, tokens); use `type: kubernetes.io/tls` or generic string data.
- **Downward API / PodSpec fields:** Expose metadata, labels, annotations to containers.

Examples:
```yaml
env:
- name: ENV
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: ENV
- name: DB_PASS
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password
```

Notes:
- Mount Secrets as files for larger data; enable encryption at rest for Secrets; avoid storing large binaries in Secrets.
- Use `imagePullSecrets` for private registries and `securityContext` for runtime constraints.
