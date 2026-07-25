# Azure Kubernetes Service (AKS)

## How it works

Azure manages the Kubernetes control plane for free, and you are responsible for the agent nodes (worker nodes) that run your workloads. These nodes are Azure Virtual Machines.

## Important features

-   **Agent Pools**: Similar to node groups/pools in other clouds, you manage capacity via agent pools.
    -   **System Node Pools**: Required for running critical system pods like CoreDNS.
    -   **User Node Pools**: Used for running your application pods. Separating them is a best practice.
-   **Virtual Machine Scale Sets (VMSS)**: Under the hood, agent pools are backed by VMSS, which provides the scalability features.
-   **Networking**:
    -   **Kubenet (Basic)**: A simple networking option where nodes get an IP from the virtual network, but pods use a separate, NAT-ed address space.
    -   **Azure CNI (Advanced)**: Pods get full virtual network connectivity and are directly reachable. This is required for some advanced features like Windows node pools and Virtual Nodes but requires more careful IP address planning.
-   **Identity**: Use **Azure AD Workload Identity** to federate an identity for your pods, allowing them to securely access other Azure resources (like Key Vault or Storage) without needing secrets. This replaces the older AAD Pod Identity.
-   **Storage**: AKS integrates with Azure Disk and Azure Files. The built-in CSI drivers allow you to dynamically provision `managed-csi` (Azure Disk) and `azurefile-csi` (Azure Files) storage.

## Essential commands

```sh
# Authenticate kubectl to the cluster
az aks get-credentials --resource-group <resource-group> --name <cluster-name>
kubectl get nodes -o wide

# Inspect cluster and agent pools
az aks show --resource-group <resource-group> --name <cluster-name>
az aks nodepool list --resource-group <resource-group> --cluster-name <cluster-name>
az aks nodepool show --resource-group <resource-group> --cluster-name <cluster-name> --name <pool-name>
```

## Upgrade Process

AKS provides a coordinated upgrade process for the control plane and nodes.

1.  **Plan**: Check the supported Kubernetes versions in your region and review the release notes for any breaking changes.
2.  **Upgrade Control Plane**: First, upgrade the AKS control plane. You can do this via the Azure Portal or CLI.
    ```sh
    az aks upgrade --resource-group <resource-group> --name <cluster-name> --kubernetes-version <new-version>
    ```
3.  **Upgrade Node Pools**: After the control plane upgrade is complete, upgrade your node pools one by one. AKS performs a **cordon-and-drain** operation for each node in the pool. It adds a new node with the updated version to the pool, drains an old node, and then removes it, repeating until all nodes are upgraded.
    ```sh
    az aks nodepool upgrade --resource-group <resource-group> --cluster-name <cluster-name> --name <pool-name> --kubernetes-version <new-version>
    ```
    You can control the surge level with the `--max-surge` parameter to speed up the process at the cost of more resources.

## Interview distinction

AKS is deeply integrated into the Azure ecosystem. Key features include its use of Virtual Machine Scale Sets for agent pools, the choice between Kubenet and Azure CNI for networking, and the first-class support for Windows containers via Windows node pools. Azure AD Workload Identity is its modern solution for pod-level cloud permissions.

## Official references

-   AKS Core Concepts
-   Upgrade an AKS cluster
-   Azure AD Workload Identity