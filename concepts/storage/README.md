# Storage

Kubernetes provides a powerful volume abstraction that allows containers to consume different types of storage, from ephemeral on-disk files to persistent disks from a cloud provider.

## Core Storage Resources

-   **Volume**: A directory, possibly with some data in it, which is accessible to the containers in a Pod. The lifecycle of a volume is tied to the Pod, but its contents can be preserved across container restarts.
-   **PersistentVolume (PV)**: A piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using StorageClasses. It is a cluster resource, just like a node.
-   **PersistentVolumeClaim (PVC)**: A request for storage by a user. It is similar to a Pod. Pods consume node resources and PVCs consume PV resources.
-   **StorageClass**: Provides a way for administrators to describe the "classes" of storage they offer. Different classes might map to quality-of-service levels, or to backup policies, or to arbitrary policies determined by the cluster administrators.
-   **CSI (Container Storage Interface)**: A standard for exposing arbitrary block and file storage systems to containerized workloads on Kubernetes.

*See also: [Storage Concepts](https://kubernetes.io/docs/concepts/storage/)*