# What happens when you run `kubectl apply -f deployment.yaml`

This document explains the step-by-step flow when a Deployment manifest is applied to a Kubernetes cluster.

## 1. Client submits the manifest
- `kubectl` reads `deployment.yaml`.
- It sends the object to the API server via the Kubernetes API.
- `kubectl apply` uses server-side apply or a patch operation to create/update the resource.

## 2. API Server request handling
- The API server authenticates the request.
- It authorizes the action using RBAC or other policies.
- Admission controllers may mutate or validate the object.
- The final `Deployment` object is stored in `etcd`.

## 3. Controller processing
- The `DeploymentController` sees the new or updated Deployment.
- It creates or updates a `ReplicaSet` object with the desired pod template.
- The `ReplicaSetController` ensures the desired replica count by creating or deleting pods.

## 4. Pod scheduling
- Each new pod is created with `status: Pending` and no `spec.nodeName`.
- The `kube-scheduler` watches unscheduled pods.
- It selects a suitable node based on resources, affinity, taints/tolerations, and topology.
- The scheduler binds the pod by setting `spec.nodeName`.

## 5. Pod creation on worker node
- The kubelet on the assigned node sees the scheduled pod.
- The kubelet pulls images, mounts volumes, and creates the pod sandbox.
- The container runtime starts the pod’s containers.
- The kubelet updates pod status back to the API server.

## 6. Result
- The desired Deployment state is reflected in the cluster.
- The specified number of pod replicas should become `Running`.
- If there are failures, the controllers and scheduler retry reconciliation.

## Interview points
- `kubectl` is just the client; the API server is the entry point.
- `Deployment` → `ReplicaSet` → `Pod` is the object creation chain.
- Scheduler assigns nodes; kubelet/runtime execute pods.
- Controllers and the control plane manage desired state reconciliation.
