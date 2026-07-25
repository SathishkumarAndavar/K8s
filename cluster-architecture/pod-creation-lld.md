# Kubernetes Pod Creation Low-Level Design (LLD)

This document describes the low-level design for Kubernetes pod creation, including component interactions, object lifecycle transitions, and the exact sequence of operations.

## Scope

Covers pod creation from client request to container runtime execution, including:

- API Server request handling
- admission control and validation
- persistence in etcd
- scheduling decisions
- kubelet execution on worker nodes
- container runtime orchestration
- status reporting and reconciliation

## Component interaction summary

### Control plane components
- `kubectl` / client: submits resource YAML or API calls.
- `kube-apiserver`: central API endpoint, validates requests, persists objects.
- `etcd`: source of truth storing cluster state.
- `kube-scheduler`: determines node placement for unscheduled pods.
- `kube-controller-manager`: contains controllers such as ReplicaSet, Deployment, and Node controllers.
- Admission controllers: validate/mutate incoming pod objects.

### Worker node components
- `kubelet`: node agent that syncs pod state and manages lifecycle.
- container runtime (containerd / CRI-O): creates and runs containers.
- `kube-proxy`: manages service network rules for pod traffic.

## Data model and objects involved

### Primary Kubernetes objects
- `Pod`: the smallest deployable compute unit.
- `Deployment` / `ReplicaSet`: higher-level controllers that create pods.
- `Node`: worker node representation.
- `Service`: load balancing and discovery object.
- `Namespace`, `ConfigMap`, `Secret`, `PersistentVolumeClaim`: supporting objects used during pod creation.

### Pod fields relevant to scheduling and creation
- `spec.nodeName`: assigned by the scheduler.
- `spec.containers`: list of containers and runtime configuration.
- `spec.volumes`: declared volumes and mount targets.
- `status.phase`: pod lifecycle phase.
- `status.conditions`: readiness/liveness and scheduling conditions.
- `status.containerStatuses`: runtime container state details.

## Sequence of operations

### 1. Client request and API server processing

1. The client sends a `POST` or `PATCH` request to `/api/v1/namespaces/{ns}/pods`.
2. `kube-apiserver` authenticates the request using configured auth methods.
3. `kube-apiserver` authorizes the request against RBAC / ABAC policies.
4. Admission controllers execute sequentially.
   - `MutatingAdmissionWebhook` may alter pod spec.
   - `ValidatingAdmissionWebhook` checks policies.
   - built-in admission plugins validate image policies, resource quotas, or security contexts.
5. The validated pod object is stored in `etcd`.
6. `kube-apiserver` writes the object to its watch cache and triggers watchers.

### 2. ReplicaSet / Deployment controller behavior

1. A user may create a `Deployment` instead of a bare `Pod`.
2. The `Deployment` controller writes a `ReplicaSet`.
3. The `ReplicaSet` controller writes the desired number of `Pod` objects.
4. Each created `Pod` enters `Pending` status with no `spec.nodeName`.

### 3. Scheduler decision flow

1. `kube-scheduler` watches the `Pod` informer for unscheduled pods.
2. For each pod, the scheduler evaluates candidate nodes.
   - Node capacity and allocatable resources.
   - `PodSpec` requirements: CPU, memory, GPUs.
   - Node selectors, node affinity, and pod affinity/anti-affinity.
   - Taints and tolerations.
   - Topology spread constraints and resource claims.
3. A scoring function ranks viable nodes.
4. Scheduler chooses the best node and patches the `Pod` with `spec.nodeName`.
5. `kube-apiserver` updates the pod object and broadcasts the change.

### 4. kubelet pod sync and container creation

1. The kubelet on the assigned node receives the pod update from the API server watch stream.
2. The kubelet compares the desired pod spec with the current state of local pods.
3. The kubelet executes the pod sync flow:
   - Validate pod spec and locate images.
   - Ensure secrets/configmaps are available.
   - Create a pod sandbox via the CRI.
   - Create containers in the pod sandbox.
   - Mount volumes and establish network interfaces.
4. The kubelet monitors container startup and updates status.

### 5. Container runtime execution

1. The kubelet calls the CRI runtime to pull images if needed.
2. The runtime creates container processes and configures namespaces, cgroups, and storage.
3. The runtime returns container state to the kubelet.
4. For each container, probes may execute to determine readiness or liveness.

### 6. Status reporting and state reconciliation

1. The kubelet updates pod `status.phase` and `containerStatuses` back to the API server.
2. The API server persists status updates into `etcd`.
3. Controllers observe the updated pod state and compare actual state with desired state.
4. If containers fail or nodes become unreachable, controllers may recreate pods.

## Low-level design considerations

### Watchers and informers
- Each component uses watches/informers to stay up to date with Kubernetes objects.
- Informer caches reduce API server load and improve responsiveness.
- The scheduler and kubelet use shared informer factories to watch `Pod`, `Node`, and related resources.

### Retry and backoff
- The scheduler retries failed scheduling decisions with exponential backoff.
- Kubelet handles transient runtime failures and container image pulls with retries.
- Controllers use work queues to process objects and requeue on errors.

### API server caching and consistency
- The API server serves data from an in-memory cache.
- Write operations are serialized and persisted in `etcd`.
- `etcd` uses leader election and quorum writes for HA.

### Admission control details
- Admission plugins can be configured in `kube-apiserver`.
- `ValidatingAdmissionWebhook` ensures security, compliance, and policy enforcement.
- `MutatingAdmissionWebhook` can inject sidecars or default values.

## Pod creation internals by component

### kube-apiserver
- Serializes incoming JSON/YAML into internal API objects.
- Runs object validation schema checks.
- Sends watch events for created/updated objects.

### kube-scheduler
- Uses `Predicate` checks to filter nodes.
- Uses `Priority` functions to rank nodes.
- Performs `Bind` operation via API server patch.

### kubelet
- Uses the `Pod` API spec and local container runtime status.
- Implements pod lifecycle state machine:
  - `Pending` → `Running` → `Succeeded` / `Failed`.
- Handles pod eviction during node pressure or maintenance.

### Container runtime
- Uses CRI interface to create sandboxes, containers, and network configuration.
- Integrates with CNI plugins for pod networking.
- Reports `ContainerStatus` through CRI to kubelet.

## Sequence diagram (text-based)

```text
Client -> API Server: Create pod manifest
API Server -> Admission: Validate / mutate
Admission -> API Server: Approval
API Server -> etcd: Persist pod object
API Server -> Watchers: Pod created event
Scheduler -> API Server: Get unscheduled pod
Scheduler -> Nodes: Evaluate scheduling constraints
Scheduler -> API Server: Patch pod nodeName
API Server -> Kubelet: Node assignment event
Kubelet -> CRI runtime: Create pod sandbox and containers
CRI runtime -> Kubelet: Container status
Kubelet -> API Server: Pod status update
Controller -> API Server: Reconcile pod state
```

## Interview-ready notes

- Emphasize the separation of concerns between control plane and data plane.
- Highlight the API server as the single source of truth and `etcd` as the persistent store.
- Explain how the scheduler is responsible only for placement, not pod execution.
- Connect `kubelet` with the runtime and CRI for actual container creation.
- Mention how controllers continuously reconcile actual state to desired state.

## References
- Kubernetes control plane overview: https://kubernetes.io/docs/concepts/overview/components/
- Kubernetes scheduler concept: https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/
- kubelet concept: https://kubernetes.io/docs/concepts/architecture/nodes/#kubelet
- CRI and container runtimes: https://kubernetes.io/docs/concepts/architecture/cri/
