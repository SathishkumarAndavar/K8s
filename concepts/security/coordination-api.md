# Coordination API (coordination.k8s.io)

The `coordination.k8s.io` API group provides resources for coordinating actions between components, primarily for leader election and health checking.

## Lease

A `Lease` object is a lightweight resource used to manage leader election and node heartbeats.

### Use Cases:

1.  **Leader Election**: This is the primary use case. Components like `kube-scheduler` and `kube-controller-manager` create `Lease` objects to acquire leadership. The component that successfully holds and renews the lease is the leader. This is more efficient than using `ConfigMaps` or `Endpoints`.

2.  **Node Heartbeats**: Since Kubernetes 1.14, `Lease` objects are used for node heartbeats. Each `kubelet` creates and periodically updates a `Lease` in the `kube-node-lease` namespace. This is much more lightweight than updating the full `Node` object for every heartbeat.

### Example `kubectl` command:

```sh
# Find the leader election lease for the controller manager
kubectl get lease -n kube-system kube-controller-manager -o yaml
```

## LeaseCandidate

A `LeaseCandidate` is a highly specialized resource introduced for **Dynamic Resource Allocation (DRA)**.

-   **Purpose**: It is used by third-party resource drivers to signal their availability to become the leader for managing a specific set of resources.
-   **How it works**: A DRA driver creates a `LeaseCandidate` object. The Kubernetes controller responsible for DRA (part of `kube-controller-manager`) observes these candidates and grants a `Lease` to one of them, making it the active resource manager.
-   **Relevance**: You will likely only encounter `LeaseCandidate` if you are developing or managing a custom resource driver that integrates with the Dynamic Resource Allocation framework. It is not used for general-purpose leader election.