# Google Kubernetes Engine (GKE)

## How it works

GKE provides a managed Kubernetes control plane and data plane, offering different modes of operation. Worker nodes are Compute Engine VMs that are automatically registered with the cluster control plane.

## Important features

-   **Modes of Operation**:
    -   **Standard**: You manage the underlying node infrastructure, giving you flexibility over node configuration. You pay for nodes per second.
    -   **Autopilot**: GKE provisions and manages the entire cluster infrastructure, including nodes and node pools. You are billed for the pods you run, not the nodes, simplifying operations and cost management.
-   **Node Pools**: The primary way to manage data plane capacity. You can create multiple node pools with different machine types, zones, labels, and taints (e.g., for system workloads vs. applications).
-   **Autoscaling**: GKE offers excellent autoscaling capabilities, including cluster autoscaler for nodes and Vertical Pod Autoscaling (VPA) out-of-the-box.
-   **Networking**:
    -   **VPC-native clusters (Alias IPs)**: Pods get IPs directly from the VPC's secondary range, making them natively routable within the VPC. This is the recommended and default mode.
    -   **Routes-based clusters**: Uses a network bridge on each node, requiring custom routes for pod-to-pod communication across nodes.
-   **Identity**: Use **Workload Identity** to bind Kubernetes Service Accounts to Google Cloud Service Accounts, providing a secure, recommended way for pods to access Google Cloud APIs.
-   **Storage**: The **GCE Persistent Disk CSI Driver** is automatically enabled, allowing seamless use of `pd-standard`, `pd-balanced`, and `pd-ssd` storage classes.

## Essential commands

```sh
# Authenticate kubectl to the cluster
gcloud container clusters get-credentials <cluster-name> --zone <zone> --project <project-id>
kubectl get nodes -o wide

# Inspect cluster and node pools
gcloud container clusters describe <cluster-name> --zone <zone>
gcloud container node-pools list --cluster <cluster-name> --zone <zone>
gcloud container node-pools describe <pool-name> --cluster <cluster-name> --zone <zone>
```

## Upgrade Process

GKE simplifies upgrades with **release channels**.

1.  **Release Channels**: Subscribe your cluster to a channel (e.g., `Rapid`, `Regular`, `Stable`). GKE will automatically manage and roll out version upgrades for both the control plane and nodes within that channel. This is the highly recommended approach.
2.  **Manual Upgrades**:
    -   **Upgrade Control Plane**: You can manually initiate a control plane upgrade to a specific available version.
    -   **Upgrade Node Pools**: After the control plane is upgraded, you upgrade each node pool one by one. GKE uses a **surge upgrade** strategy by default: it adds new "surge" nodes with the new version, drains old nodes, and then removes them, minimizing downtime.

## Interview distinction

GKE's key differentiators are its **Autopilot mode**, which offers a serverless-like Kubernetes experience, and its mature, tightly integrated **release channels** for automated and secure cluster upgrades. Workload Identity provides a very clean model for pod-level GCP permissions.

## Official references

-   GKE Architecture
-   About GKE release channels
-   Node pool upgrades