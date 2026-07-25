# Kubernetes Pod Creation Flow

This file explains how a pod is created in Kubernetes and which control plane components are involved.

## 1. User submits a manifest
A user or automation submits a pod/deployment manifest to the Kubernetes API Server.

- Common submission methods:
  - `kubectl apply -f pod.yaml`
  - `kubectl create deployment`
  - CI/CD pipeline or GitOps controller

## 2. API Server receives the request
The `kube-apiserver` validates and stores the manifest in `etcd`.

- Authentication and authorization occur.
- Admission controllers may mutate or validate the request.
- The desired state is persisted in the cluster store.

## 3. Scheduler assigns a node
The `kube-scheduler` watches for new unscheduled pods.

- It evaluates node suitability based on:
  - resource requests and limits
  - node selectors and affinity/anti-affinity
  - taints and tolerations
  - available resources and topology
- Once a node is selected, the scheduler updates the pod object with the chosen node name.

## 4. Kubelet on the target worker node acts
The `kubelet` on the assigned node observes the scheduled pod.

- It pulls pod and container specs from the API server.
- It creates the pod sandbox and container(s) using the container runtime.
- It mounts volumes, config maps, secrets, and applies network setup.

## 5. Container runtime starts containers
The container runtime (for example, `containerd` or `CRI-O`):

- pulls required container images
- creates container processes
- reports container status back to `kubelet`

## 6. Pod becomes running
The `kubelet` reports pod and container status back to the API server.

- The pod transitions through phases: `Pending` → `ContainerCreating` → `Running`
- Readiness probes and liveness probes may still run after the pod enters `Running`.

## 7. Control plane continues reconciliation
The `kube-controller-manager` continuously reconciles the cluster state.

- ReplicaSet or Deployment controllers ensure the desired number of pods exist.
- If a pod fails or a node goes down, controllers recreate pods as needed.

## Key points to remember
- The API server is the single source of truth for desired state.
- The scheduler assigns pods to nodes, but does not create containers.
- The kubelet and container runtime on the worker node are responsible for actual pod lifecycle management.
- Controllers observe the current state and act to match it with the desired state.

## Recommended interview talking points
- Explain the separation between desired state and actual state.
- Describe how a Deployment leads to ReplicaSet and pod creation.
- Mention the role of admission controllers and the scheduler.
- Highlight the kubelet’s role in reporting status back to the control plane.
