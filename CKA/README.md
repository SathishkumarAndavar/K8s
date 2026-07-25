# Kubernetes Practical Examples

This document contains practical, copy-paste-ready examples for common Kubernetes administration and development tasks. Each example includes the manifest, `kubectl` commands, and an explanation of the outcome.

## 1. Advanced Deployment with Probes and Rolling Update Strategy

This example demonstrates a Deployment with liveness and readiness probes, resource requests/limits, and a configured rolling update strategy for zero-downtime updates.

**`deployment-advanced.yaml`**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-prod
  labels:
    app: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp-container
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "250m"
            memory: "256Mi"
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
```

**Commands:**
```sh
# Apply the manifest
kubectl apply -f deployment-advanced.yaml

# Watch the rollout
kubectl get pods -l app=webapp -w

# Trigger a rolling update by changing the image tag
kubectl set image deployment/webapp-prod webapp-container=nginx:1.26

# Watch the rolling update status
kubectl rollout status deployment/webapp-prod
```

## 2. NetworkPolicy for Namespace Isolation

This example creates a default "deny-all" ingress policy for a namespace, then adds a specific rule to allow traffic only from Pods in another namespace (e.g., `monitoring`).

**`netpol-isolate.yaml`**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {} # An empty selector matches all pods in the namespace
  policyTypes:
  - Ingress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-monitoring-ingress
spec:
  podSelector:
    matchLabels:
      app: webapp # Apply this policy only to our webapp pods
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: monitoring
```

**Commands:**
```sh
# Create the namespaces and a pod to test with
kubectl create ns web-app
kubectl create ns monitoring
kubectl run test-pod -n web-app --image=nginx --labels=app=webapp

# Apply the policies to the web-app namespace
kubectl apply -n web-app -f netpol-isolate.yaml

# Test connectivity (this will fail/timeout)
kubectl run busybox -n default --image=busybox --rm -it -- sh -c "wget -O- --timeout=2 http://test-pod.web-app"

# Test connectivity from the allowed namespace (this will succeed)
kubectl run busybox -n monitoring --image=busybox --rm -it -- sh -c "wget -O- --timeout=2 http://test-pod.web-app"
```