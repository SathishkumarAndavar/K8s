# Practical Kubernetes Examples: YAML and Commands

This document provides hands-on, practical examples for common Kubernetes tasks. Each example includes the necessary YAML manifests and the `kubectl` commands to apply and verify the configuration. Use these scenarios to prepare for hands-on interview questions.

---

## Example 1: Create and Verify a Deployment

**Goal:** Deploy a simple NGINX web server with 3 replicas.

#### 1. Deployment Manifest

Save this content as `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
```

#### 2. Commands

```sh
# Apply the manifest to create the Deployment
kubectl apply -f deployment.yaml

# Check the status of the rollout
kubectl rollout status deployment/nginx-deployment

# List the Pods created and managed by the Deployment
kubectl get pods -l app=nginx

# Get detailed information about the Deployment
kubectl describe deployment nginx-deployment
```

---

## Example 2: Expose a Deployment with a Service

**Goal:** Expose the `nginx-deployment` inside the cluster using a `ClusterIP` Service.

#### 1. Service Manifest

Save this content as `service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

#### 2. Commands

```sh
# Apply the manifest to create the Service
kubectl apply -f service.yaml

# Verify the Service and its endpoints
kubectl get service nginx-service

# Describe the service to see the selector and endpoints
kubectl describe service nginx-service

# Test connectivity from another pod in the cluster
kubectl run tmp-shell --rm -i --tty --image busybox -- /bin/sh
# Inside the busybox shell:
# wget -O- nginx-service
```

---

## Example 3: Use a ConfigMap for Configuration

**Goal:** Create a ConfigMap and mount it as a file into the NGINX Pods.

#### 1. ConfigMap and Updated Deployment Manifests

Save this as `configmap.yaml`:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome from ConfigMap!</title>
    </head>
    <body>
    <h1>This page is served from a ConfigMap volume.</h1>
    </body>
    </html>
```

Update `deployment.yaml` to mount the ConfigMap:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-index
          mountPath: /usr/share/nginx/html
      volumes:
      - name: nginx-index
        configMap:
          name: nginx-config
```

#### 2. Commands

```sh
# Create the ConfigMap first
kubectl apply -f configmap.yaml

# Apply the updated Deployment manifest
kubectl apply -f deployment.yaml

# Verify the file is mounted inside one of the pods
POD_NAME=$(kubectl get pods -l app=nginx -o jsonpath='{.items[0].metadata.name}')
kubectl exec $POD_NAME -- cat /usr/share/nginx/html/index.html
```