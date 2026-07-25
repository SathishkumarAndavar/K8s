# Deployment YAML Fields Explained

This file explains key fields in a `deployment.yaml` and what `selector` and `template` mean.

## Complete Deployment example

This includes the required fields plus the most important production fields. Values such as image, resource size, probe path, and replica count must match the application.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: production
  labels:
    app.kubernetes.io/name: my-app
    app.kubernetes.io/component: api
spec:
  replicas: 3
  minReadySeconds: 5
  revisionHistoryLimit: 5
  progressDeadlineSeconds: 600
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: my-app
  template:
    metadata:
      labels:
        app.kubernetes.io/name: my-app
        app.kubernetes.io/component: api
    spec:
      serviceAccountName: my-app
      terminationGracePeriodSeconds: 30
      securityContext:
        seccompProfile: { type: RuntimeDefault }
      containers:
      - name: web
        image: nginx:1.27
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 8080
          protocol: TCP
        env:
        - name: LOG_LEVEL
          value: info
        resources:
          requests: { cpu: 100m, memory: 128Mi }
          limits: { cpu: 500m, memory: 256Mi }
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          capabilities: { drop: [ALL] }
        startupProbe:
          httpGet: { path: /startupz, port: http }
          failureThreshold: 30
          periodSeconds: 5
        readinessProbe:
          httpGet: { path: /readyz, port: http }
          initialDelaySeconds: 2
          periodSeconds: 5
        livenessProbe:
          httpGet: { path: /healthz, port: http }
          initialDelaySeconds: 10
          periodSeconds: 10
```

## Key fields
- `apiVersion`: API version of the Deployment resource.
- `kind`: `Deployment`.
- `metadata.name`: deployment name.
- `spec.replicas`: desired number of pods.

## `selector`
- Defines how the Deployment finds its pods.
- Must match labels in `template.metadata.labels`.
- Cannot be changed after creation in many cases.

## `template`
- Defines the pod spec for pods managed by the Deployment.
- Contains pod metadata and spec.
- The template labels must match the selector.

## Important subfields
- `spec.template.spec.containers`: list of containers.
- `image`: container image.
- `ports`: container ports.
- `resources`: resource requests and limits.
- `env`: environment variables.
- `volumeMounts` and `volumes`: storage and config.
- `strategy`: rolling update or recreate.
- `minReadySeconds`: wait time before pod is ready.
- `revisionHistoryLimit`: number of old ReplicaSets kept.

## Example extended fields
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```

## Important points
- `selector` is the stable identity for the Deployment.
- `template` is the pod specification used to create pods.
- Labels are the link between Deployment and pods.
- `spec.replicas` is the desired state, not the current state.
